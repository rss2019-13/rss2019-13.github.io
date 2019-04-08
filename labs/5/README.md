#Lab 5: Localization

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vTX9UHYMN6F9P_uf9IQ6b1OAXbz3sXFKd_jL8gPUZn_0H1Jb4tpJtq0qgGNKi-lr2JAXZma9b8WczIM/embed?start=false&loop=false&delayms=3000).

***

## **Overview and Motivations** (Mia)
For lab 5, our goal was to give the robot the ability to find its position in a known environment. Localization is important for our future tasks in path planning and following. Without knowing where the racecar is, it is impossible to follow a path. We can do this by integrating the velocity of the robot as given by the interoceptive sensors measuring wheel motion. However as the car drives, error is builds up between this estimated position and the actual position, as a result of the surface it is driving on as well as wheel alignment and servo error. Therefore we must not solely rely on interoceptive sensors to localize. Instead we use a particle filter which takes input from both the interoceptive motion sensors, and LIDAR data. The particles in the particle each represent a guess for the robot’s position and heading. As the car drives, the motion model updates each particle based on the odometry. Then at a frequency of 8 Hz, the particle filter prunes the particles based on whether the LIDAR scan matches with the position, duplicating the points that were not pruned. After testing in both the simulator and on the actual robot, we tuned the motion and sensor models to get better results.

***

## **Proposed Approach**
### *Motion Model* (Mia)
The motion model takes in an array of particles and updates them according to the odometry measurements from the robot's internal sensors. The robot outputs an odometry message which gives an approximate position [x, y, ፀ] in a global frame. The motion model takes the difference between the current position and the previous position to get [dx, dy, d\theta] in the odometry frame.

<img src="https://drive.google.com/uc?export=view&id=14jsAD1c1eZi8pzeezz6YRyr2v_RCTtpp" alt="Odometry Frame" height="474" width="573">

Then the differences in position are rotated by the negative of its angle in odometry coordinates.

<img src="https://drive.google.com/uc?export=view&id=1t7Kh0-cNQcf1TxZq201GpsRCQtQeNdw1" alt="Car Frame Rotation" height="100" width="496">
<img src="https://drive.google.com/uc?export=view&id=1uKtovqCwPBCsbLXBirO6edtgLkD7kRoS" alt="Car Frame" height="462" width="583">



Then noise is added to each particle randomly using a normal distribution. We will tune the noise values later.

<img src="https://drive.google.com/uc?export=view&id=1G2fua__Wffb1HbkPeoUB7yR7yNL0-iJK" alt="Adding Noise" height="79" width="464">

Then the differences in position are rotated to map coordinates according to each particle’s angle.

<img src="https://drive.google.com/uc?export=view&id=1YQ-_vXY8JTIcpgq0jOMs3MLTunQAqs5S" alt="Map Rotation" height="85" width="464">

<img src="https://drive.google.com/uc?export=view&id=10MG8khs0M42zed0mdMHaLdvmIYQTIwO0" alt="Map Frame" height="587" width="705">

Then each change in position is added to its particle.


###*Sensor Model (Nada)*
Once we had determined particle positions with the motion model, the sensor model used LIDAR data to filter particles based on probability. In this way, we were able to use our LIDAR sensor data and the robot's current particle distribution, and compute the probability of receiving our LIDAR readings given the car location denoted by each particle in our motion model distribution. We then used these probabilities to update particle weights and determine the most likely car position.

We calculated each particle’s likelihood based on four factors: the probability of detecting a known obstacle in the map, the probability of a short measurement, the probability of a very large/missed measurement, and the probability of a random measurement.

These probabilities were defined as followed:

<img src="https://drive.google.com/uc?export=view&id=1BbGoKBRhd75FGShA9HBLQFdDmKtOmln7" alt="phit" height="100" width="400">
**Figure 3A: Calculating p_hit**
The probability of detecting a known obstacle in the map was represented as a gaussian distribution centered around the ground truth distance between the hypothesis pose and the nearest map obstacle.


<img src="https://drive.google.com/uc?export=view&id=1OcmCqA5pXyZKDzSe_bwgoy-HwejmLkWS" alt="pshort" height="100" width="400">
**Figure 3B: Calculating p_short**
The probability of a short measurement was represented as a downward sloping line as the ray gets further from the robot. This could happen if an object that was not accounted for in the map was detected by the robot before a wall was.


<img src="https://drive.google.com/uc?export=view&id=1N1YYRUOQmxf5M7G9-SabJ45citCjG8av" alt="pshort" height="100" width="400">
**Figure 3C: Calculating p_max**
The probability of a very large measurement was represented as a large spike in probability at the maximal range value. This would occur if the ground truth distance was larger than the maximum LIDAR reading distance. 


<img src="https://drive.google.com/uc?export=view&id=10rHN6TO6L8_ED4lO6_n5HDSM0goDL5BL" alt="pshort" height="100" width="400">
**Figure 3D: Calculating p_rand**
The probability of a random measurement was represented by a small uniform value.


From here, we mixed these four distributions by a weighted average as follows:

<img src="https://drive.google.com/uc?export=view&id=17wS-hThxEVmGtmoKbmubv5UmoW5vYJbA" alt="pshort" height="75" width="600">
**Figure 4: Calculating p_total**
To find the total probability of any given particle denoting the robot's actual position, we simply added up all the individual probabilities, each multiplied by some factor such that the factors summed to one. 




##*Precomputing the Sensor Model (Nada)*
In order to speed up computation, we precomputed a discretized sensor model table that we could use to simply look up any values for z_t and z_t\*. To do this, we computed all p_total values for all combinations of z_t and z_t\* in the range of 0 to z_max, incrementing z_t and z_t\* by 0.1 each time. In doing this, we were able to simply look up any probability given a z_t and z_t\* value, which sped up our computation significantly. 


<img src="https://drive.google.com/uc?export=view&id=15ePjn5CkA6SGagCsABDTC4qZRBxjKtNQ" alt="probgoal" height="250" width="400">
**Figure 5: Probability Distribution of Precomputed Model**
For any given ground truth distance (shown at the green line), a cross section like this one showed us the probability distribution of reading some measured distance from the LIDAR scan.


Once we had these cross sections, we were able to create the entire lookup by creating the cross section for each given ground truth distance, and normalizing each distribution. Here you can see what this probability distribution looked like.
 

<img src="https://drive.google.com/uc?export=view&id=14FQ-v9YAzApMxqbjjSzSEOzVOu5Ja4Kr" alt="probdist" height="250" width="400">
**Figure 6: Probability Distribution**
This probability distribution shows all combinations of z_t and z_t\*. 



##*Applying the Sensor Model (Nada)*
Once we had a precomputed lookup table, we could take in some particles from the motion model as well as the LIDAR observations, and use this data to find the likelihood of each motion model particle accurately denoting the robot’s position on the map. From here, we were able to choose the higher likelihood particles and update our pose estimate based on those particles. 



## **Experimental Evaluation**

### *In Simulation*

We first tested the particle filter on the simulated car. We created a copy of the car’s odometry data with noise added and attempted to localize given this noise. The noise chosen included a random factor with mean 1 and SD 0.01 multiplied onto the odometry vector as well as additive noise with a mean of 0 and an SD of 0.01. This appeared to recreate the noise effects we saw on the real car at our chosen driving speed of 1 m/s. 

The filter is 


### *On the Racecar*

Though not as reliable as in simulation, the particle filter performs well on the car and is able to reliably update the car’s pose. It is much more difficult to measure error on the physical car than it is in the simulation, so the results in this section will largely be observations about the strengths and weaknesses of the algorithm when it is run on the car. 

We did all of our testing in the basement of the Stata center and found that in hallway close to 32-044 produced very good results. The hallway has enough complicated features that there are rarely ambiguous positions, so the sensor model is able to update the car’s position accurately even as the motion model drifts away from the real result slightly. We found that it was important to tune the relative frequency of motion model and sensor model updates. The motion model is updated every odometry message, while the sensor model is only evaluated every five scans. If this ratio is too large the sensor model is not run frequently enough and the pose is allowed to drift with odometry error, but if it is run too frequently the car’s motion is typically ignored as the results are effectively overridden by the sensor model.

On hallways such as the wider rectangular spaces further from the classroom, the car can sometimes struggle as the sensor model can not uniquely identify its position along the hall. Movement along the minor axis of the hallway is tracked well, but the sensor model is not able to fix odometry errors in movement along the major axis of the hallway because the scan data is approximately constant along this axis.

The filter functions very well at speeds up to 1.5 m/s. If acceleration is kept reasonable the car can still localize at higher speeds in the proper environment, but errors seem to accumulate faster and the chances of divergence increase. With further tuning the filter’s performance at higher speeds could be improved.

## **Lessons Learned (Nada)**

Although we attempted to keep our past lessons learned in mind while doing this lab, we definitely still came across similar problems for this lab. We learned that these labs always take longer than we expect, and although we attempted to front-load the work for this lab, we still worked right up to the deadline in order to get this lab working. In the future, we will continue to work ahead of schedule as much as possible to ensure that we are able to get everything done in time and satisfactorily. 

We also learned during this lab that it is important to maintain good communication when working on these labs. It was difficult to meet for this lab due to spring break and other obligations, and making sure we all stayed in contact and communicated out schedules was crucial to us being able to work through this lab. 

We also learned that the robot behaves very different in the simulator than in real life. Tuning the robot to work well in the simulator results in malfunction in real life and tuning the robot to work in real life did not perform well in the simulator.



