#Lab 5: Localization

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vTX9UHYMN6F9P_uf9IQ6b1OAXbz3sXFKd_jL8gPUZn_0H1Jb4tpJtq0qgGNKi-lr2JAXZma9b8WczIM/embed?start=false&loop=false&delayms=3000).

***

## **Overview and Motivations**
For lab 5, our goal was to give the robot the ability to find its position in a known environment. This is useful for future projects as in order for the robot to plan a path it first needs to know where it is. In order to do localize the robot, we use both interoceptive sensors measuring the wheel actuation and the robot’s LIDAR, an exteroceptive sensor. We use both of these measurements together because motion data is quick to calculate and LIDAR data corrects the error of the motion model. 

***

## Table of Contents
Motion Model
Sensor Model
Particle Filter
SLAM?
***

## **Proposed Approach**
### *Motion Model*
The motion model takes in an array of particles and updates them according to the odometry measurements from the robots internal sensors. The robot outputs an odometry message which gives an approximate position [x, y, ፀ].

###*Sensor Model (Nada)*
Once we had determined particle positions with the motion model, the sensor model used LIDAR data to filter particles based on probability. We calculated this probability based on four factors:

Probability of detecting a known obstacle in the map
Probability of a short measurement
Probability of a very large/missed measurement
Probability of a random measurement

These probabilities were defined as followed:
<img src="https://drive.google.com/uc?export=view&id=1BbGoKBRhd75FGShA9HBLQFdDmKtOmln7" alt="phit" height="100" width="400">
**Figure 3A: Calculating p_hit**
The probability of detecting a known obstacle in the map was represented as a gaussian distribution centered around the ground truth distance between the hypothesis pose and the nearest map obstacle.

<img src="https://drive.google.com/uc?export=view&id=1OcmCqA5pXyZKDzSe_bwgoy-HwejmLkWS" alt="pshort" height="100" width="400">
**Figure 3B: Calculating p_short**
The probability of a short measurement was represented as a downward sloping line as the ray gets further from the robot.

<img src="https://drive.google.com/uc?export=view&id=1N1YYRUOQmxf5M7G9-SabJ45citCjG8av" alt="pshort" height="100" width="400">
**Figure 3C: Calculating p_max**
The probability of a very large measurement was represented as a large spike in probability at the maximal range value.

<img src="https://drive.google.com/uc?export=view&id=10rHN6TO6L8_ED4lO6_n5HDSM0goDL5BL" alt="pshort" height="100" width="400">
**Figure 3D: Calculating p_rand**
The probability of a random measurement was represented by a small uniform value.


From here, we mixed these four distributions by a weighted average as follows:

<img src="https://drive.google.com/uc?export=view&id=17wS-hThxEVmGtmoKbmubv5UmoW5vYJbA" alt="pshort" height="75" width="600">
**Figure 4: Calculating p_total**






##*Precomputing the Sensor Model (Nada)*
In order to speed up computation, we precomputed a discretized sensor model array that we could use to simply look up any values for z_t and z_t*. To do this, we computed all p_total values for all combinations of z_t and z_t* star in the range of 0 to 10, where we incremented each value by 0.1 each time we computed the probability. 


##*Applying the Sensor Model (Nada)*


## **Experimental Evaluation**

### *In Simulation*

Sensor Model


### *On the Racecar*

## **Lessons Learned (Nada)**



