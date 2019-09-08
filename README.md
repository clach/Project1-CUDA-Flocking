# Project 1: Flocking
**University of Pennsylvania, CIS 565: GPU Programming and Architecture,
Project 1 - Flocking**

Caroline Lachanski: [LinkedIn](https://www.linkedin.com/in/caroline-lachanski/), [personal website](http://carolinelachanski.com/)

Tested on: Windows 10, i5-6500 @ 3.20GHz 16GB, GTX 1660 (personal computer)

![10000 Boids](/images/20000boidsBIG.gif) 

## Project Description

The goal of this project was to implement a flocking simulation based on the Reynolds Boids algorithm using CUDA. The flocking simulation is heavily based on [Conrad Parker's notes](http://www.vergenet.net/~conrad/boids/pseudocode.html), with some small changes. 

10,000 Boids | 100,000 Boids
------------ | -------------
![10000 Boids](/images/10000boids.gif) | ![100000 Boids](/images/100000boids.gif)

This project includes three implementations: a naive simulation, then an optimization using a uniform grid, and lastly another optimization using the uniform grid with semi-coherent memory access.

### Boids Simulation
In a basic boids flocking simulation, the boid particles follow three rules:
1. Cohesion - boids move towards the perceived center of mass of their neighbors
2. Separation - boids avoid getting to close to their neighbors
3. Alignment - boids generally try to move with the same direction and speed as their neighbors

In this particular project, each rule has a neighborhood distance: for each rule, each boid is only affected by its neighbors who are within a particular distance. During each simulation timestep, each boid calculates the velocity change contribution from each of the three rules and adds it to its current velocity, which is then used to update the boid's position.

### Naive Implementation
In the naive implementation, we check each boid against every other boid in the simulation to see if it is within each rule's neighborhood distance.

### (Scattered) Uniform Grid Implementation
We can improve on the naive implementation with the addition of a uniform grid. Checking each boid against every other boid to see if it's within a certain neighborhood distance is inefficient, especially when that neighborhood distance is much smaller than the simulation space. Instead, we can bin the boids within a uniform grid, where each cell width is the double the maximum of the three neighborhood distances. 

![](/images/Boids%20Ugrid%20base.png)

This way, in a 3D implementation, a maximum of 8 cells has to be checked to find boids within the neighborhood distance.

![](/images/Boids%20Ugrid%20neighbor%20search%20shown.png)

The uniform grid is constructed by determining which grid cell a boid belongs to using the boid's 3D position, then sorting the grid cell indices of the boids. From there, we can determine the start and end indices of each cell, making it easy to get the indices of the boids that belong to a given cell we wish to check.

![](/images/Boids%20Ugrids%20buffers%20naive.png)

### Coherent Uniform Grid Implementation

One can optimize on the previous implementation by removing a level of indirection. In the previous implementation, one relies on a buffer of boid indices (originally an identity buffer) that was sorted using the boid's grid cell index as the key. This buffer of boid indices can then be used to index into the position and velocity buffers that are the input to the simulation. In the coherent uniform grid implementation, we simply rearrange the position and velocity buffers according to this buffer. These buffers now follow the order of the grid cell indices, which is useful because we will access them based on the sorted order of grid indices. This is what is meant by semi-coherent memory access.

![](images/Boids%20Ugrids%20buffers%20data%20coherent.png)

## Performance Analysis

Performance was analyzed by recording the FPS under various conditions.

First, we can look at how FPS changes by increasing the number of boids in the simulation. This is with visualization of the boids turned off, meaning no computing power is being used to actually draw the boids.

![FPS vs Num Boids (Vis OFF)](/images/FPSvsNumBoidsVisOff.png)

We can look at the same conditions with boid visualization turned on (the y-axis scale is kept the same as in the previous graph, to emphasize the difference in FPS). This change seems to decrease performance, likely because some amount of the GPU's computing power is needed to draw the boids. Nicely enough, the overall trend seen as the number of boids increase looks the same as when visualization is off.

![FPS vs Num Boids (Vis ON)](/images/FPSvsNumBoidsVisOnAdjustedScale.png)

We can also look at how block size, or number of threads per block (and correspondingly, number of blocks), affects performance.

![FPS vs Block Size (25000 Boids)](/images/FPSvsBlockSize25000Boids.png)

I found any change in performance difficult to discern, so I performed the same tests with an increased number of boids (now 100,000), wondering if differences would begin to appear. I also included block sizes that were not multiples of 32 (the warp size) to see the effect. Again, the differences seem minimal.

![FPS vs Block Size (100000 Boids)](/images/FPSvsBlockSize100000Boids.png)

Lastly, here is a comparison between grids of different cell width, one where cell width is 2 * max distance, meaning we generally have to check 8 cells, and one where cell width is simply the max distance, meaning we generally have to check 27 cells. The first graph is for 50,000 boids:

![FPS vs Grid Cell Size (50000 Boids)](/images/FPSvsGridCellSize50000Boids.png)

The next is for 100,000 boids:

![FPS vs Grid Cell Size (100000 Boids)](/images/FPSvsGridCellSize100000Boids.png)

Interesting enough, the trends do not match when you increase the number of boids.

## Question Responses

**For each implementation, how does changing the number of boids affect performance? Why do you think this is?**

Looking at each implementation separately, as you increase the number of boids, performance decreases. This is likely due to there being an increased number of boids to simulate, which also means that for each simulation step, each boid must be checked against a larger number of boids. This is perhaps why the naive implementation has a more drastic downward trend; because it involves checking each boid against every other boid, increasing the number of total boids in the simulation will have a large effect. Interestingly, for smaller numbers of boids (1000 and 5000), the "more efficient" implementations don't necessarily have better performance (in fact, for 1000 boids, the naive implemenation performs the best). This is probably because for such a small number of boids, the overhead accrued for running the preprocessing steps and maintaining the uniform grid is probably greater than the advantage it provides.

**For each implementation, how does changing the block count and block size affect performance? Why do you think this is?**

For each implementation, changing the block count and block size doesn't seem to affect performance in any meaningful way. Since the number of blocks is calculated based on the block size, it seems that a similar number of threads is created for each run, thus not changing performance. I was interested in seeing if block sizes that were not multiples of 32 would change anything, and surprisingly, the changes seemed generally minimal. I would have expected to see decreases in performance since not using multiples of 32 (the warp size) would mean there were unused threads, but perhaps the differences were simply ndiscernible. Also, human error in recording FPS counts could also be at fault.

**For the coherent uniform grid: did you experience any performance improvements with the more coherent uniform grid? Was this the outcome you expected? Why or why not?**

Yes, using the coherent uniform grid over the scattered uniform grid saw large performance improvements, especially at larger numbers of boids. I personally didn't expect such as a drastic difference, thinking one level of indirection wasn't such large issue, especially when the coherent uniform grid implementation also required two cudaMemcpy()'s. The difference is likely because in the scattered uniform grid implementation, we had to index into an additional array within a triple for-loop (iterating over the number of grid cells to check) for each boid, versus simply re-indexing two buffers for each boid along with two cudaMemcpy()'s. And because we rearranged the position and velocity buffers to match the order of grid cells indices, we are less likely to cause a cache miss as we iterate over grid cells in the same order as they are in memory.

**Did changing cell width and checking 27 vs 8 neighboring cells affect performance? Why or why not? Be careful: it is insufficient (and possibly incorrect) to say that 27-cell is slower simply because there are more cells to check!**

In my implementation, the number of cells check is not hard-coded and is instead based on how many cells fit within the max neighborhood distance, though this should generally correspond to checking 27 versus checking 8 neighboring cells. Changing cell width between 1 * max neighborhood distance and 2 * max neighborhood distance had varying effects on performance given the number of boids and the implementation as seen in the graphs above. While using 1 * max neighborhood distance, one generally has to check more cells, there will also generally be a fewer number of boids within each cell, meaning one has to iterate over the boids for-loop within the cells for-loops less times. No matter cell size, the number of boids within the neighborhood distance would not change. However, using a smaller cell size means we achieve a tighter search resolution, and are less likely to check cells that do not contain boids within the neighborhood distance. But, increasing the total number of boids would mean one is more likely to find boids within a nearby cell, and perhaps cell size becomes less important. It's hard to say what exactly caused the changes we see in the graphs above.
