---
title: Tile Code - Generalization
date: 2016-12-24 01:40:24
tags:
  - reinforcement learning
  - richard sutton
  - coarse coding
  - coarse code
  - tile coding
  - tile code
  - generalization
---
# Introduction

{% asset_img image_0.png "Course Coding Types" %}

Coarse Coding, such as Tile Coding, is a nice way of storing a model of the environment linear in nature or have an ordinal characteristics (for instance, alphabets are ordinal since it can be ordered: A < B < C, …). The image above illustrates how Coarse Coding may be utilized. *a) Narrow Generalization* is for problem space where precision over backup speed (samples have to be taken for neighbour values to propagate). *b) Broad Generalization* is for problem space where generalization can be taken advantage of, making value propagation from neighbours faster.

Problem often occur if our Course Coding model is too fine we want to "Broaden" our model. Before we begin, I’ll discuss a real world problem I encountered recently.

In one of a project with a client, we are given thousands of metrics and ask to solve the following problem:

*Given a pattern in a metric, what patterns in other metric entails it. For instance, given a pattern in metric M1 what pattern in other metrics are a culprit or causes it.*

In our project, one of the pattern of interest is a rising *web response time *metric, and we are asked what entails/causes it? With a little [feature engineering](https://en.wikipedia.org/wiki/Feature_engineering), we are able to linearize metrics. By running this through a program that utilizes my [rl library](https://github.com/JoeyAndres/rl), *db server response time* was found as one of the culprit. These patterns are given below.

{% asset_img image_1.png "Web Response Time vs DB Response Time" %}

What the program did is basically create a model of the system so that if the *db server response time *pattern do occur again, we know that it will entail rising response time. An problem arises, it is obvious that *db response time* don’t have to follow the exact pattern above to entail *web response time *to rise. That is *db response time* just have to be elevated around 40-50ms response time for a problem to occur. That is why we want to generalize our model stored in Coarse Coding (specifically Tile Coding).

# Solution (Tile Coding)

Although this solution is only for Tile Coding, general idea applies to other types of Coarse Coding. So in our metric above, we want to generalize it to:

{% asset_img image_2.png "Web Response Time vs DB Response Time (Broad)" %}

Note on how the bumps are "smoothed out". This should now give us a better model for this problem. So *how do we generalize Tile Codes?*

To begin, let us first review a simplified version of Tile Code where it stores a 2D state-action space (1D state and 1D action). Suppose it is stored in the following tile (we will ignore multiple tiling to simplify this further):

{% asset_img image_3.gif "Grid" %}

Suppose actions are represented by the x axis while the states are represented by the y axis. Generalizing would mean storing the values stored in this grid to the following "Broader Grid":

{% asset_img image_4.png "Broad Grid" %}

Each square in *Broad Course Code* covers many vertex in the original Course Code. The idea then is each vertex in *Broad Coarse Code* have a value is the weighted average of all the covered vertex in the original Course Code. This is better illustrated visually here:

{% asset_img image_5.png "Overlay Narrow/Original and Broad Grid" %}

* Let green grid represent the original Tile Code grid.

* Let the red grid represent our wanted Broad Tile Code grid.

* Blue dot be one of the vertex (storage of state-action value) in our new Broad Tile Code grid.

So for the value in the vertex represented by the blue dot, it will be the weighted average of all the green vertex that is under the 4 red squares it sits on. (If vertex is at the side, then the vertex will be sitting on two squares whilst if it is in corner then it will be sitting on just one square). The weights would be the cartesian distance between a green vertex and the blue vertex. The weighted average can easily be done by the following formula:

{% asset_img image_6.png "Weighted Average Formula" %}

# Conclusion

Here we assume that the Tile Coding did not do any hashing. This is an equivalent of using the *TileCodeCorrect *class in my [rl library](https://github.com/JoeyAndres/rl). Utilizing something with hashing shouldn’t be that big of a deal since the same representation of grids above still holds except that vertex are in "superposition", in which a vertex is utilized by more than one state-action pair. More sophisticated methods are also needed for “non conventional tiles”:

{% asset_img image_7.png "Types of Tiling" %}

This method is really helpful in many situations in which discarding precision for myopic view of things is more useful.
