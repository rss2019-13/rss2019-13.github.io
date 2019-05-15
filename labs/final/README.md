##**RSS Final Challenges**##

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vSekBXbRD01jhey_eg-2L4vt1B33mCh0rY9d0mkfi_EToSflpEm3nEaTdERE42kCxp2E9qdc8IwDWxN/embed?start=false&loop=false&delayms=3000).


The RSS Final Challenge includes two components. The first is a race around a loop in the Stata Center basement, which directly builds on components developed in labs 5 and 6. The other challenge is fast obstacle avoidance, which requires the robot to quickly navigate a randomly-generated course that it does not have a map for. This report will focus on the second challenge.

##*Fast Obstacle Avoidance*##

###*Overview (Andrew)*###
For this final challenge, our racecar needed to be able to successfully navigate an obstacle course that it had never seen before as quickly as possible. To do this, our car needed to detect obstacles in real time and quickly follow a route around them.

###*Proposed Approach*###

####*Obstacle Identification (Nada)*####
We used the racecar's LIDAR data to process the obstacles around it in real time. To do this, at every time step we took in the LIDAR data that the car was seeing. From here, we looked only at the angle sweep in front of the robot, which was from -pi/2 radians to pi/2 radians. This allowed us to only consider obstacles that could potentially get in the way of the car in the future, rather than anything it had already passed. 

We then stepped through the angle sweep at the angle increment that the LIDAR used to create the laserscan data. We set a lookahead distance such that we were only considering obstacles that were within a specified radius from the robot. 

From here, in order to find each distinct obstacle, we decided that any continuous angle sweep that was within the lookahead distance would count as a single obstacle. We then took note of the start and end of the obstacle in terms of the angle that it was located at, as well as the radial distance that it was located with respect to the racecar.

<img src="https://drive.google.com/uc?export=view&id=1XlLoluyORkRVosqr394GfXYmmRSiGaMC" alt="Obstacles in Rviz" height="462" width="583">
**Figure 1.1: Obstacle Detection**
Here, the racecar has looked at the LIDAR data within a specified lookahead distance and visualized each obstacle it found in Rviz.


Then we expanded the angle sweep by the width of the car such that the car would not cut too close that it would clip the obstacles.


<img src="https://drive.google.com/uc?export=view&id=1YKFQUuJhjK2Ive3knsb67nto8BX8Hul-" alt="Enlarged Obstacles" height="462" width="583">
**Figure 1.2: Enlarged Obstacles**
Here, the obstacles have been dilated by the width of the car. The initial obstacle size is denoted here by red markers, and the enlarged obstacles are in green.


We then added this enlarged obstacle in a tuple of the form (center, width, radial distance) to an obstacle list, and visualized each obstacle in RViz using markers. 

We then passed the generated obstacle list to our gap selection module, so that we could decide the best direction for the car to drive in order to avoid obstacles and continue navigating towards the goal.


####*Gap Selection (Andrew)*####

The gap selector is a function that receives a list of obstacles and a goal and produces an angle that the car should steer towards to avoid that obstacles while still making progress to the goal. It works by looking at the gaps between the obstacles, including the spaces between the end obstacles and the maximum and minimum lidar angles. It first checks if the goal angle is within any of the gaps and commands to steer in the goal direction if it is. In the case where there are gaps but the goal is behind an obstacle, it finds the gap that is closest to the goal angle and returns the end of this gap that minimizes the angle difference. Finally, if there are no gaps it commands the controller to lower speed and steers towards the most distant obstacle in the hopes that a gap will become visible before it reaches the obstacle.


####*Controller (Mia)*####

The controller in this case is proportional controller to the drive angle. The car keeps a constant speed, with the exception of slowing down if it sees no gaps.

###*Experimental Evaluation*###
####*Tuning*####

To improve the performance of our solution, we changed the values of both the maximum obstacle detection radius and the angle controller's proportional gain. Our goal while tuning these parameters was to allow the car to navigate cluttered, complicated courses at high speeds. We found this optimization to be more complicated than in previous labs, because the best values seemed to shift with the exact setup of the course. Even when we chose to optimize for a single course, we found that changing the parameters to allow it to navigate the end of the course more effectively would sometimes lead to it failing to navigate the beginning, or to take a different enough path through the start of the course that it would require a different tuning in the end. 

<iframe src="https://giphy.com/embed/dXL2XsFDAnHcsuh5vN" width="480" height="269" frameBorder="0" class="giphy-embed" allowFullScreen>
**Figure 2.1: Lookahead: 1 meter**

lookahead 1.5m
143122

lookahead 2m
142722


####*Final Results (Mia)*####

sparser
200225

slalom
143831

more dense
img_9171

The videos above demonstrate the cars performance in various types of courses varying from sparse obstacles, to a slalom, to densely packed obstacles. During the demo day we were able to complete the courses at the following speeds.

**Table 1: Drive Speed for Each Course Successfully Completed on Demo Day**

<img src="https://drive.google.com/uc?export=view&id=1fkCsD8pH_LXnvqlT0GA9uBFtqLf2Y1-J" alt=”Enlarged Obstacles" height="233" width="590">


###*Conclusion (Andrew)*###

We achieved our goal as the car was able to navigate a range of random course designs quickly without major collisions. The solution was not perfect though, as it required tuning of a few parameters if the course design or speed changed significantly. A specific set of parameters would allow the car to make it through a style of course (simple, slalom, complex) well, but it was very difficult to find a single configuration that performed on all course designs. Future versions of our solution could reduce the amount of manual tuning required by changing values like controller gain and lookahead distance automatically as functions of speed and measured distance between obstacles. This would allow the car to adapt to its surroundings instead of the operator having to judge the values that will lead to the best performance.
