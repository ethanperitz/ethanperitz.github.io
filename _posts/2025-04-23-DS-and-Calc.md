---
layout: post
title: Calculus vs. Data Science
date: 2025-04-23
---

# Calculus vs. Data Science

There is a debate that has been brewing for several years in mathematics education.  Should Calculus remain the gold standard for high-achieving high school math students? If not, what does a rich math experience in high school look like without Calculus as the end goal?

[This article](https://www.nytimes.com/2023/07/13/us/california-math-data-science-algebra.html?smid=url-share) from 2023 in the New York Times describes the complicated landscape in California public schools.  On the one hand, providing a non-Calculus pathway provided greater access for students to take higher level mathematics, but on the other, many perceived the data science courses as less rigorous than a more traditional pathway.  

In my experience teaching all levels of high school math, including Data Science, I worry about the segmentation of mathematics into two seemingly distinct worlds:  the pure and the applied.  Mathematics tends to be perceived by students as a heavily segment subject anyway; lots of students have a hard time seeing the inherent connections between, say, the quadratic formula and the Pythagorean theorem.  But, mathematics instructors should be finding opportunities to build these connections whenever possible. 

In my experience as a Data Scientist, especially when I was getting my MS, I found that my knowledge of pure mathematics, especially the content I learned at the high school level, was invaluable.  Knowing even rudimentary information about derivatives and asymptotic function behavior allowed me to access the graduate-level content much more readily.  

So, I do not believe in the bifurcation of pure and applied mathematics.  One of the few constants of my career has been how learning more mathematics helps me to understand the mathematics I thought I already knew and understood.  

As I started to teach Data Science, I found it very easy to fall into a deadly trap.  When the work you do with students is mostly *application-based*, students tend to gravitate towards *how* as opposed to *why*.  And as a teacher, I found that I could get away with teaching the *how* without the *why*, and my students could still produce really nice work.  

For example, I teach my Data Science course in R.  When teaching students about k-Nearest Neighbors algorithms, I could send along some template R code that used tidymodels to establish a coding framework.  From there, students only needed to plug-and-chug their way through problems.

## Template Code for k-Nearest Neighbors Classification Tasks in R
```r
#factorize target/outcome variable
data$ <- factor(data$ )

#split into train and test sets
data_split <- initial_split(data = data , prop = ())
training <- training(data_split)
testing <- testing(data_split)

#Create recipe with intended explanatory variable(s)
recipe <- recipe( ~ , data = training)

#Examine the recipe to make sure the outcome/predictor variables are correct.
prepped_recipe <- prep(recipe)
data.frame(prepped_recipe$var_info)

# Create the specifications of a nearest neighbors model - do not edit
knn <- nearest_neighbor(neighbors = ) %>%
set_engine("kknn") %>%
set_mode("classification")

#make the workflow - add the recipe and the model type
knn_workflow <- workflow() %>%
add_recipe() %>%
add_model() 

# Fit the model to the training data - only edit if your training dataset isn't called "training"
knn_fit <- knn_workflow %>%
fit(data = training)

# Make predictions on the testing data - only edit if your testing dataset isn't called "testing"
predictions <- knn_fit %>%
predict(testing, ) %>%
pull(.pred_class)
```

With enough practice, students found it easy to create and evaluate simple machine learning models.  To use some familiar terms from above, this template increases access and equity for students to do some really cool things.  

But,

...this template *often* fails, and it's important for students to know when to use it.  Complicating matters further, this template often fails in ways that aren't immediately obvious.  I found that students would use poorly-chosen explanatory variables, attempt to use this template for non-categorical classification, or attempt to use this template for classification tasks with **many** targets, all of which indicate a poor understanding of Data Science.  So, teaching students *how* to use this template fails the rigor test.  

Here is where the pure math approach really helps.  Most students are going to find this stuff really cool, and it can motivate them to better understand some of the inherent mathematics that are lying just under the surface.  

For starters, students need to know how **distance functions** work to choose their explanatory variables well.  How do we decide what constitutes a "neighbor" in this algorithm? What about a *nearest* neighbor? 

Then, there is the problem of **scale**.  What if I want to use height (in cm) and age (in years) to classify someone as a high or low income earner? There is more variability in height than in age just because of the units of measurement, which complicates our understanding of the "nearness" of the neighbors.  Two people who are 10 cm apart in height should not be treated as the same distance apart as two people who are 10 years apart in age.  This motivates the creation of some function to standardize the units, which relies on a solid understanding of some pure math topics.

One solution that I have really enjoyed to this rigor/equity problem is teaching Data Science topics without coding first.  A human being can absolutely carry out an entire k-nearest neighbors algorithm with a small enough dataset, and guiding students through that process can be done in a rigorous, inquiry-based way.  Students can also be pushed to work on their argumentation skills with good discussion and questioning habits.  

In a task where students must, by hand, classify a day as SUNNY or NOT SUNNY by comparing it to labeled data, I tend to have every student decide within 2 minutes whether the data indicates a sunny day or something else, just with their eyeball check.  I may even tell them if they are right or wrong.  The fun and learning start when I then say, "Write me a sound argument supporting your belief."  I then have the time of my life rejecting their arguments until I am satisfied.  This is one of those classic "low floor, high ceiling" tasks that ends in some deep takeaways both in applied and pure mathematics. 

The goals of the California initiative to bring Data Science into high school curriculum will inevitably fail if we, as teachers, fail to let rigor, curiosity, and motivated mathematics lead the way.  

Last, the wedge driven between the "Calculus" track and the "Data Science" track fails to acknowledge how much these two disciplines lift each other up.  A student motivated to solve problems in Data Science will write more successful algorithms if they know the Calculus behind, say, Gradient Descent. A student motivated to pursue Calculus will understand the mathematics more deeply if they know how to code an optimization algorithm that they carry out with a real dataset (ideally one that they have cleaned, studied, and contains relevant, impactful data).  Calculus and Data Science *should* be best of friends, but they are currently treated like two roads that diverge in a yellow wood.  

As teachers, we want students to feel successful with mathematics to the point of deep understanding and continual pursuit.  Data Science, when done well, should not be seen as a less-rigorous alternative to high-level, pure mathematics, but rather as a natural companion and motivator of those mathematics.  

In my non-Data Science classes, I tend to get the dreaded "When am I ever going to use this" question (read: complaint).  I never get that question in my Data Science classes, and my students are doing Algebra and Geometry daily.  That speaks volumes about how Data Science *should* be viewed - not as an alternative, but as a spark that lights the mathematical fire.    
