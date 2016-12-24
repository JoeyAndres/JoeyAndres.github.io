---
title: Sparse Distributed Memory - Addressing the Up/Down Counter Overflow
date: 2016-12-10 14:16:33
tags:
  - ai
  - sdm
  - sparse distributed memory
---
# Introduction

Sparse Distributed Memory (SDM) is a fascinating way of managing exponential growth in memory. As opposed to the traditional random-access memory (RAM) where there is a one to one correspondence between an address and a *hard location, *SDM can map multiple address to a multiple of *hard location*. I won't delve into why this is useful, I suggest reading Mr. Kanerva’s paper, *A Cerebellar-Model Associative Memory As A Generalized Random-Access Memory*, which I believe is available in IEEE. The purpose of this article is to discuss the shortcoming of writing data in SDM and an amendment from Reinforcement Learning community.

# SDM Write/Read operations

Before we delve on addressing the shortcoming, let us quickly review on how the SDM *write/read* operation works. The figure below is taken from Mr. Kanerva’s paper mentioned above. Again, this is just a quick summary of the *write/read* operations, thus if lost, consult the paper. 

In the paper, it describes the writing of *1011001010* and then *0101010110 *to the SDM. Prior to any operations in SDM, the* Up/Down Counters* or the data storage itself are all zeros. For the first insertion, *1011001010* activated 3 address register in which its Hamming Distance is less than or equal 3. This then updates each cell in activated/selected rows in *Up/Down Counters*. For instance, all columns aligned to the *1s* in *1011001010* are all incremented by one, and all columns aligned to *0s* are decremented.

{% asset_img image_0.png "SDM Writing operation" %}

Similar operations occurred for second insertion, *0101010110* but different rows due to different hamming distance less than or equal three are activated.

Reading is by summing all the activated columns. For instance, reading the data addressed by *0101010110* is through summing all the activated columns, resulting in (*-3, -5, -3, 5, 5, 3, -3, 3, -3, 3). *The sum is finally converted to bits, assigning 0 if the sum of a column is negative, whilst 1 if otherwise. So in our case this is: *0001110101*.

{% asset_img image_1.png "SDM Reading operation" %}

# Problem with SDM Writing

Now suppose each cell in *Up/Down Counter* is represented by a 64bit signed integer (to allow for negatives) in a 64bit machine, thus occupying exactly one memory cell. Then there is a scenario in which after 264. Although this may seem like a super worst-case, it’s an overflow we’d rather not have. I have been reading unmentioned implementation and one of it have a *try/catch* or an *if/else* guard, considering these write operations happens a lot, this is a significant overhead.

# Solution 1: Geometric Series

So our problem is integer overflow. How do we solve this? We want something that is something with a horizontal asymptote, something that slows growing as it approaches a value, all to avoid the overflow.

Geometric series have the following form:

{% asset_img image_2.png "Geometric series" %}

Where *a *is the initial value, I usually set this to 1 and *r *is the common ratio. As you can see above, the sum of geometric series is:

{% asset_img image_3.png "Geometric series sum." %}

If *r* is set to |r| < 1, then the series converges or "hits" a horizontal asymptote. So how is this useful as a replacement for integers in *Up/Down Counters*?

Imagine each cell in *Up/Down Counters* is a double (64bit floating point), *a=1, and |r| < 1 *(again so we converge), how do we utilized Geometric Series’ converging nature? 

* Let **R** be the current value of a cell.

* Let **R’ **be the next value of the same cell.

So we acquired **R** in our nth update. Since it is a geometric series where *a=1, and |r| < 1*. 

R = 1 + r + … + rn

This is not very useful since it assumes we keep track of integer *n* which defeats the purpose. How can we make a recurrence relationship, that is, how do we get **R’ **from **R**? Continuing from formula above:

R = 1 + r + … + rn

R’ = 1 + r + … + rn + rn+1

R’ = 1 + (r + … + rn) x r

**R’ = 1 + R x r**

To generalize *a=1*,

**R’ = a + R x r**

That’s it. That is how we increment our cell in *Up/Down Counter.* Are we done? How do we decrement?

# Solution 2: From Reinforcement Learning

To solve the decrement problem above, we’ll introduce an equation from Reinforcement Learning, specifically, [SARSA](https://en.wikipedia.org/wiki/State-Action-Reward-State-Action).

{% asset_img image_4.png "SARSA Update Equation" %}

To quickly summarize (you can skip if you want) SARSA, (st, at) is the state-action pair. Q(st, at) is what we are trying to update base on the current observed reward of the next state Q(st+1, at+1). Step-size α controls how fast Q(st, at) learns or converge while 𝛄 dictates how influential future states are. Basically, by shifting rt+1 + 𝛄Q(st+1, at+1) to something greater than Q(st, at), Q(st, at) will be incremented and vice-versa. There lies the idea behind our final solution. If you decompose the equation above, you’ll also unravel a geometric series.

If you don’t care about the SARSA equation above, it’s fine. Basically it allows us to increment/decrement. To make build our final solution, simply replace Q(st, at) as **R**, or the floating point we are trying to update in our *Up/Down counter *cell.

**R <- R + r[1 - R], ** _for increment_

**R <- R + r[-1 - R], ** _for decrement_

Done! All I can say is shift your common-ratio/step-size to a really small value so you won’t converge quickly. Maybe change **1** to something bigger for the same reason. Other than that, NO MORE OVERFLOW!

# Conclusion

Truth be told that the update formula from Reinforcement Learning is not unique. It is everywhere in machine learning literature such as *Perceptron Learning Rule* and the ever more popular *Linear Regression.* I just really love Reinforcement Learning that’s why I want to nudge it in.
