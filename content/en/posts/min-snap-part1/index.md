+++
title = 'Minimum Snap Trajectory Generation-Part 1'
date = 2024-07-19T07:28:02-04:00
draft = false
math = true
tocOpen = true
+++
# Minimum Snap Trajectory Generation-Part 1


## 👋Introduction
In robotics, motion planning addresses the challenge of moving a robot from a starting point (point A) to a desired goal point (point B). The specific approach to motion planning depends heavily on the type of robot and its environment. For instance, the solution for an industrial robot arm's end effector to reach a specific point will be vastly different from that needed for a quadcopter to navigate a complex environment.

<img src="https://moveit.picknik.ai/humble/_images/grasp_pipeline_demo.gif" alt="Grasp Planning for a robot arm" width="400"/>

_Grasp Planning for a robot arm. 
Source: https://moveit.picknik.ai_


<img src="https://github.com/HKUST-Aerial-Robotics/Fast-Planner/blob/master/files/icra20_3.gif?raw=true" alt="FAST Planner" width="400"/>

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

 **😭Velocity**: Minimizing velocity can lead to huge jumps in acceleration, which could result in sudden start and stops.

<img src="https://media4.giphy.com/media/4aRCnf3LuPnRzlEE6A/200w.gif?cid=6c09b952s9e78bayjxdtfag9x7cymxrdp2e5xr2mqlhxp0kp&ep=v1_videos_search&rid=200w.gif&ct=v" alt= "Sudden Stop" width="300"/>

_Minimizing only velocity can lead to sudden and unsafe braking_

😓 **Acceleration**: Minimizing acceleration can lead to discontinuous jumps in jerk, which would result in jerk motion. This is not suitable for applications where tracking is important.

<img src="https://media.tenor.com/PiLg9bsD7NYAAAAM/car.gif" alt="Vehicle shaking" width="300" />

_Focusing only on acceleration can sometimes lead to jumps in jerk causing damange to vehicle componenets_

🙂 **Jerk**: There are some approaches that minimize jerk. Specifically, in applications for tracking, minimizing jerk might be enough and is less computationally intensive than minimizing snap. However, minimizing snap takes it a step further and generates even smoother flight profiles. This is important when flying with delicate cargo or trying to generate efficient motion profiles.

<img src="https://www.diyphotography.net/wp-content/uploads/2017/03/follow_mode.gif" alt="Drone tracking a target" width="400"/>

_Minimizing jerk is a good approach for tracking targets_

😃 **Snap**
There are a lot of inherent advantages of minimizing snap over minimizing velocity and acceleration so let’s look and understand some of the most significant ones.

Continuity in control inputs
Snap-minimized trajectories ensure continuity up to the third derivative of position (jerk), which means the control inputs required to follow the trajectory will be continuous.
Compatibility with differential flatness:
Quadrotor dynamics are differentially flat with respect to the position and yaw angle. Minimizing snap aligns well with this property, making it easier to generate dynamically feasible trajectories.
Continuity in control inputs:
Snap-minimized trajectories ensure continuity up to the third derivative of position (jerk), which means the control inputs required to follow the trajectory will be continuous.
This is particularly important for aggressive maneuvers where discontinuities in lower-order derivatives could lead to instability.

These properties help generate smooth and elegant trajectories for quadcopters when minimizing snap. Furthermore, minimizing snap also helps reduce the stress on actuators by preventing sudden changes in thrust.

![DIF FPV Smooth](https://images.squarespace-cdn.com/content/v1/58fd838a9f7456e0b92a446a/1615414861152-U322WCC9G8ERL5TVF9YH/170-drone-video.gif)

_Minimizing Snap creates smooth motion profiles without excessive jerk_
## References
1. Mellinger, Daniel, and Vijay Kumar. "Minimum snap trajectory generation and control for quadrotors." 2011 IEEE international conference on robotics and automation. IEEE, 2011.

2. Richter, Charles, Adam Bry, and Nicholas Roy. "Polynomial trajectory planning for aggressive quadrotor flight in dense indoor environments." Robotics Research: The 16th International Symposium ISRR. Springer International Publishing, 2016.