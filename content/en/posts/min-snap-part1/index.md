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

_Minimizing Snap creates elegant motion profiles_

## 🥫Differential Flatness: The secret sauce!
Differential flatness is a potent concept in control theory that can be applied to the trajectory optimization problem for quadrotors to generate dynamically feasible trajectories. 

A set of variables in a system is considered differentially flat if we can reconstruct all the system's states and control inputs using just a chosen few of these variables and their derivatives. It might sound like a magic trick, but there's a clear logic behind it! Let's unravel this concept with the help of a quadrotor model example.


![Differential Flatness](diff_flat.jpg)

For a quadrotor, if we pick the 3D position (x, y, z) and yaw angle (ψ) as our flat outputs, then remarkably, we can express all other states (velocity, acceleration, etc.) and the control input (motor thrust) as functions of these chosen flat outputs and their derivatives.

Let’s look at some straightforward equations to drive this point home.

- **Flat outputs**: [x,y,z,ψ]
- **States**
    - Velocity: v = [ẋ, ẏ, ż] is just the first order derivative of flat outputs  x, y, z
    - Acceleration: a = [ẍ, ÿ, z̈] is just the second order derivative of  x, y, z. 

    - Roll: $\varphi = \sin^{-1} \left( \frac{\ddot{x} \sin \psi - \ddot{y} \cos \psi}{g} \right)
$
    - Pitch: $\theta = \sin^{-1} \left( \frac{\ddot{x} \cos \psi + \ddot{y} \sin \psi}{g \cos \varphi} \right)
$

Jerk and Snap can be represented in a similar way. To drive the point home further, we can look at attitude equations above that show  roll and pitch can also be represented just using the derivatives of the differentially flat variables we chose.

### From Theory to Flight: How Flatness Simplifies Trajectory Planning

The gargantuan advantage of the differential flatness property is that we don’t have to optimize over the entire state space. We can just solve for the flat outputs (x,y,z, yaw) and the rest of the states and inputs can be obtained using the equations above.

And it doesn’t stop there! This property also ensures that the generated trajectories are **dynamically feasible**. This is due to the fact that whatever values we obtain for the inputs using the flat output variables are calculated using the dynamics of the system. Let’s look at the example below to fully grasp this concept.

Let’s say we solve for flat outputs X = [x, y, z, ѱ]. Then we can obtain the states (velocity, acceleration, ..etc.) and inputs (Thrust from motors) using these values as shown above.
After computing acceleration, we can use that to calculate Thrust as:

$$
T = m \sqrt{\ddot{x}^2 + \ddot{y}^2 + (\ddot{z} + g)^2}
$$

Since acceleration, Thrust and other states (pitch,roll,etc.) are computed using the equations of motion, the generated trajectories are guaranteed to be dynamically feasible.

## Minimum Snap, Maximum Control

Now, let’s do what we came here for and understand the problem setup for minimum snap trajectory generation.

- **Objective:** Generate a minimum snap trajectory for a quadcopter.
- **Inputs:**
    1. 3D position of waypoints.
    2. Boundary conditions: Desired velocities and acceleration for start and end point
    3. Maximum velocity of the quadcopter.

- **Constraints:**
    1. Fixed start and end states. 
    2. Trajectory passes through the waypoints

- **Notation:**

    ![Notation](notation.jpeg)
The notation in the rest of this article to show trajectory terms is defined above. For instance, $x^{2}_{0,1}$ represents the 2nd derivative (acceleration) for segment 1 at time 0.

![Notation](wpts.png)
_For our example we have 3 waypoints with 2 segments connecting the waypoints_

## 🧪Formulation
The first step is to represent our desired trajectory using a polynomial equation. In order to minimize snap we need at least a seventh degree polynomial as shown below.

$$
f(t) = p_7 t^7 + p_6 t^6 + p_5 t^5 + p_4 t^4 + p_3 t^3 + p_2 t^2 + p_1 t + p_0
$$

### Why the Magic number 7️⃣?

The reason we need at least a 7th degree polynomial to solve for a minimum snap trajectory is due to the number of unknown boundary conditions (explained in more detail later). 
For minimizing snap over segment, we need to solve for the following variables at the start and end points:

1. Position (0th derivative)
2. Velocity (1st derivative)
3. Acceleration (2nd derivative)

The total unknowns are 6 (3 for each start and end waypoint of each segment). Moreover, we can strategically set certain coefficients to zero based on the chosen boundary conditions (e.g., setting jerk to zero at the beginning). This allows us to use a 7th-degree polynomial to represent a minimum snap trajectory with 7 unknowns.

For our example, let's break down the trajectory into segments. The polynomial for each segment i is represented by a polynomial $f(t)$ as shown below. 
![Segments](segments_wpts.png)

We assume each segment starts at time $T_{i-1}$ and ends at time $T_i$, i.e time interval for segment i, $t_i ∊ [T_{i-1}, T_i]$. For now let’s assume the time interval is fixed for each segment. In the next part of this blog, we will see how we can optimize for this parameter as well. The equation below shows one possible way to compute the time for a segment i using the distance between waypoints and the maximum velocity of the vehicle.

$$T = \frac{||(x_1, y_1) - (x_2, y_2)||}{V_{max}}
$$

## What We Know, What We Don't: The Known and the Unknowns of the problem

It is crucial to understand the known and unknown entities. As shown in the figure below, we know the positions of all the waypoints and we can also set the desired velocities and accelerations for the start and end waypoints.
![knowns_unknowns](knwons_unknowns.png)

This brings us to the more interesting part, THE UNKNOWNS! 

<img src="https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExejF0dGtwazh1dXBleXJndmQxdmZtMXBrN2ppN3E2M2ZwdWgwanQwYSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/UuGjHrJw6JB4aFHU15/giphy.gif" alt="the_unknown" width="480" />

Let's zoom out and consider the bigger picture of trajectory generation. We aim to utilize provided waypoints to design a smooth motion path for the quadcopter to follow. This translates to pinpointing the exact velocities, accelerations, and so on, that the drone needs to maintain at each waypoint. While we typically have the desired values for the start and end points, the challenge lies in determining the first-order and higher derivatives (velocity, acceleration, etc.) for the intermediate waypoints. In our simple example above, there's only one such intermediate point: waypoint 2.

## 🪙Cost Function



## References
1. Mellinger, Daniel, and Vijay Kumar. "Minimum snap trajectory generation and control for quadrotors." 2011 IEEE international conference on robotics and automation. IEEE, 2011.

2. Richter, Charles, Adam Bry, and Nicholas Roy. "Polynomial trajectory planning for aggressive quadrotor flight in dense indoor environments." Robotics Research: The 16th International Symposium ISRR. Springer International Publishing, 2016.