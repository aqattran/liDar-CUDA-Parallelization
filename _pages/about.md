---
layout: about
title: about
permalink: /
subtitle: <a href='https://www.cs.cmu.edu/afs/cs/academic/class/15418-f24/www/'>15418</a> Parallel Computer Architecture and Programming Final Project. Created by Alice Tran (aqt) and Alex Blasberg (ablasber)

profile:
  align: right
  image: 
  image_circular: false # crops the image to make it circular
  more_info:
  
news: false # includes a list of news items
selected_papers: false # includes a list of papers marked as "selected={true}"
social: false # includes social icons at the bottom of the page
---

View our project proposal [here](https://docs.google.com/document/d/1ECPBBzLD85j11i7b2N0DGE6IpOhdvrIZKCrS3YNjKvs/edit?usp=sharing)

### Summary
This project focuses on parallelizing the filter and cone coloring algorithm that are part of the CMR autonomous vehicle perception pipeline. Given the data points from the LiDAR sensor, we will take the point cloud and run our improved algorithms on GPUs to analyze the improvement from python. 

### Background
Carnegie Mellon Racing (CMR) is a student organization that builds and designs autonomous vehicles to compete in the FSAE Driverless competition. The car uses a variety of sensors in order to interpret the surrounding environment around it and the cones the car must drive between. The perceptions pipeline is the first step in the overall vehicle’s autonomous pipeline, converting the raw LiDAR data to a set of coordinates of known cones.

One of the sensors the car uses is the HESAI AT128 Solid State LiDAR sensor. The sensor works by shining a laser onto its environment and analyzing the reflected light. From this we receive a three dimensional point cloud of information with the sensor taking in approximately 1,536,000 points per second. Each point (with an x,y,z coordinate) represents an object a certain distance away from the sensor itself. 

Given that this information is from the point of view of the sensor, all the data points in the point cloud must be transformed to the car’s frame of reference. From there the points will then be filtered, which is the specific algorithm that we will be parallelizing. We aim to filter out points that are determined to be on the ground and those that are too far away.

Currently the process arranges the point clouds into segmented bins. Within each bin the algorithm determines whether the points are part of the ground or not. If it is part of the ground, the algorithm then removes it. We then perform a plane fit on each of the segmented bins to flatten the data.
The filtering process is ideal for parallelism as we are dealing with millions of data points from the original point cloud. This algorithm also lends itself well to parallelization, as you can split the computation up so that each process takes in a few “bins” and computes the ground plane in that manner. Currently the algorithm is sequential and written in Python, which slows it down dramatically. By executing the algorithm in parallel, we stand to see a large speedup, cutting down on the perceptions pipeline latency greatly.

### The Challenge
The largest issue that we have to deal with is the fact that we are dealing with a large number of data points and additionally that the data points will not arrive in spatial order. Therefore we are receiving over a million data points that will vary in terms of their spatial location and therefore complicate how we decide to partition them. Without an easy way to partition them spatially, the problem becomes far from trivial to parallelize.

While we plan to use CUDA to filter the points in parallel,  a problem we could face is the lack of locality present. For example, if we choose to simply batch the points spatially we may suffer from a high communication cost as we wait for remaining points to be received. However, if we were to randomly batch and analyze points regardless of spatial location we would have to then spatially arrange the points after the fact.

Additionally, we will deal with the challenge that the algorithm is part of a much larger pipeline. After the data points have been filtered they also must be clustered into objects (specifically cones on the race track) and then identified as right or left cones. This also allows us possible extension of our project by allowing us to parallelize other sections of the LiDAR pipeline. 

### Resource
We are currently working with the already existing [perceptions library](https://github.com/carnegiemellonracing/PerceptionsLibrary22a) for CMR. All of our code will be run on a Cincoze GM-1000 on the vehicle, which has an incredibly powerful GPU that is currently underutilized by the driverless vehicle’s pipeline. Therefore, we believe our best path for speedup would be to implement this algorithm in CUDA. Specifically we hope to optimize the [filtering algorithm](https://github.com/carnegiemellonracing/PerceptionsLibrary22a/blob/main/perc22a/predictors/utils/lidar/filter.py)  for the points in point cloud from the data given to us by the LiDAR sensor. If we accomplish this, we can also parallelize other tools within the LiDAR pipeline as mentioned [here](https://cmr.red/PerceptionsLibrary22a/documentation/html/build/html/perc22a/predictors/utils/lidar/lidar.html). 

### Goals and Deliverables
We have split our project into three tiers of goals. We see this as our main objective, a strong (but difficult) goal, and a best case scenario goal. Our main goal is the ground filtering algorithm, but if we have the ability to expand our scope we will. We have broken our plan down into phases here.

#### Phase 1: Parallel Ground Filtering
Implement GraceAndConrad algorithm in CUDA, binning with Thrust and integrating with existing perceptions pipeline.
It is difficult to calculate expected speedup from this implementation, as much of the existing ground filtering algorithm utilizes incredibly well optimized libraries for computation. However, we would like to cut computation time by 50%.

#### Phase 2: Parallel Ground Filtering and cone coloring
Implement parallel cone coloring in C, cutting down on the majority of the pipeline’s latency with a CUDA based cone coloring algorithm.

#### Phase 3: Complete Parallel Perceptions Pipeline
Implement full front to back perceptions pipeline in C, with calls to our CUDA helper functions.

CMR has a perceptions visualizer, which we can use to demo our work  and the effectiveness of our ground filtering algorithm at the poster session. This visualizer will show the point cloud and let us see the impact of our filtering algorithm. Along with this, we can run our pipeline on existing LiDAR data to characterize speedup and overall performance gain from our parallel implementation. This will allow us to quantify speedup and our implementation’s impact on the overall vehicle latency.

### Platform Choice

We have chosen to implement this algorithm in CUDA because of its clear benefits when dealing with massively parallel problems. Since the LiDAR returns so many points, even after binning and downsampling we are dealing with quite a few sections of computation that can be run in parallel. GPU programming makes the most sense for this, as we will not be constrained by hardware, as CMR uses the Cincoze GM-1000. Although the system has a high core count CPU, it does not have nearly enough processors to properly parallelize this problem. Furthermore, the perceptions pipeline runs in parallel to the path planning and controls pipelines, which use a large amount of CPU compute. By offloading this work to the GPU, the vehicle will be much faster overall and see a decrease in its overall end-to-end latency.

### Schedule

| Dates     | Work                                                                                                                                   |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 11/6 - 11/13|  Project Proposal and Ideation, Familiarizing with the current CMR codebase and perception pipeline |
| 11/13 - 11/21 |Measure old times on python and compare to new times with improved version, Performance Debugging, improve parallel implementation if necessary  |
| 11/28-12/5| Parallel Cone Coloring algorithm with improved pipeline, Measure new pipeline times compared to old times in Python|
| 12/5 - 12/13 |Gather testing data, make final performance improvements, create research poster|
| 12/13 | Presentation |                                                                                                                     







