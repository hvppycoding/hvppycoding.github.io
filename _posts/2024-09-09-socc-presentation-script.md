---
title: "SOCC2024"
excerpt: ""
date: 2024-09-09 12:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
mathjax: true
---

<audio controls loop>
  <source src="{{site.baseurl}}/assets/media/2024-09-09-socc-presentation-script.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>

![20240909-151915]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-151915.png){: .align-center}  

Hello everyone. My name is Jayoung Yang, a Master's student at Seoul National University.  
Today, I will talk about our work on "Improving Timing Quality Through Net Topology Optimization in Global Routing".  

![20240909-151944]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-151944.png){: .align-center}  

This slide outlines my presentation.  
I'll begin with an introduction to our work.
And then I will explain the background on net topology structures.  
Then, I'll describe our proposed methodology and experimental results.  
Finally, I'll summarize the key points of this talk.  

![20240909-151952]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-151952.png){: .align-center}  

This slide introduces our research.  
Our study aims to improve global routing in the physical design process.  
Global routing creates coarse routing guides for all nets.  
As shown in the right figure, it generates guides on coarse-grained grids called gcells and assigns metal layers.  
Net topology construction is typically the first step performed in many global routing tools.  
This step determines the topology for each net, which is then used to decompose multi-pin nets into several 2-pin nets.  
For example, the bottom figure shows pins that need to be connected as a single net.  
The blue triangle represents the source pin driving the net.  
The black and red squares indicate non-critical and critical sinks, respectively.  
Many existing global routing tools focused on minimizing wirelength (WL).  
However, focusing solely on total wirelength can lead to longer pathlengths to some pins.  
This can negatively impact critical path timing.  
Therefore, we propose a practical net topology refinement that enhances chip timing quality with minimal wirelength increase.  

![20240909-152000]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152000.png){: .align-center}  

Now, I'll compare the related works on net topology structures.  

![20240909-152051]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152051.png){: .align-center}  

I'd like to compare different net topology construction methodologies using two example nets and explain our motivation.  
Case 1 represents a net composed of non-critical pins, while Case 2 includes a critical pin.  
The physical locations of pins are identical in both cases.  

![20240909-152059]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152059.png){: .align-center}  

Here, we have RSMT, one of the most common algorithms in this area.  
RSMT stands for Rectilinear Steiner Minimum Tree.  
Its goal is to connect pins with minimal wirelength.  
FLUTE and REST are notable works in this field.  
As mentioned earlier, this can lead to longer pathlengths, potentially degrading timing quality.  

![20240909-152105]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152105.png){: .align-center}  

As chips became faster, RSSLT algorithms were developed to take account of pathlength as well.  
RSSLT stands for Rectilinear Steiner Shallow-Light Tree.  
These algorithms aim to balance maximum pathlength and wirelength.  
PD-II and SALT are notable works in this field.  
They ensure reasonable pathlengths for all pins.  
However, in cases like Case 1, where pins have sufficient timing margin, these algorithms may reduce pathlengths unnecessarily.  
This can increase wirelength, which in turn negatively impacts power consumption and routing congestion.  

![20240909-152111]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152111.png){: .align-center}  

Existing approaches have a limitation.  
They simplify the problem by ignoring the timing information of the pins.  
Our approach balances pin delay and wirelength while considering the timing of entire circuit.  
As a result, it not only improves timing but also achieves reasonable wirelength.  
For instance, in Case 1, we favor the shortest wirelength.  
On the other hand, for Case 2, we reduce the pathlength to the critical pin with minimal increase in wirelength.  

![20240909-152116]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152116.png){: .align-center}  

Now, I will discuss our proposed methodology in detail.  

![20240909-152122]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152122.png){: .align-center}  

This is the overall flow of our proposed methodology.  
We iteratively improve the topologies until timing enhancement stops.  
In the first step of each iteration, we perform parasitic estimation and static timing analysis based on the previous global routing results.  
This provides us with timing information for each pin.  
If TNS improvement stops, we finish the iteration and move on to detailed routing.  
Otherwise, we continue with the next steps.  
In Step 2, Parallelization, we identify a set of nets that can be refined simultaneously.  
This step, we extract timing-independent nets for parallel processing.  
In Step 3, we apply our refinement algorithm to each net.  
I'll explain this in detail on the next slides.  
Finally, we complete remaining GR steps for the refined topologies.  

![20240909-152132]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152132.png){: .align-center}  

I'll now explain our proposed Timing-Driven Topology Refinement Method.  
We initialize the net topologies using traditional methodologies.  
In the first step, we identify the critical paths based on the previous STA results.  
In the example figure, the critical path is v0-v2-v4-v3.  
In the second step, we remove each edge from the critical path.
The example shows removing the edge between v2 and v4.
This splits the tree into two parts, T1 and T2.  
We then find edge candidates, which are the shortest rectilinear edges connecting T1 and T2.  
For each candidate edge, we calculate the slack time gain after connecting it.  
I'll explain the slack time gain formulation on the next slide.  
Finally, we create a new tree by adding the edge with the highest gain.  

![20240909-152144]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152144.png){: .align-center}  

The formula for calculating timing improvement for a topology perturbation EE-tah(η) is as follows:  
It consists of two terms.  
The first term represents the gain obtained from changes in wire delay to the pins.  
The second term accounts for the impact due to changes in cell delay.  
I'll explain each of these terms in more detail.  

![20240909-152201]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152201.png){: .align-center}  

The first term in the gain equation is obtained from changes in wire delay.  
The gain is calculated for each pin based on its slack, which we obtain from the previous STA.  
We place a higher score on delay reductions for pins with negative slack.  
To achieve this, we introduced a weighting function, shown in the right figure.  
This function discounts delay reductions for pins with large positive slack.  
As the refinement keeps improving total gain, it reinforces the reduction of wire delay to critical pins.  
A key challenge here is estimating wire delay changes accurately in the early stages.  
This is because we can't predict detours induced by routing congestion or layer assignments.  
We estimated delay changes by modeling the topology as a pi-model and using Elmore delay.  
We estimate wire delay changes using Elmore delay differences before and after perturbation.  
For delay calculations, we use the resistance and capacitance per unit length of Metal 3.  

![20240909-152207]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152207.png){: .align-center}  

The second term in the Gain equation represents the change in cell delay.  
Cell delay is affected by net topology perturbation because it's influenced by load capacitance.  
Predicting cell delay is crucial as the cell delay impacts all connected pins.  
We assumed that wire capacitance is proportional to the wire length of the net topology.  
The capacitance before perturbation, C_WL, is obtained from the previous parasitic estimation.  
We can calculate the change in cell delay using the NLDM cell library.  
As a result, This cell delay term prevents excessive increases in wirelength.  

![20240909-152215]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152215.png){: .align-center}  

We proposed a parallelization step to perform net topology refinement in parallel.  
This process prevents modifications of timing-dependent nets.  
This begins by creating an adjacency matrix M from the netlist.  
Next, we create a transitive closure matrix M\* from the adjacency matrix.  
We can derive this using the Floyd-Warshall algorithm.  
From M\*, we find the maximal net subset where nets are not connected to each other.  
We solved this problem by formulating it as a maximal independent set problem.  

![20240909-152221]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152221.png){: .align-center}  

Now I will describe experimental setup and results.

![20240909-152241]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152241.png){: .align-center}  

We implemented our methodology in OpenROAD, a state-of-the-art open-source physical design tool.  
OpenROAD's Global router, FastRoute, operates in five stages.  
We implemented our algorithm in the first stage, Steiner tree construction.  
We utilized OpenROAD's infrastructure for timing calculations, such as parasitic extraction and static timing analysis.  
For our experiments, we used ASAP7nm and Nangate45nm process design kits, and tested on various open-source designs.  
To compare our proposed method with others, we used two different algorithms.  
These algorithms are built into OpenROAD by default: FLUTE for RSMT, and PD-II for RSSLT.  

![20240909-152247]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152247.png){: .align-center}  

First, we evaluated the accuracy of our delay change estimation method.  
We compared our predicted changes with those calculated by the STA engine after global routing.  
The correlation coefficient for cell delay was 0.785, and for wire delay, it was 0.877.  
We suspect the main sources of error are layer assignment and detours caused by routing congestion.  

![20240909-152252]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152252.png){: .align-center}  

This slide shows the experimental results after completing global routing.  
We compared key PPA metrics like Worst Negative Slack (WNS), Total Negative Slack (TNS), and Power consumption.  
The numbers in parentheses in the Design column indicate the technology node.  
"Number of GRs" represents additional incremental global routing iterations due to our algorithm's loop.  
"Runtime" shows the extra runtime caused by our algorithm.  
Our methodology demonstrates superior results compared to existing algorithms.  
Compared to PD-II, we achieved a 25% reduction in WNS, 40% in TNS, and a 0.04% reduction in power consumption.  
FLUTE shows similar power consumption to ours but has poor timing metrics due to longer pathlengths.  
The figure below shows an example of our refinement method.  
We can see significant improvement in delay to the critical pin, which is colored red, without a large increase in wirelength.  

![20240909-152258]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152258.png){: .align-center}  

This table shows the results after the post-route stage.  
We compared WNS, TNS, Power, and total wirelength.  
Our methodology showed good performance in most evaluations.  
Compared to the PD-II algorithm, we improved WNS by 18%, TNS by 53%, and power consumption by 0.06%.  
FLUTE had the lowest total wirelength but showed the worst timing results.  
Our algorithm showed the best performance in terms of power.  
Considering that cell sizes need to be increased to improve timing metric, the potential power reduction effect could be even more significant.  

![20240909-152304]({{site.baseurl}}/assets/images/2024-09-09-socc-presentation-script/20240909-152304.png){: .align-center}  

To wrap up, let me highlight the key points of our work:  
We developed a practical method to refine net topology, which enhances timing robustness early in chip design.  
Our approach uses two main techniques: comprehensive topology exploration and parallel acceleration.  
Our method works smoothly with existing global routing tools, making it easy to adopt.  
Our experiments showed great results.  
We significantly improved circuit timing without compromising power consumption or wirelength.  
In the end, we produced chips with better overall performance.  
We believe this approach can really boost the efficiency and performance of chip designs.  
