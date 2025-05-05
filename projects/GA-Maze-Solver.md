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
Making the maze was a fun challenge.  Essentially, the steps I followed were to generate an nxn array of 0's, randomly select two spots along the border to represent the start and goal, find a possible pathway from start to goal, build walls around the possible pathway, then build out more dead ends.  The following function structure made this possible:
<code>
def findBorder(dimension):
  '''
  Finds random locations along the border of the maze for start and goal
  '''
  index1 = random.randint(0, dimension-1)
  if index1 == 0 or index1 == dimension-1:
    index2 = random.randint(0, dimension-1)
  else:
    index2 = random.choice([0, dimension - 1])
  return index1, index2
</code>
<code>
def differentStartGoal(start_index_1, start_index_2, goal_index_1, goal_index_2):
  '''
  Ensure that the start and the goal are different locations
  '''
  if goal_index_1 == start_index_1:
      if goal_index_2 == start_index_2:
        return False
  return True
</code>
<code>
def findPath(maze):
  '''
  Use depth-first search to find a pathway from start to goal.
  '''
  dimension = len(maze)
  path = []
  visited = set()
  #find goal
  goalx, goaly = 0, 0
  while('G' not in maze[goalx]):
    goalx += 1
  while maze[goalx][goaly] != 'G':
    goaly += 1
  goal = (goalx, goaly)
  #find start
  x, y = 0, 0
  while 'S' not in maze[x]:
    x += 1
  while maze[x][y] != 'S':
    y += 1
  start = (x, y)
  #depth-first-search for maze path
  def dfs(x, y):
    '''
    Credit to ChatGPT for help with DFS algorithm
    '''
    if (x, y) == goal:
      path.append((x, y))
      return True
    visited.add((x, y))
    directions = [(0,1), (1,0), (0,-1), (-1,0)]
    random.shuffle(directions)
    for dx, dy in directions:
      nx, ny = x + dx, y + dy
      if 0 <= nx < dimension and 0 <= ny < dimension:
        if (nx, ny) not in visited:
          if dfs(nx, ny):
            path.append((x, y))
            return True
    return False
  if dfs(start[0], start[1]):
    return path[::-1]
</code>
<code>
def wallsAroundPath(maze, path):
  '''
  Set all squares around the solution path of a maze to 1 (a wall)
  '''
  dimension = len(maze)
  for x, y in path:
    for dx, dy in [(-1,0),(1,0),(0,-1),(0,1)]:
      nx, ny = x+dx, y+dy
      if 0 <= nx < dimension and 0 <= ny < dimension and (nx, ny) not in path:
        maze[nx][ny] = 1
  return maze
</code>
<code>
def wallsOutsidePath(maze, path):
  '''
  Scatter walls in random places around the maze, but outside the solution path
  '''
  dimension = len(maze)
  x, y = 0, 0
  for i in range(dimension):
    for j in range(dimension):
      if (i, j) not in path:
        maze[i][j] = random.choice([0,0,0,1,1,1,1])
  return maze
</code>
<code>
def add_branches(maze, path, num_branches=10, max_branch_length=4):
    '''
    Add branches from random spots on the solution path to make dead ends
    '''
    n = len(maze)
    path_set = set(path)
    for _ in range(num_branches):
        start = random.choice(path)
        x, y = start
        branch = []
        for _ in range(random.randint(1, max_branch_length)):
            directions = random.sample([(-1,0), (1,0), (0,-1), (0,1)], 4)
            for dx, dy in directions:
                nx, ny = x + dx, y + dy
                if (0 <= nx < n and 0 <= ny < n and
                    (nx, ny) not in path_set and maze[nx][ny] != 0):
                    maze[nx][ny] = 0  # carve a path
                    branch.append((nx, ny))
                    x, y = nx, ny
                    break
            else:
                break  # no directions left to expand
    return maze
</code>
<code>
def buildMaze(dimension=10):
  '''
  Unifying function that builds the maze.
  '''
  maze = [[0 for _ in range(dimension)] for _ in range(dimension)]
  while True:
  #start and goal of maze can be anywhere on the border
    start_index_1, start_index_2 = findBorder(dimension)
    goal_index_1 , goal_index_2 = findBorder(dimension)
    if differentStartGoal(start_index_1, start_index_2, goal_index_1, goal_index_2):
      maze[start_index_1][start_index_2] = "S"
      maze[goal_index_1][goal_index_2] = "G"

      #find a path through the maze
      solution = findPath(maze)
      #add walls
      maze = wallsAroundPath(maze, solution)
      maze = wallsOutsidePath(maze, solution)
      maze = add_branches(maze, solution, num_branches=6, max_branch_length= 4)
      return maze, (start_index_1, start_index_2), (goal_index_1, goal_index_2)
</code>
<code>
def printMaze(maze):
  '''
  Print the maze in a nice format
  Credit to ChatGPT for this code
  '''
  symbols = {1: "█", 0: " ", "S": "S", "G": "G"}
  for row in maze:
    print("".join(symbols.get(cell, "?") for cell in row))
</code>


### Generate Maze Solutions Greedily

<iframe src="/assets/img/GA-Maze-Solution-Image.pdf" width="100%" height="600px">
    This browser does not support PDFs. Please download the PDF to view it: 
    <a href="/assets/img/GA-Maze-Solution-Image.pdf">Download PDF</a>.
</iframe>

### Fitness Scores, Decision Boundaries


### Crossover


### Mutation


### Preserve Best Solutions


### Graph Convergence




