# Project 1: Flocking
**University of Pennsylvania, CIS 565: GPU Programming and Architecture,
Project 1 - Flocking**

Caroline Lachanski: [LinkedIn](https://www.linkedin.com/in/caroline-lachanski/), [personal website](http://carolinelachanski.com/)

Tested on: (TODO) Windows 10, i5-2222 @ 2.22GHz 22GB, GTX 222 222MB

## Project Description

Insert gifs and description here

## Performance Analysis

Performance was analyzed by recording the FPS under various conditions.

First, we can look at how FPS changes by increasing the number of boids in the simulation. This is with visualization of the boids turned off, meaning no computing power is being used to actually draw the boids.

We can look at the same conditions with boid visualization turned on. This seems to decrease performance, likely because some amount of the GPU's computing power is needed to draw the boids.

We can also look at how block size, or number of threads per block (and correspondingly, number of blocks), affects performance.

I found the change in performance difficult to discern, so I performed the same tests with an increased number of boids (now 100,000), wondering if differences would begin to appear. I also included block sizes that were not powers of two to see the effect.

Lastly, here is a comparison between grids of different cell width, one where cell width is 2 * max distance, meaning we generally have to check 8 cells, and one where cell width is simply the max distance, meaning we generally have to check 27 cells.


## Question Responses

**For each implementation, how does changing the number of boids affect performance? Why do you think this is?**

**For each implementation, how does changing the block count and block size affect performance? Why do you think this is?**

**For the coherent uniform grid: did you experience any performance improvements with the more coherent uniform grid? Was this the outcome you expected? Why or why not?**

**Did changing cell width and checking 27 vs 8 neighboring cells affect performance? Why or why not? Be careful: it is insufficient (and possibly incorrect) to say that 27-cell is slower simply because there are more cells to check!**
