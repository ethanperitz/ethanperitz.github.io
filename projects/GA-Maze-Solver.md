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


```python
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
```
```python
def differentStartGoal(start_index_1, start_index_2, goal_index_1, goal_index_2):
  '''
  Ensure that the start and the goal are different locations
  '''
  if goal_index_1 == start_index_1:
      if goal_index_2 == start_index_2:
        return False
  return True
```
```python
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
```
```python
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
```
```python
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
```
```python
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
```               
```python
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
```

```python
def printMaze(maze):
  '''
  Print the maze in a nice format
  Credit to ChatGPT for this code
  '''
  symbols = {1: "█", 0: " ", "S": "S", "G": "G"}
  for row in maze:
    print("".join(symbols.get(cell, "?") for cell in row))
```
### Generate Maze Solutions Greedily

```python
def manhattan(pos1, pos2):
  '''
  Computes Manhattan distance between two positions
  '''
  return abs(pos1[0]-pos2[0]) + abs(pos1[1]-pos2[1])
  

def verify_path(path, maze, start, goal):
  '''
  Verifies the solution path actually works
  '''
  if not path or path[0] != start or path[-1] != goal:
        return False
  for i in range(1, len(path)):
        x0, y0 = path[i-1]
        x1, y1 = path[i]
        # Ensure step is legal
        if abs(x1 - x0) + abs(y1 - y0) != 1:
            return False
        # Ensure no walls
        if maze[y1][x1] == 1:
            return False
  return True

def move(curr_position, visited, maze, fitness_score, path, goal):
  '''
  Unifying function that moves a solver to an open square, prioritizing the goal,
  followed by any unvisited space (greedily based on Manhattan distance), followed
  by any visited space (backtracking)
  '''
  possible_moves = [] #[up, down, right, left]
  #check up
  possible_moves.append(next_space(position = (curr_position[0] - 1, curr_position[1]), maze = maze, visited = visited))

  #check down
  possible_moves.append(next_space(position = (curr_position[0] + 1, curr_position[1]), maze = maze, visited = visited))

  #check right
  possible_moves.append(next_space(position = (curr_position[0], curr_position[1] + 1), maze = maze, visited = visited))

  #check left
  possible_moves.append(next_space(position = (curr_position[0], curr_position[1] - 1), maze = maze, visited = visited))

  #if goal is in adacent square, immediately go there
  if 'G' in possible_moves:
    index = possible_moves.index('G')

  #if unvisited space is available, greedily choose the one with closest manhattan distance to goal
  elif 0 in possible_moves:
    indices =[index for index, move in enumerate(possible_moves) if move == 0]
    min_distance = float('inf')
    best_index = None
    for index in indices:
      curr_dist = manhattan(next_move(curr_position, index), goal)
      if curr_dist == min_distance and random.random() < 0.5: #choose randomly between two equally good options
        best_index = index
      if  curr_dist < min_distance:
        min_distance = curr_dist
        best_index = index
    index = best_index

  #if only visited spaces are available
  elif 'V' in possible_moves:
    indices =[index for index, move in enumerate(possible_moves) if move == 'V']
    index = random.choice(indices)

  elif 'S' in possible_moves:
    index = possible_moves.index('S')

  else:
    fitness_score += 1
    return curr_position, visited, fitness_score, path

  #update position, visited, and fitness score
  new_position = next_move(curr_position, index)
  visited.add(new_position)
  path.append(new_position)
  fitness_score += 1
  return new_position, visited, fitness_score, path

def next_move(curr_position, index):
  '''
  Set the next move's coordinates.
  '''
  if index == 0:
      new_position = (curr_position[0] - 1, curr_position[1])
  elif index == 1:
      new_position = (curr_position[0] + 1, curr_position[1])
  elif index == 2:
      new_position = (curr_position[0], curr_position[1] + 1)
  elif index == 3:
      new_position = (curr_position[0], curr_position[1] - 1)
  return new_position


def next_space(position: tuple, maze: list, visited):
  '''
  Returns the type of cell in the intended position
  for determining the best moves to make.
  '''
  rows, cols = len(maze), len(maze[0])
  x, y = position
  if not (0 <= x < rows and 0 <= y < cols):
      return 1  # Treat out-of-bounds as wall
  if maze[position[0]][position[1]] == 1:
    return 1
  elif maze[position[0]][position[1]] == 0:
    if (position[0], position[1]) in visited:
      return 'V'
    return 0
  elif maze[position[0]][position[1]] == 'G':
    return 'G'
  elif maze[position[0]][position[1]] == 'S':
    return 'S'

def solution(start, goal, maze):
  '''
  Unifying function to find a solution through the maze
  '''
  visited = set()
  max_score = (len(maze)**2)*2
  curr_position = start
  fitness_score = 0
  visited.add(start)
  path = [start]
  while goal not in visited and fitness_score < max_score:
    new_position, visited, fitness_score, path = move(curr_position = curr_position,
                                                      visited = visited,
                                                      maze = maze,
                                                      fitness_score = fitness_score,
                                                      path = path,
                                                      goal = goal)
    curr_position = new_position
  return path, fitness_score

def printSolution(maze, path):
  '''
  Nicely print the solution with X's as the forged path
  '''
  solvedMaze = [row.copy() for row in maze]
  for x, y in path:
    if maze[x][y] == 0:
      solvedMaze[x][y] = 'X'
  symbols = {1: "█", 0: " ", "S": "S", "G": "G", "X":"X"}
  for row in solvedMaze:
      print("".join(symbols.get(cell, "?") for cell in row))
```

<iframe src="/assets/img/GA-Maze-Solution-Image.pdf" width="100%" height="600px">
    This browser does not support PDFs. Please download the PDF to view it: 
    <a href="/assets/img/GA-Maze-Solution-Image.pdf">Download PDF</a>.
</iframe>

### Fitness Scores, Decision Boundaries
```python
def decisionBoundaries(fitness_scores):
  normalized_scores = [1/score for score in fitness_scores]
  probability_model = [score/sum(normalized_scores) for score in normalized_scores]
  boundaries = [0]
  for index, score in enumerate(probability_model):
   boundaries.append(boundaries[index]+score)
  return boundaries

def selectParents(solution_paths, boundaries):
  selected_paths = []
  for _ in range(len(solution_paths)):
    r = random.random()
    for index,boundary in enumerate(boundaries[1:]):
      if r < boundary:
        selected_paths.append(solution_paths[index])
  return selected_paths
```

### Crossover
```python
def crossover(solution_paths, selected_paths, maze):
  children = []
  children_scores = []
  high_mutation = []
  for index, path1 in enumerate(solution_paths):
    path2 = selected_paths[index]

    #Heuristic 1:  Preserve very short paths into next generation with doubled mutation rate
    if len(path1) < 4 or len(path2) < 4:
      child = path1 if len(path1) < len(path2) else path2
      children.append(child)
      children_scores.append(len(child)-1)
      high_mutation.append(True)
      continue

    #Heuristic 2: Use manhattan distance bias to select near-optimal splice points
    min_distance = float('inf')
    best_pair = None
    for p1 in path1[:len(path1)//2]:
      for p2 in path2[len(path2)//2:]:
        dist = abs(p1[0]-p2[0]) + abs(p1[1]-p2[1])
        if dist < min_distance:
          min_distance = dist
          best_pair = (p1, p2)
    if best_pair is None:
      child = path1 if len(path1) < len(path2) else path2
      children.append(child)
      children_scores.append(len(child)-1)
      high_mutation.append(True)
      continue

    point1, point2 = best_pair

    #validate that the best pair has valid indices
    try:
      idx1 = path1.index(point1)
      idx2 = path2.index(point2)
    except ValueError:
      #default to Heuristic #1
      child = path1 if len(path1) < len(path2) else path2
      children.append(child)
      children_scores.append(len(child)-1)
      high_mutation.append(True)
      continue
    bridge, new_score = solution(start=point1, goal=point2, maze=maze)
    if bridge:
      child = path1[:idx1] + bridge + path2[idx2+1:]
    else:
      child = path1
    children.append(child)
    children_scores.append(len(child) - 1)
    high_mutation.append(False)
  return children, children_scores, high_mutation
```

### Mutation
```python
def mutate(children, scores, high_mutation, maze, rate):
  mutated_children = []
  for index, child in enumerate(children):
    #preserve best
    if scores[index] == min(scores):
      mutated_children.append(child)
      continue
    if high_mutation[index]:
      adjusted_rate = rate*1.2 #increase mutation rate by 50%
    else:
      adjusted_rate = rate
    r = random.random()
    if r < adjusted_rate:
      #choose random starting and ending points, build path between them
      mutate_start_index = random.randint(1, len(child)//2)
      mutate_end_index = random.randint(len(child)//2, len(child) - 2)
      if mutate_start_index >= mutate_end_index:
        mutated_children.append(child)
        continue
      mutate_start = child[mutate_start_index]
      mutate_end = child[mutate_end_index]
      bridge, _ = solution(start = mutate_start, goal = mutate_end, maze = maze)
      if not bridge or not isinstance(bridge, list) or bridge[0] != mutate_start or bridge[-1] != mutate_end:
        mutated_children.append(child)
        continue
      child = child[:mutate_start_index] + bridge + child[mutate_end_index+1:]
      #print("mutation!") #helpful for tracking frequency of mutations
    mutated_children.append(child)

  return mutated_children
```

### Preserve Best Solutions
```python
def select_elites(population, scores, k):
    # Return the top-k elite paths and their scores
    elite_indices = sorted(range(len(scores)), key=lambda i: scores[i])[:k]
    return [population[i] for i in elite_indices], [scores[i] for i in elite_indices]
```

### Graph Convergence
```python
def fitnessScoreGrapher(average_fitness_scores, min_fitness_scores):
  '''
  Graphs fitness score for each generation.
  In black, graph average fitness score of the generation.
  In red, graph minimum fitness score of the generation.
  '''
  xvals = list(range(0, len(average_fitness_scores)))
  yvals = average_fitness_scores
  y2vals = min_fitness_scores
  plt.plot(xvals, yvals, linestyle = '-', marker = '', color = 'black', label = "Average Value")
  plt.plot(xvals, y2vals, linestyle = '--', marker = '', color = 'red', label = "Min Value")
  plt.plot(list(range(0, len(average_fitness_scores))), [min(min_fitness_scores)]*len(average_fitness_scores), linestyle = '-', marker = '',color= 'blue', label = "Best Value")
  plt.xlabel("Generation")
  plt.ylabel("Fitness Score (Path Length)")
  plt.title("Convergence Plot for Fitness Scores")
  plt.grid(True)
  plt.legend()
  plt.xlim(0, max(xvals)+1)
  plt.ylim(min(y2vals)-5, max(yvals)+5)
  plt.show()
```



