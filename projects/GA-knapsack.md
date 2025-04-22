---
layout: page
title: Genetic Algorithm – Knapsack Problem
permalink: /projects/genetic-algorithm-knapsack/
---
<script type="text/javascript" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

# A Genetic Algorithm for Solving the 0-1 Knapsack Problem

A classic problem in computer science, the 0-1 Knapsack is a great way to test out different implementations of a genetic algorithm, then comapre them with a simpler greedy algorithm, which tends to solve this problem quite well.  

## Problem Statement

Given a set of items with known weights and values, we want to pack a knapsack that maximizes the value of the items inside without exceeding the knapsack's maximum capacity.  

## Algorithm Overview

Problems like this can be tricky to find optimal solutions.  As the number of items in the population increases, the time complexity of the problem increases exponentially.  

Genetic algorithms allow us to start with potential solutions at random, then iteratively select the best (or "fittest") solutions across many generations.  As the name implies, this behaves very much like evolution.  The chief ingredients are:

1.  A metric for assessing the "fitness" of a particular solution.
2.  A method for creating the next generation that is biased towards fitter solutions.
3.  An occasional injection of randomness (or mutation) so that we don't converge on a less-than-optimal solution too soon.

My algorithm for this problem included the following steps:

1.  Initialize the algorithm with randomly selected groups of items (i.e. potential knapsack solutions) from the population (more later about how to do this sensibly)
2.  Score each knapsack solution.  If the weight of the items is within capacity, its score is simply the value of the items inside.  If the items are overweight, we successively remove items based on their value-to-weight ratio until the items are within capacity.
3.  Use the scores to create a probability model.  We want the fittest solutions to be more likely to breed into the next generation.
4.  Use the probability model to randomly select (with replacement) from the current group of knapsacks.  The fittest ones are more likely to be selected.
5.  We now have a set of original knapsacks and a set of "fitter" knapsacks.  These sets are the same size.  We pair them together for breeding.
6.  Use a crossover model to combine items from each parent pair:
      a.  We first randomly select b% of the items from the original knapsack, where b is a tuned hyperparameter.  A smaller b selects less from the original knapsack and more from the "fitter" knapsack mate.
      b.  Then, we randomly select items from the "fitter" parent until the knapsack fills up or exceeds the weight limit.
      c.  If the weight limit is exceeded, greedily remove the worst value-to-weight items from the knapsack until the weight constraint is satisfied.
      d.  Repeat for each parent pair.
7.  Use a mutation model for the new set of "children" knapsacks:
      a.  Set a mutation rate r between 0 and 1 and generate a random real number x between 0 and 1.
      b.  If $x < r^3$, we swap out 10 items randomly with items from the population.
      c.  Otherwise, if $x < r^2$, we swap out 3 items.
      d.  Otherwise, if $x < r$, we swap out 1 item.
      e.  Otherwise, no swaps occur.
8.  Search through the children knapsacks for the current best solutions.  Be sure to keep them.
9.  Repeat for many generations, using the new children as the next set of knapsacks to be scored in step 2.

## Algorithm Implementation

### 1.  Initialize the algorithm with randomly selected groups of items

<code>
```python
def inclusionRates(pop, max_capacity, sample_size):
  '''
  Generates a distribution of inclusion rates based on binomial probabilities,
  where p is the probability that an item will be selected for the knapsack,
  and p is the E(Items that will fit) / N, the population size (which is itself
  Capacity/Total Weight)

  Then, the distribution is binomial with mean population size * p and variance
  population size * p * (1-p)

  We want to bias against knapsacks that are overweight, so we will generate
  inclusion rates that will select from about 2 standard deviations below the
  expected number of items up to one half of a standard deviation above
  the expected number of items
  '''
  pop_weight_total = sum([b[0] for b in pop])
  p = max_capacity/pop_weight_total
  rates = []
  while len(rates) < sample_size:
    rate = np.random.normal(loc = 0.9 * p, scale = 0.05 * p)
    rates.append(min(max(rate, 0), p))
  return rates

def getInitialChromosomes(sample_size, pop_size, inclusion_rates):
  '''
  Uses distributed inclusion rates to make lists of chromosomes,
  sets of 0's and 1's equal to the population size, where 0 indicates an 
  item that should not be packed into the knapsack, and 1 indidicates an
  item that should be packed.  
  '''
  chromosomes = []
  for inclusion_rate in inclusion_rates:
    included_items = [random.randint(1,100) for _ in range(pop_size)]
    threshold = int(100*inclusion_rate)
    for index, item in enumerate(included_items):
      if item <= threshold:
        included_items[index] = 1
      else:
        included_items[index] = 0
    chromosomes.append(included_items)
  return chromosomes

  def pack(pop, chromosomes_list):
  '''
  Uses the chromosome lists to "pack" a knapsack - taking the indexed
  items from the population
  '''
  knapsacks = []
  for chromosomes in chromosomes_list:
    knapsack = [pop[index] for index, chromosome in enumerate(chromosomes) if chromosome == 1]
    knapsacks.append(knapsack)
  return knapsacks
</code>

### 2.  Score each knapsack solution.

<code>
```python
  def weight(knapsack):
  '''
  compute the total weight of a collection of items in knapsack
  '''
  return sum([item[0] for item in knapsack])


def value(knapsack):
  '''
  compute the total value of a collection of items in knapsack
  '''
  return sum([item[1] for item in knapsack])


def fitness_score(knapsack, max_capacity):
  '''
  Gives fitness score to each knapsack and repairs overweight knapsacks greedily
  '''
  return value(repair_overweight(knapsack, max_capacity))


def repair_overweight(knapsack, max_capacity):
  '''
  Removes lowest value-to-weight items from overweight knapsacks
  '''
  if weight(knapsack) <= max_capacity:
    return knapsack
  sorted_knapsack = sorted(knapsack, key = lambda x:  x[1]/x[0])
  while weight(sorted_knapsack) > max_capacity:
    sorted_knapsack.pop(0)
  return sorted_knapsack
</code>
