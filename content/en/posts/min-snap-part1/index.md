+++
title = 'Minimum Snap Trajectory Generation-Part 1'
date = 2024-07-19T07:28:02-04:00
draft = false
math = true
+++
# Minimum Snap Trajectory Generation-Part 1


## 👋Introduction
In robotics, motion planning addresses the challenge of moving a robot from a starting point (point A) to a desired goal point (point B). The specific approach to motion planning depends heavily on the type of robot and its environment. For instance, the solution for an industrial robot arm's end effector to reach a specific point will be vastly different from that needed for a quadcopter to navigate a complex environment.

![Grasp Planning for a robot arm](https://moveit.picknik.ai/humble/_images/grasp_pipeline_demo.gif)

_Grasp Planning for a robot arm. 
Source: https://moveit.picknik.ai_



![FAST-Planner](https://github.com/HKUST-Aerial-Robotics/Fast-Planner/blob/master/files/icra20_3.gif?raw=true)

_Motion Planning for a quadcopter. 
Source: https://github.com/HKUST-Aerial-Robotics/Fast-Planner_

Consequently, a motion planning strategy ideal for a robotic arm performing a pick-and-place operation wouldn't translate well to a quadrotor navigating a forest. This highlights the importance of considering factors like the robot's dynamics and environmental constraints when choosing the most effective motion planning approach.

In this blog, we'll delve into generating efficient and dynamically feasible trajectories specifically for quadrotors using **Minimum Snap**.

## 🕵️Overview

First we are going to look at approach [Mellinger et al., 2011] that generates minimum snap trajectory using a constrained Quadratic Programming (QP) formulation by leveraging the differential flatness property of quadrotors. After that we are going to look at the limitations of this approach and see how [Richter et al., 2016] builds on this idea and alleviates some of the issues by using an Unconstrained QP approach.
But first let’s understand Snap.

## ⚡Snap? What’s that?
![Fast](oh_snap.webp)

In Motion planning, Snap refers to the fourth derivative of position, the second derivative of acceleration, or the third derivative of velocity.
So, why not just focus on minimizing acceleration? Minimizing acceleration is definitely important, but it's not the whole story. Let's delve deeper and discover why snap plays a crucial role in generating smooth and efficient trajectories for robots!


## References
1. Mellinger, Daniel, and Vijay Kumar. "Minimum snap trajectory generation and control for quadrotors." 2011 IEEE international conference on robotics and automation. IEEE, 2011.

2. Richter, Charles, Adam Bry, and Nicholas Roy. "Polynomial trajectory planning for aggressive quadrotor flight in dense indoor environments." Robotics Research: The 16th International Symposium ISRR. Springer International Publishing, 2016.