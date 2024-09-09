---
title: "SOCC2024"
excerpt: ""
date: 2024-09-09 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

<audio controls>
  <source src="{{site.baseurl}}/assets/media/2024-09-09-socc-presentation-script.mp3" type="audio/mp3">
</audio>

Hello everyone. My name is Jayoung Yang, a Master's student at Seoul National University.
Today, I'll be presenting our work titled "Improving Timing Quality Through Net Topology Optimization in Global Routing".

This slide outlines my presentation.
I'll begin with an introduction to our work, followed by an explanation of the background on net topology structures.
Then, I'll describe our proposed methodology and experimental results.
Finally, I'll summarize the key points of this talk.

This slide introduces our research.
Our study aims to improve global routing in the physical design process.
Global routing creates approximate routing guides for all nets.
As shown in the right figure, it routes on coarse-grained grids called gcells and assigns metal layers.
Net topology construction is typically the first step performed in global routing tools.
This process determines the topology of each net, decomposing multi-pin nets into several 2-pin nets.
For example, the bottom figure shows pins that need to be connected as a single net.
The blue triangle represents the source pin driving the net. The black and red squares indicate non-critical and critical sinks, respectively.
Many existing global routing tools focused on minimizing wirelength (WL).
However, as chips have become faster, focusing solely on WL reduction can lead to longer pathlengths, negatively impacting critical path timing.
Therefore, we propose a practical net topology refinement that enhances chip timing characteristics with minimal WL increase.

Now, I'll compare the related works on net topology structures.

I'd like to compare different net topology generation methodologies using two example nets and explain our motivation.
Case 1 represents a net composed of non-critical pins, while Case 2 includes a critical pin.
The physical locations of pins are identical in both cases.

Here, we have RSMT, a commonly used algorithm.
RSMT stands for Rectilinear Steiner Minimum Tree.
Its goal is to connect pins with minimal wirelength.
Well-known algorithms include FLUTE and REST.
As mentioned earlier, this can lead to longer pathlengths, potentially degrading timing quality.

As chips became faster, RSSLT algorithms were developed to consider pathlength as well.
RSSLT stands for Rectilinear Steiner Shallow-Light Tree.
These algorithms aim to balance maximum pathlength and wirelength.
Notable examples include PD-II and SALT.
They ensure reasonable pathlengths for all pins.
However, in cases like Case 1, where pins have sufficient timing margin, these algorithms may reduce pathlengths unnecessarily. This can increase wirelength, negatively impacting power consumption and routing congestion.

Existing approaches have a limitation. They simplify the problem by ignoring the timing information of the pins.
Our approach balances pin delay and wirelength while considering the overall circuit timing. As a result, it not only improves timing but also achieves reasonable wirelength, enhancing power efficiency.
For instance, in Case 1, we favor the shortest wirelength. In contrast, for Case 2, we reduce the pathlength to the critical pin while minimally increasing wirelength.

Now, I will discuss our proposed methodology in detail.

This is the overall flow of our proposed methodology.
We iteratively improve the topology until timing enhancement in global routing stops.
In the first step of each iteration, we perform parasitic estimation and static timing analysis based on the previous global routing results.
This provides us with timing information for each pin.
If TNS improvement stops, we halt the iteration and move on to detailed routing.
Otherwise, we continue with the next steps.
In Step 2, Parallelization, we identify a set of nets whose topologies can be improved simultaneously. This allows us to extract timing-independent nets for parallel processing.
In Step 3, we apply our refinement algorithm to each net. I'll explain this in detail on the next slides.
Finally, we complete global routing by performing the remaining GR steps for the modified net topologies.

I'll now explain our proposed Timing-Driven Topology Refinement Method.
We initialize the initial Net Topology using traditional methodologies.
In the first step, we identify the critical path from the Net Topology.
In the example figure, the critical path is v0-v2-v4-v3.
In the second step, we remove each edge from the critical path.
The example shows removing the edge between v2 and v4.
This results in the tree being split into two parts, T1 and T2.
We then find edge candidates, which are the shortest rectilinear edges connecting T1 and T2.
For each candidate edge, we calculate the slack time gain after connecting it.
I'll explain the slack time gain formulation on the next slide.
Finally, we create a new tree by adding the edge with the highest gain.

The equation for calculating timing improvement for a topology perturbation η is as follows:
It consists of two terms.
The first term represents the gain obtained from changes in wire delay to the pins.
The second term accounts for the impact due to changes in cell delay.
I'll explain each of these terms in more detail.

The first term in the Gain equation represents the value obtained from changes in wire delay.
The Gain is calculated differently for each pin based on its slack value, which we obtain from the previous STA.
We place a higher value on delay improvements for pins with negative slack, that is, pins on the critical path.
To achieve this, we introduced a weighting function, shown in the right figure. This function discounts delay improvements for pins with large positive slack.
As the refinement converges towards increasing total gain, it ultimately reinforces the reduction of wire delay to critical pins.
One challenge here is accurately estimating wire delay changes due to topology perturbation at an early stage.
This is because we can't predict routing congestion-induced detours or layer assignments during the topology construction phase.
We estimated delay changes by modeling the topology as a pi-model and using Elmore delay.
The change in wire delay is estimated using the difference in Elmore delay before and after perturbation.
For this, we use the resistance and capacitance per unit length conditions of Metal 3.

The second term in the Gain equation represents the change in cell delay.
Cell delay can be affected by net topology perturbation because it's influenced by load capacitance.
Predicting this change is crucial as cell delay impacts the delay of all connected pins.
We assumed that wire capacitance is proportional to the wire length of the net topology.
The capacitance before perturbation, C_WL, is obtained from the previous parasitic estimation.
We can calculate the final cell delay change using the NLDM cell library.
This term prevents excessive increases in WL, ultimately reducing power consumption and routing congestion.

To perform Net topology refinement in parallel while preventing redundant modifications of timing-dependent nets, we proposed a parallelization step.
This begins by creating an adjacency matrix M from the netlist.
Next, we create a transitive closure matrix M* from the adjacency matrix.
We can calculate this using the Floyd-Warshall algorithm.
From M*, we find the maximal net subset where nets are not connected to each other.
We can solve this by formulating it as a maximal independent set problem.

Now I will describe experimental setup and results.

We implemented our methodology in OpenROAD, a state-of-the-art open-source physical design tool.
OpenROAD's Global router, FastRoute, operates in five stages.
We implemented our algorithm in the first stage, which is Steiner tree construction.
We utilized OpenROAD's infrastructure, such as parasitic extraction and static timing analysis, for timing calculations.
For our experiments, we used ASAP7nm and Nangate45nm process design kits, and tested on various open-source designs.
To compare our proposed method with others, we used two different algorithms.
These algorithms are built into OpenROAD by default: FLUTE for RSMT, and PD-II for RSSLT.

First, we evaluated the accuracy of our delay change estimation method.
We compared our predicted changes with those calculated by the STA engine after global routing.
The correlation coefficient for cell delay was 0.785, and for wire delay, it was 0.877.
We suspect the main sources of error are layer assignment and detours caused by routing congestion, which are difficult to predict at the early estimation stage.

This slide shows the experimental results after completing global routing.
We compared key PPA metrics like Worst Negative Slack (WNS), Total Negative Slack (TNS), and Power consumption.
The numbers in parentheses in the Design column indicate the technology node.
"Number of GRs" represents additional incremental global routing iterations due to our algorithm's loop. "Runtime" shows the extra runtime caused by our algorithm.
Our methodology demonstrates superior results compared to existing algorithms.
Compared to PD-II, we achieved a 25% reduction in WNS, 40% in TNS, and a 0.04% decrease in power consumption.
FLUTE shows similar power consumption to ours but has poor timing characteristics due to longer pathlengths.
The figure below shows a real example before and after applying our algorithm.
When applying our refinement algorithm, we can see significant improvement in delay to the critical pin (colored red) while minimizing the increase in total wire length.

This table shows the results after the post-route stage.
We compared WNS, TNS, Power, and total wirelength.
Our methodology showed good performance in most evaluations.
Compared to the PD-II algorithm, we improved WNS by 18%, TNS by 53%, and power consumption by 0.06%.
FLUTE had the lowest total wirelength but showed the worst timing results.
Our algorithm demonstrated the best characteristics in terms of power.
Considering that cell sizes often need to be increased or lower threshold processes used to improve timing characteristics, the potential power improvement effect could be even more significant.

To wrap up our presentation, I'd like to summarize the key points of our work.
We proposed a practical net topology refinement method that enhances timing robustness in the early stages of chip implementation.
Our approach focuses on improving circuit performance without compromising other critical factors.

We developed two key techniques:
First, a comprehensive net topology exploration that considers overall circuit timing.
Second, a parallel acceleration method for net topology tuning, which significantly speeds up the process.

An important feature of our method is its seamless integration with conventional global routing tools. This makes it practical and easy to adopt in existing design flows.
Our experimental results are particularly encouraging. We achieved significant improvements in circuit timing while maintaining, and in some cases even improving, power consumption and wirelength. This is a crucial balance in modern chip design.
In conclusion, our method successfully produced final chips with enhanced performance characteristics across the board. We believe this approach has the potential to make a real difference in the efficiency and performance of chip designs.
