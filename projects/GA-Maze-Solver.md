---
layout: page
title: Genetic Algorithm – Solving a Maze
permalink: /projects/GA-Maze-Solver/
---

# A Genetic Algorithm for Optimizing Solution Paths Through a 2-D Maze

Combining a greedy heuristic approach with generational adaptation for solving long mazes efficiently

## Problem Statement

We have a randomly generated nxn maze with one start point and one goal point (randomly determined along the edges) and at least one solution pathway.  Find the shortest path from the start point to the goal point.  

## Interesting Aspects of this Problem

This was a challenging and really cool project from start to finish.  This is similar to the knapsack problem where a greedy solution (I used a Manhattan distance heuristic) tends to work really well.  A genetic algorithm allows us to capitalize on the effectiveness of the greedy solution strategy by applying it to multiple generations, allowing for a cool interplay between stochasticism and determinism.  

## Algorithm Overview

1.  Create a sample of k solutions through the maze. 
  - Beginning at start point, choose move based on minimizing Manhattan distance to goal.  In the case of ties, break them randomly (this is super important to preserve some stochastic methods in the algorithm)
2.  Score each of the k solutions by the number of steps needed to reach the goal. 
3.  Select k solutions (with replacement) from the k solutions with weighted probabilities, with higher weights afforded to more efficient solutions.
4.  Perform crossover on the original sample of solutions (parent #1) with its corresponding member of the "more fit" solutions generated in step 3 (parent #2).  
  -Start by choosing a starting point along the first half of parent #1.  I did this heuristically using the same Manhattan distance method.  We choose the point along the first half of parent #1 that is closest to the goal.  Then, check to see if that same point is in parent #2.  If so, splice them at that point (or the last occurrence of that point if it shows up multiple times).  If not, find a random point along parent #2's second half and greedily forge a path from the splice point to that point. 
5.  Mutate (using a fairly low mutation rate) such that some of the new paths are randomly spliced with new solutions generated from the maze.  
6.  Preserve the best p% of solutions as "elites" that will certainly carry on to the next generation(s).
7.  Repeat from Step 2 for g generations, using the children as the set of solutions.  

## Functions in Python

### Make the Maze



### Generate Maze Solutions Greedily



### Fitness Scores, Decision Boundaries


### Crossover


### Mutation


### Preserve Best Solutions


### Graph Convergence



## Example Maze
<code>
█ ███   █████    █  ██ ████  ████  ██   
 █ █ ████ █ ██ ████ ████ ████  ██ ███ █ 
█  ███ ███     █  ██████  █   ██ █ █  █ 
█  █  █   █ █ ███ ████  █ █ █  █ █    █ 
 ██ █  █████ ████████ █  █ █████    ██  
█ █████  █   ██ █ ███ ███ █ █ ██  █ ██ █
███  ██   █  ████  █████     ██ ██  █   
   ██ █    █    █ █        █ ███    █  █
██ █ █    █ ███████ █ █ ███ ███ ██ █ ███
  ██ █ █    █ █  ███ █ █ ████    █   █  
  █ █ █  ██████  ██  █ █ █  █ █  █   ███
███ █ ██ ███ ██   █  ██  █   ██  █   █ █
█████ ██ █ ███     █   ██ █             
█  █  █   ██ ██      █          █ █    █
███ ██  █   █  █  █  ████   █    ██     
 █ ███ █ █    █ █               █    █ G
█   █ ███ █   █ █ █  █       █     █   █
███  ██    ██ █ █              █  ██   █
█ ███ █ ███ ██ █  ██                  ██
 ██ █    ███ ██ ███         █         ██
█  █ ████ █ █ ███       ████     ██    █
 █████ █████ █  █         ██     █ █    
  ██ ██████  █ ██         █       ██    
 ████ ██  █  ██ ██   █    █          █  
  █ ███████ █ ██       █  █           █ 
█ ██  ██  ███ █    ██          ██       
██  █   █ █ ██     █    █               
████    █ █  ██          █    ██       █
 ██     █████   █                       
█ ██   ██  █          █     █ █     ██  
█ ██   █      ██  █         █ █         
█  █     ██  █  █   ██     █         █  
███   █        ███   █     ██   █  █    
 █                     ██  ██ █         
   █                  █    █    ██  ██  
██                     █   ██         █ 
                  █        █          ██
S         █                ██ █ █  █    
   █    █   █ █  █ █               █    
     █  █ █        ███        █  █  ██ █
</code>

## Solution Found by Algorithm Within 100 Generations
<code>
█ ███   █████    █  ██ ████  ████  ██   
 █ █ ████ █ ██ ████ ████ ████  ██ ███ █ 
█  ███ ███     █  ██████  █   ██ █ █  █ 
█  █  █   █ █ ███ ████  █ █ █  █ █    █ 
 ██ █  █████ ████████ █  █ █████    ██  
█ █████  █   ██ █ ███ ███ █ █ ██  █ ██ █
███  ██   █  ████  █████     ██ ██  █   
   ██ █    █    █ █        █ ███    █  █
██ █ █    █ ███████ █ █ ███ ███ ██ █ ███
  ██ █ █    █ █  ███ █ █ ████    █   █  
  █ █ █  ██████  ██  █ █ █  █ █  █   ███
███ █ ██ ███ ██   █  ██  █   ██  █   █ █
█████ ██ █ ███     █   ██ █             
█  █  █   ██ ██      █          █ █    █
███ ██  █   █  █  █  ████   █    ██ XXX 
 █ ███ █ █    █ █           XXXX█XXXX█XG
█   █ ███ █   █ █ █  █   XXXX█ XXX █   █
███  ██    ██ █ █       XX     █  ██   █
█ ███ █ ███ ██ █  ██  XXX             ██
 ██ █    ███ ██ ███   X     █         ██
█  █ ████ █ █ ███    XX ████     ██    █
 █████ █████ █  █    X    ██     █ █    
  ██ ██████  █ ██   XX    █       ██    
 ████ ██  █  ██ ██  X█    █          █  
  █ ███████ █ ██  XXX  █  █           █ 
█ ██  ██  ███ █   X██          ██       
██  █   █ █ ██   XX█    █               
████    █ █  ██XXX       █    ██       █
 ██     █████XXX█                       
█ ██   ██  █XX        █     █ █     ██  
█ ██   █    X ██  █         █ █         
█  █     ██ X█  █   ██     █         █  
███   █  XXXX  ███   █     ██   █  █    
 █      XX             ██  ██ █         
   █    X             █    █    ██  ██  
██    XXX              █   ██         █ 
  XXXXX           █        █          ██
SXX       █                ██ █ █  █    
   █    █   █ █  █ █               █    
     █  █ █        ███        █  █  ██ █
</code>
