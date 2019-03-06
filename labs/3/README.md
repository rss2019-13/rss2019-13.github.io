# Lab 3

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vQFcznMfWvz6esCwC0eGIOW0gzoU9PRd8C5C76ecpRsc0y0IkMSZnbuT8rqx0JtE4O_xf_ZyatpwM2w/embed?start=false&loop=false&delayms=3000).

#Overview and Motivations
For Lab 3, our goal was to give our racecar the ability to follow walls at some desired distance while also using a safety protocol to prevent unavoidable collisions. In order to do this, we created a hierarchy of programs that would allow the racecar to efficiently follow walls without causing harm to itself or its surroundings. A hierarchy of programs is needed to allow the racecar to navigate on its own while giving higher priority to the racecar’s safety controller to allow the racecar to detect potential obstacles and prevent crashes. The user still maintains highest priority and can intervene in the racecar’s navigation and safety controller at any time. 

The racecar is able to successfully navigate a path in its environment by following a wall and stopping when our program deems necessary. The wall follower we implemented is able to follow a chosen side of wall, navigate corners, and go around any foreseen obstacles while maintaining a given desired distance away from the wall. When the racecar detects an obstacle within a rectangular area in front of itself, and given the speed the racecar is travelling at and the distance to it, it has the option to either stop or go around it. If the racecar is travelling too quickly or too close to the obstacle to find a way around it, the racecar will come to a stop until the obstacle is removed. Otherwise, the racecar will go around the obstacle, if an alternative path is detected. For navigational purposes, it seems wise to have the racecar behave thus so, since the aim is to navigate safely while avoiding crashes.

***

## Table of Contents
1. Scan Parser
2. Wall Follower
3. Safety Controller

***

## Scan Parser
Our racecar uses a LIDAR 2D laser scanner to collect data about its surroundings. This is given to the car in a series of polar coordinates, which represent the distance to the nearest obstacle at any given angle in the sweep. We looked at specific subsets of this data that varied based on the task we were looking to accomplish. Once this data was filtered, we converted the laserscan data from polar coordinates to cartesian coordinates to make it easier to work with.

## Wall Follower
Our wall follower worked in three parts. First, we used the laserscan data to determine where the walls were relative to the robot. From here we determined the error of the car's position relative to its desired position from the closest wall, which can be set by giving the program different parameters. Finally, we implemented a controller that changed the turning angle of the car in order to direct it back to its desired position.

### Wall Detector
Once we received the LaserScan data from the LIDAR, we filtered out points that were not relevant. In the wall-following situation, this meant that we only looked at points to the left and in front of the racecar if it was following the left wall, and only looked at points to the right and in front of the racecar if it was following the right wall. From here, we were able to use a simple linear regression to consider all the LaserScan points on the correct side of the robot, and create a linear model for the wall that represented its location relative to the robot.

### Error Measurement
After locating the wall, we needed to know two things: the distance the robot was positioned from the wall, and the angle it was rotated relative to the wall. We used trigonometry and algebra to find these values. From here we calculated the error values by subtracting the measured distances and angles from the desired distance and angle, in which the angle we want is parallel to the detected wall.

### Wall Follower Controller
Using our error values, we implemented a PD controller that allows the racecar to adjust its turning angle based on its previous error from the desired position. We multiplied each error by some gain constant and published the sum to the Ackermann Drive message of the racecar as the turning angle. The racecar then adjusted its wheels to follow this turning angle, and from here out feedback cycle would start again, measuring the new errors from the LIDAR data and computing a new turning angle. 

## Safety Controller
When we implemented our safety controller, we wanted to make sure our racecar would be able to avoid harmful situations, such as crashing into walls, people, or other racecars. However, we wanted to make sure our racecar was not too scared of its surroundings. We created a controller that only stopped the racecar when certain it would collide with an obstacle otherwise, and allow it to adjust to avoid obstacles otherwise.

### Object Detection
As mentioned previously, our racecar took in LaserScan data from the LIDAR. We only wanted the the car’s safety protocol to come into play if there were obstacles directly in front of the racecar. To find these obstacles, we decided to only look at the subset of LaserScan points that were detected directly in front of the racecar.

### Detecting Unsafe Situations
Once we had determined the obstacles directly in front of the racecar, we needed to find a way to decide which obstacles the racecar still had time to avoid if we steered it away, and which ones were too close and required the racecar to stop immediately. 

### Collision Prevention Controller

***

##Experimental Evaluation
***

##Lessons Learned
