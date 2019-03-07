# Lab 3

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vQFcznMfWvz6esCwC0eGIOW0gzoU9PRd8C5C76ecpRsc0y0IkMSZnbuT8rqx0JtE4O_xf_ZyatpwM2w/embed?start=false&loop=false&delayms=3000).

## **Table of Contents**
1. Scan Parser
2. Wall Follower
3. Safety Controller

***

## **Overview and Motivations**
For Lab 3, our goal was to give our racecar the ability to follow walls at some desired distance while also using a safety protocol to prevent unavoidable collisions. In order to do this, we created a hierarchy of programs that would allow the racecar to efficiently follow walls without causing harm to itself or its surroundings. This structure is needed to allow the racecar to navigate on its own while giving higher priority to the safety controller, which allows the racecar to detect potential obstacles and prevent crashes. The user still maintains highest priority and can intervene in the racecar's navigation and safety controller at any time. 

The racecar is able to successfully navigate a path in its environment by following a wall and stopping when our program deems necessary. The wall follower we implemented is able to smoothly follow a chosen side of wall, navigate corners, and go around any foreseen obstacles while maintaining a given desired distance away from the wall. When the racecar detects an obstacle within a rectangular area in front of itself, given the speed that the racecar is travelling at and the distance to the obstacle, it has the option to either stop immediately or go around the obstacle. If the racecar is travelling too quickly or is too close to the obstacle to find a way around it, the racecar will come to a stop until the obstacle is removed. Otherwise, the racecar will go around the obstacle, if an viable alternative path is detected. For navigational purposes, it seems wise to have the racecar behave thus so, since the aim is to navigate safely while avoiding crashes. 


***

## **Proposed Approach**

## *Scan Parser*
Our racecar uses a LIDAR 2D laser scanner to collect data about its surroundings. This is given to the car in a series of polar coordinates, which represent the distance to the nearest obstacle at any given angle in the sweep. We looked at specific subsets of this data that varied based on the task we were looking to accomplish. Once this data was filtered, we converted the LIDAR data from polar coordinates to cartesian coordinates to make the points easier to work with.

## *Wall Follower*
Our wall follower worked in three parts. First, we used the laserscan data to determine where the walls were relative to the robot. From here we determined the error of the car's position relative to its desired position from the closest wall, which can be set by giving the program different parameters. Finally, we implemented a controller that changed the turning angle of the car in order to direct it back to its desired position.

### Wall Detector
Once we received the LaserScan data from the LIDAR, we filtered out points that were not relevant. In the wall-following situation, this meant that we only looked at points to the left and in front of the racecar if it was following the left wall, and only looked at points to the right and in front of the racecar if it was following the right wall. From here, we were able to use a simple linear regression to consider all the LaserScan points on the correct side of the robot, and create a linear model for the wall that represented its location relative to the robot.

### Error Measurement
After locating the wall, we needed to know two things: the distance the robot was positioned from the wall, and the angle it was rotated relative to the wall. We used trigonometry and algebra to find these values. From here we calculated the error values by subtracting the measured distances and angles from the desired distance and angle, in which the angle we want is parallel to the detected wall.

### Wall Follower Controller
Using our error values, we implemented a PD controller that allows the racecar to adjust its turning angle based on its previous error from the desired position. We multiplied each error by some gain constant and published the sum to the Ackermann Drive message of the racecar as the turning angle. The racecar then adjusted its wheels to follow this turning angle, and from here out feedback cycle would start again, measuring the new errors from the LIDAR data and computing a new turning angle. 

## *Safety Controller*
The safety controller is important to make sure the robot does not crash into walls while driving autonomously. It has to be able to protect the racecar from running itself into objects, but not interrupt the wall follower when there is no danger. To accomplish this we created a controller that triggers only if there is an object directly in front of the car.

### Object Detection
As mentioned, our racecar took in LaserScan data from the LIDAR. We only wanted the safety controller's override to activate if there were obstacles directly in front of the racecar. To find these obstacles, we decided to only look at the subset of LaserScan points that were within a rectangle in front of the car. This rectangle, the safety lockout region,  has the same width as the racecar and a length dependent on the current velocity. This length scaling was necessary to allow the safety controller to have the right level of caution at different speeds, because the racecar's stopping distance is a function of its velocity. The safety controller compares all laser scan points within a certain angle range to see if they are within the safety lockout region. To avoid the controller being too sensitive to noise, we implemented a configurable threshold that the number of points inside of the lockout region must exceed before the car is stopped. Based on experimental data, we chose a value of five for this threshold.

![](https://drive.google.com/file/d/18pBTRBkcVbOLTtaEQwphwP7SrstPoKqr/preview)

### Collision Prevention Controller
When an unsafe condition is detected, the safety controller overrides the output from the wall follower by setting the robot's velocity setpoint to zero. This override will stay active as long as the laser scan data suggests that an obstacle is too close to the robot. When the obstruction is cleared, the wall follower continues operation seamlessly.

***

## **Experimental Evaluation**
Throughout the development process we tested the racecar in real world conditions.
### *Wall Follower*
We first evaluated the wall follower in a simulator to tune the control system. Here is a plot of the distance and angle errors as the robot navigates a 90 degree corner.

Once we were satisfied with performance in the simulator we tested the wall follower at varying speeds and follow distances and found that the wall follower was robust until the racecar followed the wall into a corner that is smaller than the racecar's turning radius.


### *Safety Controller*
The safety controller was difficult to test because our wall follower is very good at navigating the racecar around obstacles. However, the safety controller was still needed for smaller obstructions and obstacles suddenly getting in the way. Below are examples of the safety controller in action. As you can see in the second video, as soon as the obstacle leaves the lockout region, the racecar continues following the wall.

## **Lessons Learned**
To ensure more efficient group work, rigorous testing of edge cases and better delegation of tasks are needed. The initial approach we took to detecting obstacles in a collision zone was to detect LaserScan points in a triangular area in front of the racecar. However, we found that this method did not cover all cases of potential obstacles, since points directly in front of the left or right edges of the racecar would not be detected and the racecar would collide into those undetected obstacles. We then changed our approach to identifying obstacles in a rectangular area in front of the racecar, which works well. 

The new approach of using a rectangular collision area caused the racecar to have delayed reactions to obstacles, which was solved by only processing a subset of LaserScan messages received. The computation required by the new approach is slower, and the racecar is unable to process messages as quickly as they are received. As older LaserScan messages were being dequeued before they could be processed, the racecar had delayed reactions to obstacles faced and could not stop in time. Our solution to this problem was to process one in every five LaserScan messages received, which still allowed the racecar to detect new obstacles quickly enough and process the LaserScan messages to react appropriately. 

To ensure that future labs take less time and group work is more efficient, better delegation of tasks is needed. Group work time would be better used merging different working parts of a program rather than tackling a single problem at a time as a group.