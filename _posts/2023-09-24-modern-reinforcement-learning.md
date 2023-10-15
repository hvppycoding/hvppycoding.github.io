---
title: "Modern Reinfocement Learning"
excerpt: "Fundamentals"
date: 2023-09-24 21:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-thomas-t-math.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Thomas T**](https://unsplash.com/@pyssling240) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Reinforcement Learning
mathjax: "true"
---

## First Exercise

- Frozen Lake environment
- Check out docs at gym.openai.com
- Random agent; 1000 games
- Plot win % over trailing 10 games

```python
import gym
import numpy as np

env = gym.make("FrozenLake-v1")

scores = []
win_rates = []

for i in range(1000):
    observation, info = env.reset()
    score = 0
    while True:
        action = env.action_space.sample()
        observation, reward, done, truncated, info = env.step(action)
        score += reward
        if done:
            break
    scores.append(score)
    
    if i % 10 == 0:
        average = np.mean(scores[-10:])
        win_rates.append(average)
env.close()

import matplotlib.pyplot as plt

plt.plot(win_rates)
plt.show()
```

## Fundamental Concepts

- **Policy**
  - Mapping of states to actions
  - Can be probabilistic
  - $\pi$

## Next Exercise

- Frozen Lake environment
- Reasonable deterministic policy
- 1000 games
- Plot win % over trailing 10 games

## Value Functions

$\pi(s, a)$: probability of taking action $a$ in state $s$ 일 때,

- **State-value function**
  - $v_\pi(s)$: expected return starting from state $s$ and following policy $\pi$

$v_\pi(s) = E_\pi[G_t \| S_t = s] = E_\pi[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \| S_t = s] \text{ for all } s \in S$

- **Action-value function**
  - $q_\pi(s, a)$: expected return starting from state $s$, taking action $a$, and following policy $\pi$

$q_\pi(s, a) = E_\pi[G_t \| S_t = s, A_t = a] = E_\pi[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \| S_t = s, A_t = a] \text{ for all } s \in S, a \in A$

## Bellman Equations

- $v_\pi(s) = \sum_{a \in A} \pi(s, a) \sum_{s', r} p(s', r \| s, a) \left[ r + \gamma v_\pi(s') \right]$
- Recursive relationship between value functions

## A Problem?

- Bellman Equation에서 $p(s', r \| s, a)$을 정확히 알 수 없다. 이를 모르면 Bellman Equation을 풀 수 없다.

## Linear Classifier

```python
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
import torch as T


class LinearClassifier(nn.Module):
    def __init__(self, lr, n_classes, input_dims):
        super(LinearClassifier, self).__init__()
        self.fc1 = nn.Linear(*input_dims, 128)
        self.fc2 = nn.Linear(128, 256)
        self.fc3 = nn.Linear(256, n_classes)
        
        self.optimizer = optim.Adam(self.parameters(), lr=lr)
        self.loss = nn.CrossEntropyLoss() #nn.MSELoss()
        self.device = T.device('cuda:0' if T.cuda.is_available() else 'cpu')
        self.to(self.device)
    
    def forward(self, data):
        layer1 = F.sigmoid(self.fc1(data))
        layer2 = F.sigmoid(self.fc2(layer1))
        layer3 = self.fc3(layer2)
        
        return layer3
    
    def learn(self, data, labels):
        self.optimizer.zero_grad()
        data = T.tensor(data).to(self.device)
        labels = T.tensor(labels).to(self.device)
        
        predictions = self.forward(data)
        
        cost = self.loss(predictions, labels)
        
        cost.backward()
        self.optimizer.step()
```

## Naive Deep Q-Network

```python
import gym
import numpy as np
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
import torch as T
import matplotlib.pyplot as plt
import numpy as np

def plot_learning_curve(x, scores, epsilons, filename):
    fig = plt.figure()
    ax = fig.add_subplot(111, label="1")
    ax2 = fig.add_subplot(111, label="2", frame_on=False)

    ax.plot(x, epsilons, color="C0")
    ax.set_xlabel("Training Steps", color="C0")
    ax.set_ylabel("Epsilon", color="C0")
    ax.tick_params(axis='x', colors="C0")
    ax.tick_params(axis='y', colors="C0")

    N = len(scores)
    running_avg = np.empty(N)
    for t in range(N):
        running_avg[t] = np.mean(scores[max(0, t-100):(t+1)])

    ax2.scatter(x, running_avg, color="C1")
    ax2.axes.get_xaxis().set_visible(False)
    ax2.yaxis.tick_right()
    ax2.set_ylabel('Score', color="C1")
    ax2.yaxis.set_label_position('right')
    ax2.tick_params(axis='y', colors="C1")

    plt.savefig(filename)

class LinearDeepQNetwork(nn.Module):
    def __init__(self, lr, n_actions, input_dims):
        super(LinearDeepQNetwork, self).__init__()

        self.fc1 = nn.Linear(*input_dims, 128)
        self.fc2 = nn.Linear(128, n_actions)

        self.optimizer = optim.Adam(self.parameters(), lr=lr)
        self.loss = nn.MSELoss()
        self.device = T.device('cuda:0' if T.cuda.is_available() else 'cpu')
        self.to(self.device)

    def forward(self, state):
        layer1 = F.relu(self.fc1(state))
        actions = self.fc2(layer1)

        return actions

class Agent():
    def __init__(self, input_dims, n_actions, lr, gamma=0.99,
                 epsilon=1.0, eps_dec=1e-5, eps_min=0.01):
        self.lr = lr
        self.input_dims = input_dims
        self.n_actions = n_actions
        self.gamma = gamma
        self.epsilon = epsilon
        self.eps_dec = eps_dec
        self.eps_min = eps_min
        self.action_space = [i for i in range(self.n_actions)]

        self.Q = LinearDeepQNetwork(self.lr, self.n_actions, self.input_dims)

    def choose_action(self, observation):
        if np.random.random() > self.epsilon:
            state = T.tensor(observation, dtype=T.float).to(self.Q.device)
            actions = self.Q.forward(state)
            action = T.argmax(actions).item()
        else:
            action = np.random.choice(self.action_space)

        return action

    def decrement_epsilon(self):
        self.epsilon = self.epsilon - self.eps_dec \
                        if self.epsilon > self.eps_min else self.eps_min

    def learn(self, state, action, reward, state_):
        self.Q.optimizer.zero_grad()
        states = T.tensor(state, dtype=T.float).to(self.Q.device)
        actions = T.tensor(action).to(self.Q.device)
        rewards = T.tensor(reward).to(self.Q.device)
        states_ = T.tensor(state_, dtype=T.float).to(self.Q.device)

        q_pred = self.Q.forward(states)[actions]

        q_next = self.Q.forward(states_).max()

        q_target = rewards + self.gamma*q_next

        loss = self.Q.loss(q_target, q_pred).to(self.Q.device)
        loss.backward()
        self.Q.optimizer.step()
        self.decrement_epsilon()

if __name__ == '__main__':
    env = gym.make('CartPole-v1')
    n_games = 10000
    scores = []
    eps_history = []

    agent = Agent(lr=0.0001, input_dims=env.observation_space.shape,
                  n_actions=env.action_space.n)

    for i in range(n_games):
        score = 0
        done = False
        obs, info = env.reset()

        while not done:
            action = agent.choose_action(obs)
            obs_, reward, done, trun, info = env.step(action)
            score += reward
            agent.learn(obs, action, reward, obs_)
            obs = obs_
        scores.append(score)
        eps_history.append(agent.epsilon)

        if i % 100 == 0:
            avg_score = np.mean(scores[-100:])
            print('episode ', i, 'score %.1f avg score %.1f epsilon %.2f' %
                  (score, avg_score, agent.epsilon))
    filename = 'cartpole_naive_dqn.png'
    x = [i+1 for i in range(n_games)]
    plot_learning_curve(x, scores, eps_history, filename)
```

결과 그래프는 다음과 같다. 결과가 불안정하다.

![cartpole_naive_dqn]({{site.baseurl}}/assets/images/2023-09-25-cartpole_naive_dqn.png){: .align-center}  

## Actor Critic Methods

### Policy Approximation

- Policy: probability distribution
- Parameterized with N.N weights $\theta$
- Gradient ascent in performance $J(\theta)$
- Policy Gradient Method
  - $\theta_{t+1} = \theta_t + \alpha \nabla_\theta J(\theta)$

### Advantages of Policy Approximation

- Policy is continuous and finite
- Works for continuous action spaces
- All actions sampled so no dilemma(exploration vs. exploitation)

### Action Selection in Policy Gradients

- $h(s, a, \theta)$: Numerical preference for state action pair
- $\pi(a \| s, \theta) = \frac{\exp\left({h(s, a, \theta)}\right)}{\sum_{b \in A} \exp\left({h(s, b, \theta)}\right)}$

### Limitations of Policy Gradients Methods

- Sample inefficiency
- Credit assignment problem
- Unstable solutions
- May still need to learn V, Q

## Policy Gradient Theorem

- $J(\theta) = v_{\pi_\theta}(s_0)$: value of the episode start state
- $\nabla J(\theta) \propto  \sum_s{\mu(s)}\sum_a{q_\pi(s, a)}\nabla_\theta \pi(a \| s, \theta)$

## REINFORCE

- $\nabla J(\theta) = E_\pi [\sum_a{q_\pi(S_t, a)}\nabla_\theta \pi(a \| S_t, \theta)]$
- 위 식에서 policy로 나누고 곱한다 = 1을 곱하는 것과 같음
- $\nabla J(\theta) = E_\pi [\sum_a{\pi(a \| S_t, \theta) q_\pi(S_t, a) \frac{\nabla_\theta \pi(a \| S_t, \theta)}{\pi(a \| S_t, \theta)}}]$
- $\nabla J(\theta) = E_\pi [q_\pi(S_t, A_t) \frac{\nabla_\theta \pi(A_t \| S_t, \theta)}{\pi(A_t \| S_t, \theta)}]$
- $\nabla J(\theta) = E_\pi [G_t \frac{\nabla_\theta \pi(A_t \| S_t, \theta)}{\pi(A_t \| S_t, \theta)}]$
- $\frac{\nabla x}{x} = \nabla \ln{x} \implies \frac{\nabla_\theta \pi(A_t \| S_t, \theta)}{\pi(A_t \| S_t, \theta)} = \nabla_\theta \ln{\pi(A_t \| S_t, \theta)}$
- $\nabla J(\theta) = E_\pi [G_t \nabla_\theta \ln{\pi(A_t \| S_t, \theta)}]$
- $\theta_{t+1} = \theta_t + \alpha G_t \nabla_\theta \ln{\pi(A_t \| S_t, \theta)}$
- 여기에서 $G_t$는 환경으로부터 얻을 수 있고, $\pi(A_t \| S_t, \theta)$는 Neural Network로부터 얻을 수 있다.

## Actor Critic

- $G_t$ 대신에 Temporal Difference Loss $\delta = R_t + \gamma v_\pi(S_{t+1}) - v_\pi(S_t)$를 사용(Actor loss)
- Critic loss: $\delta^2$

```python
# Actor Critic
import numpy as np
import torch as T
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
import numpy as np

class ActorCriticNetwork(nn.Module):
    def __init__(self, lr, input_dims, n_actions, fc1_dims=256, fc2_dims=256):
        super(ActorCriticNetwork, self).__init__()
        self.fc1 = nn.Linear(*input_dims, fc1_dims)
        self.fc2 = nn.Linear(fc1_dims, fc2_dims)
        self.pi = nn.Linear(fc2_dims, n_actions)
        self.v = nn.Linear(fc2_dims, 1)
        self.optimizer = optim.Adam(self.parameters(), lr=lr)
        self.device = T.device('cuda:0' if T.cuda.is_available() else 'cpu')
        self.to(self.device)

    def forward(self, state):
        x = F.relu(self.fc1(state))
        x = F.relu(self.fc2(x))
        pi = self.pi(x)
        v = self.v(x)

        return (pi, v)

class Agent():
    def __init__(self, lr, input_dims, fc1_dims, fc2_dims, n_actions, 
                 gamma=0.99):
        self.gamma = gamma
        self.lr = lr
        self.fc1_dims = fc1_dims
        self.fc2_dims = fc2_dims
        self.actor_critic = ActorCriticNetwork(lr, input_dims, n_actions, 
                                               fc1_dims, fc2_dims)
        self.log_prob = None

    def choose_action(self, observation):
        state = T.tensor([observation], dtype=T.float).to(self.actor_critic.device)
        probabilities, _ = self.actor_critic.forward(state)
        probabilities = F.softmax(probabilities, dim=1)
        action_probs = T.distributions.Categorical(probabilities)
        action = action_probs.sample()
        log_prob = action_probs.log_prob(action)
        self.log_prob = log_prob

        return action.item()

    def learn(self, state, reward, state_, done):
        self.actor_critic.optimizer.zero_grad()

        state = T.tensor([state], dtype=T.float).to(self.actor_critic.device)
        state_ = T.tensor([state_], dtype=T.float).to(self.actor_critic.device)
        reward = T.tensor(reward, dtype=T.float).to(self.actor_critic.device)

        _, critic_value = self.actor_critic.forward(state)
        _, critic_value_ = self.actor_critic.forward(state_)

        delta = reward + self.gamma*critic_value_*(1-int(done)) - critic_value

        actor_loss = -self.log_prob*delta
        critic_loss = delta**2

        (actor_loss + critic_loss).backward()
        self.actor_critic.optimizer.step()

import gym
import matplotlib.pyplot as plt

def plot_learning_curve(x, scores, figure_file):
    running_avg = np.zeros(len(scores))
    for i in range(len(running_avg)):
        running_avg[i] = np.mean(scores[max(0, i-100):(i+1)])
    plt.plot(x, running_avg)
    plt.title('Running average of previous 100 scores')
    plt.savefig(figure_file)

if __name__ == '__main__':
    env = gym.make('LunarLander-v2')
    agent = Agent(gamma=0.99, lr=5e-6, input_dims=[8], n_actions=4,
                  fc1_dims=2048, fc2_dims=1536)
    n_games = 3000

    fname = 'ACTOR_CRITIC_' + 'lunar_lander_' + str(agent.fc1_dims) + \
            '_fc1_dims_' + str(agent.fc2_dims) + '_fc2_dims_lr' + str(agent.lr) +\
            '_' + str(n_games) + 'games'
    figure_file = 'plots/' + fname + '.png'

    scores = []
    for i in range(n_games):
        done = False
        observation, info = env.reset()
        score = 0
        while not done:
            action = agent.choose_action(observation)
            observation_, reward, done, trunc, info = env.step(action)
            score += reward
            agent.learn(observation, reward, observation_, done)
            observation = observation_
        scores.append(score)

        avg_score = np.mean(scores[-100:])
        print('episode ', i, 'score %.1f' % score,
                'average score %.1f' % avg_score)

    x = [i+1 for i in range(n_games)]
    plot_learning_curve(x, scores, figure_file)
```
