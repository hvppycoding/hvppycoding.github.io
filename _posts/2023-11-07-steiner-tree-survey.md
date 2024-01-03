---
title: "A Survey on Steiner Tree Construction and Global Routing for VLSI Design"
excerpt: ""
date: 2023-11-07 04:00:00 +0900
header:
  overlay_image: /assets/images/unsplash-Umberto.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Umberto**](https://unsplash.com/@umby) on [**Unsplash**](https://unsplash.com/)"
categories:
  - VLSI
---

[LINK](https://ieeexplore.ieee.org/document/9057662)

## List of Abbreviation

| Abbreviation | Full Form |
|--------------|-----------|
| ACO | Ant Colony Optimization |
| AMR | Adaptive Maze Routing |
| BFS | Breadth First Search |
| BGA | Batched Greedy Algorithm |
| BLMR | Bounded-Length Maze Routing |
| CAD | Computer-Aided Design |
| CMOS | Complementary Metal Oxide Semiconductor |
| DFS | Depth First Search |
| DR | Detailed Routing |
| DT | Delaunay Triangulation |
| EDA | Electronic Design Automation |
| FLUTE | Fast Lookup Table Estimation |
| FST | Full Steiner Tree |
| GR | Global Routing |
| HTS | Hybrid Transformation Strategy |
| IC | Integrated Circuit |
| ILP | Integer Linear Programming |
| LRSMT | Length-Restricted Steiner Minimum Tree |
| MAD | Modified Algorithm Dijkstra |
| MDSV | Multiple Dynamic Supply Voltage |
| MST | Minimum Spanning Tree |
| MSV | Multiple Supply Voltage |
| NDT | Non-Delaunay Triangulation |
| OAFST | Obstacle-Avoiding Full Steiner Tree |
| OARG | Obstacle-Avoiding Routing Graph |
| OARSMT | Obstacle-Avoiding Rectilinear Steiner Minimum Tree |
| OARST | Obstacle-Avoiding Rectilinear Steiner Tree |
| OASG | Obstacle-Avoiding Spanning Graph |
| OASMT | Obstacle-Avoiding Steiner Minimum Tree |
| OAVG | Obstacle-Avoiding  Voronoi Graph |
| OAXSMT | Obstacle-Avoiding X-architecture Steiner Minimum Tree |
| PD | Physical Design |
| PDAR | Power Domain-Aware Routing |
| POWV | Potential Optimal Wirelength Vector |
| PSO | Particle Swarm Optimization |
| RSMT | Rectilinear Steiner Minimum Tree |
| SADP | Self Aligned Double Patterning |
| SMMT | Steiner Min-Max Tree |
| SMT | Steiner Minimum Tree |
| VLSI | Very Large-Scale Integration |
| XSMT | X-architecture Steiner Minimum Tree |

## I. INTRODUCTION

The whole process of PD is usually divided into serveral phases: partition, floorplanning, layout, placement, routing. One of the most ciritcal steps is Global Routing (GR), a stage where signal nets are connected coarsely under a given placement so that wire/via spaces are allocated to each signal net.  
Given points on a plane, an SMT connects these points through some extra points (Steiner points) to achieve a minimum total length. An SMT is usually used in initial topology creation for noncritical nets. For timing critical nets, wirelength minimization is not the only goal. However, most nets are noncritical in routing phase and an SMT gives the most desirable route of such a net. Thus, SMT is often used as accurate estimations for congestion and wirelength during floorplanning and placement.  
Nowadays, timing and power cconsumption issues have become increasingly prominent. More and more constraints have emerged in GR and Steiner tree construction, which leads to the fact that the traditional routing algorithms or SMT algorithms cannot adapt to such multi-objective tasks well. Thus, GR and SMT algorithms face new challenges.  

## II. BASIC ROUTING STRATEGIES

### A. MULTI-PIN NET DECOMPOSITION

Multi-pin net decomposition is often used in the GR algorithms. A net with more than two pins is often decomposed into two-pin subnets, and then point-to-point routing for each subnet is performed in some order.

### B. MAZE ROUTING

A subproblem often encountered in GR is finding the shortest path connecting the two pins in the presence of blockages. The most widely known solution to this problem is the <span class="custom-highlight" markdown="1">Lee's algorithm</span>, that is, maze routing algorithm. Maze routing is a grid-based search algorithm that has long been considered a brute-force algorithm since it allows all possible paths. Johann and Reis used the A\* algorithm to improve the Lee's algorithm to speed up the convergence.  

### C. MULTI-SOURCE AND MULTI-SINK MAZE ROUTING

Multi-source and multi-sink maze routing evolved from traditional maze routing, which is an improvement over traditional routing, considering more and better routing paths.

### D. STEINER TREE CONSTRUCTION

Both the maze routing and the line probe approach are designed for the connection of a two-pin net. However, in practical design, routing problems often encounter some nets with more than two pins. A common approach to deal with multi-pin nets is to decompose them into a set of two-pin nets. One way to perform this decomposition is to first construct an MST on these pins and then perform maze routing on each pair of pins corresponding to each edge of the MST. The total length of the routing tree can usually be reduced by adding additional points(steiner points) outside of given pins and constructing an MST on all these nodes.  

### E. PATTERN ROUTING

Maze routing methods such as Dijkstra's algorithm and A\* search can be used to guarantee a shortest path between two pins. In practice, many nets' routes are not only short but also have few bends. Pattern routing uses a predefined pattern for routing the two-pin nets, which limits point-to-point connections to a small number of fixed shapes.

### F. MONOTONIC ROUTING

Assuming that the given source point is at the bottom left of the receiving point, monotonic routing can only move up or to the right from the grid point.

### G. INTEGER LINEAR PROGRAMMING (ILP) ROUTING

