---
title: SDM And Database - Part 1 - Planning
date: 2016-12-11 14:16:33
tags:
  - sdm
  - database
---
# Introduction

With my [Sparse Distributed Memory (SDM)](https://github.com/JoeyAndres/sdm) basic implementation finally done, in which the [Up/Down Counters won't overflow](https://joeyandres.github.io/2016/12/10/sdm-addressing-the-up-down-counter-overflow/), I’m ready to embark something bigger. At the moment, my implementation stores everything in random-access-memory (RAM), which is very limiting. I have utilize SDM with my [reinforcement learning (rl)](https://github.com/JoeyAndres/rl), solving entailments in a set of 11,000 metrics, which occupies all my 16gb of ram. This is not ideal since, even with SDM, there are many collisions, granted they are better than random hashing collision provided by *Tile Coding.* Also, I can only do machine learning on a smaller and sparse set of data, due to the space. To amend all of this, it is clear what the next step is. Utilize database systems.

# The Plan

Since all the "bottleneck" stems from the space requirements of the *Up/Down Counters*, it makes sense to somehow store all that stuff in database. For a quick review of what *Up/Down Counters *look like, see the figure below.

{% asset_img image_0.png "SDM Structure Image" %}

What I want is to store each row in database.

{% asset_img image_1.png "Saving row to database" %}

Considering that *Up/Down* *Counters* row represents a feature vector which is probably not that big, this might be sufficient. Problem occurs if the feature vector is actually huge, then we probably need a smarter way of segmenting each row into cells.

{% asset_img image_2.png "Saving chunks to database" %}

Now we are more robust, allowing us to store bigger data. One thing to note is that storing each row will be faster, due to less "context-switching" when switching to another db entry. Cell solution can do the same by setting the *column width* to something equal to the size of data. Our final schema is represented by the image below.

{% asset_img image_3.png "Schema" %}

* Each *UpDownCounter* instance have a db entry.

    * Contains the size of each row, *DataSize*.

* Each *UpDownCounter* instance have many associated *Cell*.

    * Contains the *ChunkSize* of the cell. Although can be saved in *UpDownCounter*, imagine, having some cell bigger than the other. This should allow for that possibility, although I don’t intend to use it (I don’t mind the redundancy that much).

    * And the data itself, *BinData.*

    * The *row, col* stores where this cell is located.

# Choosing A Database

Since I don’t intend to uphold the CAP theorem, that is, the possibility of SDM incrementing the same counter at the same time should be possible, and I want horizontal scalability, I must choose one of the NoSQL database. I’ve looked at MongoDB and it’s *C++ *client (Note, my SDM implementation is in C++) and have found it difficult to build and install. I’ve also looked at Cassandra and it seems like a better fit for my problem. It was easy, way faster, and way scalable.

# Conclusion

That’s it! We are done. See you in part 2 when I have a basic working implementation of this.
