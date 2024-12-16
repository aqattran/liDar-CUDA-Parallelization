---
layout: about
title: midpoint 
permalink: /final/
nav: true
nav_order: 2

profile:
  align: right
  image: 
  image_circular: false # crops the image to make it circular
  more_info:
  
news: false # includes a list of news items
selected_papers: false # includes a list of papers marked as "selected={true}"
social: false # includes social icons at the bottom of the page
---

View our report [here](https://docs.google.com/document/d/1MLtYLhmaWYhxVXP5Doyvv5CVxGbg0IdS6SBI8sdtQUQ/edit?usp=sharing)

### Milestone Progress
We have maintained our progress and are currently still on schedule to complete all phases of our project. The original algorithm for ground filtering in python approximately took 40 ms. We explored three different iterations of possible algorithms in CUDA in order to see if they would improve the desired time. Similar to the original GraceAndConrad algorithm, we chose to divide the point cloud of information in individual buckets and cells. 

Within all three of our iterations we began with the original structure. In our host, we begin with loading all the points of information from the frame into the GPU where it can be accessible to our kernels. We then launch a kernel in order to allocate each of our points into a cell in a 2D grid defined in polar coordinates (radius, angle). Here it logically makes sense to assign each kernel thread to a point in our point cloud for allocations. We organize our buckets into a horizontal bands which we call segments. The number of segments is determined beforehand by calculating the minimum and maximum angle between all our points and dividing by a given alpha (the size of each segment). 

Once we have organized our cloud of points we then experimented with multiple ways to parallelize processing our points in order to create an accurate outline of where the ground. In all our iterations, we chose to launch a kernel with segment number of blocks and each block containing the bucket number of threads.  In our first iteration, we chose to find the lowest point in each bin. With the lowest point from each bin, we then did a plane fit across the entire frame to approximate the position of the ground. While this process took approximately 7 ms compared to the original 40 ms in python this fit was not accurate compared to the base level results from python.

In our second iteration, we chose to take the three lowest points from each bucket in order to create an individual plane fit for each cell. While this provided us with the necessary accuracy it was not our fastest implementation. In our final implementation we chose to mirror the original python algorithm. Using all of the buckets in a segment, we chose to fit a line through their lowest points. From the lines in each of the segments we could then determine the ground. This gave us the fastest speed of 4ms with the best accuracy compared to the original python implementation. 

### Updated Schedule

                                                                                                                        
#### 12/1 - 12/4
- Begin Writing Summary and Background for Final Report  (Alice)
- Begin Parallelizing for Clustering (Alex) 
- Discussion on Final presentation poster format (Alice + Alex)

#### 12/5 - 12/7 
- Finish Clustering Algorithm (Alex)
- Implement visualization of objects and course (Alex)
- Approach Paper Writing Section (Alice)
- Gather/Create Graphics for final paper and presentation (Alice)
  
#### 12/7 - 12/13
- Downstream Filtering (Alex + Alice)
- Attempt Parallel Coloring
- Wrap up Paper + poster
  
#### 12/13 
- Presentation 

### Goals and Deliverables

We finished our first goal of parallelizing the ground filtering algorithm. Our next deliverables are to parallelize clustering which is grouping points into an object, coloring which is determining whether a cone is on the left or right side of the track, and downsampling. 

Our current list of goals broke down are:
- Parallelize clustering
- Parallelize downsampling
- Implement visualization support for C code
- Parallelize Coloring (Nice to have)

### Poster Session
In our poster session we plan to have a video demonstration that will show the renderings of both the ground and also the clustering results to represent the cones along the race track. We will additionally have a graph demonstrating the corresponding decrease in time between python and then switching to our CUDA implementation. 

### Concerns
 One of the main processes is coloring which identifies whether a cone is on the left or right side of the vehicle. However, when we tried to implement this we saw that this was an inherently sequential process. The current algorithm first determined a cone to be on the left or right side of the track. From there it finds the next closest cone and then determines the cone to be on the same side as its neighbor. Therefore since the approach is based on the proximity of its neighbor the path, the process relies on sequential reasoning. Therefore our current attempts at parallelizing this have either been inaccurate or the same speed as the original sequential version. 

Currently, our two proposed strategies for parallelizing are using the laser intensity or using a disjoint union find. Since the cones on the left side and right side are colored blue and yellow respectively we can distinguish these cones due to the intensity of the light reflected. Another way to distinguish the cones is that we know that cones are approximately 1 meter apart and we also know the width of the race track. We will use CUDA and assign each thread to a cone. We will then find the cone’s neighbors. Using this information we hope to then parallelize the process through a union find algorithm that treats each grouping of cones as disjoint sets. 

