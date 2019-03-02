# Lab 3
For Lab 3, our goal was to give our robot wall-following abilities as well as a safety controller that prevented collisions. In order to do this, we created a hierarchy of programs that would allow the robot to efficiently follow walls without causing harm to itself or its surroundings. 

## Table of Contents
1. Scan Parser
2. Wall Follower
3. Safety Controller

## Scan Parser
Our racecar uses a LIDAR 2D laser scanner to collect data about its surroundings. This is given to the car in a series of polar coordinates, which represent the distance to the nearest obstacle at any given angle in the sweep. We looked at specific subsets of this data that varied based on the task we were looking to accomplish. Once this data was filtered, we converted the laserscan data from polar coordinates to cartesian coordinates to make it easier to work with.

## Wall Follower
Our wall follower worked in three parts. First, we used the laserscan data to determine where the walls were relative to the robot. From here we determined the error of the car's position relative to its desired position. Finally, we implemented a controller that changed the turning angle of the car in order to direct it back to its desired position.

### Wall Detector
Once we had received the LaserScan data from the LIDAR, we filtered out points that were not relevant. In the wall-following situation, this meant that we only looked at points to the left and in front of the racecar if it was following the left wall, and only looked at points to the right and in front of the racecar if it was following the left wall. From here, we were able to use a simple linear regression to consider all the LaserScan points on the correct side of the robot, and create a linear model for the wall that represented its location relative to the robot.

### Error Measurement
Once we knew where the wall was located, we needed to know two things: the distance the robot was positioned from the wall, and the angle it was rotated relative to the wall. We used trigonometry and algebra to find these values. From here we calculated the error values by simple subtracting the measured distances and angles from the desired distance and angle, in which the angle we want is as parallel to the wall as possible.

### Wall Follower Controller
Using our error values, we implemented a PD controller that allows the racecar to adjust its turning angle based on its previous error from the desired position. We multiplied each error by some gain constant and published the sum to the Ackermann Drive message of the robot as the turning angle. The racecar then adjusted its wheels to follow this turning angle, and from here out feedback cycle would start again, measuring the new errors from the LIDAR data and computing a new turning angle. 

## Safety Controller
### Object Detection
### Detecting Unsafe Situations
### Collision Prevention Controller
