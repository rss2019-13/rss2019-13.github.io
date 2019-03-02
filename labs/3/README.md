# Lab 3
For Lab 3, our goal was to give our robot wall-following abilities as well as a safety controller that prevented collisions. In order to do this, we created a hierarchy of programs that would allow the robot to efficiently follow walls without causing harm to itself or its surroundings. 

##Table of Contents
1. Wall Follower
2. Safety Controller

#Wall Follower
##Scan Parser
Our robot uses a LIDAR 2D laser scanner to detect objects in its path. We used this scan data to determine where the walls were with respect to the robot. To do this, we 

We then converted the laserscan data from polar coordinates to cartesian coordinates to make it easier to detect walls.
## Wall Detector

##Wall Follower Controller
Our wall follower consists of a PD controller that allows the robot to follow a wall to its left or right while maintaining a desired distance from the wall. After detecting a wall, the  program computes the distance and angle that the robot is positioned relative to the wall. It then determines the error in distance and angle from the desired state, and multiplies each error by some gain constant and publishes the sum to the 

#Safety Controller
##Scan Parser
##Object Detection
##Collision Prevention Controller
