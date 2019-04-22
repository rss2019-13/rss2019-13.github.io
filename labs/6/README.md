#Lab 6: Path Planning

Our briefing slides can be found [here](https://docs.google.com/presentation/d/e/2PACX-1vSbTrn6288ExKgKn6zxwXhcQU8j9JTAl1g2xeSfVyBx5OOaQ8wHcma-ttG-lWlHgJ4d9O_niWeJlsgf/embed?start=false&loop=false&delayms=3000).

***

## **Overview and Motivations** (Nada)
For Lab 6, we had two goals: plan trajectories in an occupancy grid given a start and end pose, and program the racecar to follow a predetermined trajectory around Stata basement. Our ultimate goal was to combine these two features, such that our racecar was able to perform real-time path-planning and execution. These features were important as they are the core factors involved in autonomous driving. Without planning and control, our robot would not be able to navigate an environment efficiently and safely. With this lab, our car is finally able to reliably drive autonomously.

We tested two different approaches to path planning in this lab: search planning and sample planning. We compared these two approaches based on their efficiency and accuracy, and after considering both implementations, decided to proceed with sample planning trajectories.

Once we successfully had our racecar determining efficient paths, we implemented a pure-pursuit controller that worked with our particle filter from Lab 5. From here, the racecar was able to quickly, accurately, and safely follow the trajectory determined via path-planning in the real world, enabling autonomous driving.

***

## **Proposed Approach**

### *Search Planning* (Nada)
In order to implement a search-based path-planning algorithm, we decided to use A* search on our racecar. 

Before we could do this, we needed to account for the robot’s dimensions such that it would not clip any obstacles on its edges. To do this, we dilated all obstacles on the map, such that the point representation of the robot avoiding the dilated obstacles meant that the full body of the robot would avoid the real-world obstacles successfully.

<img src="https://drive.google.com/uc?export=view&id=1_FznJNFJbGu9pMNg4KvTh-57gCy-Lcs4” alt=”Dilated Map” height=”462” width="583">
**Figure 1: Dilated Map**
Here, the map has been dilated such that all obstacles have a bumper added to them to ensure the sides of the car do not collide with anything.

Once we had a dilated map, we were able to implement a search algorithm. Once the robot receives clicked points in Rviz, it sets these points as its start and end points. From here, we set up a queue of neighbors to explore. This queue was ordered such that the closest neighbor to the current point was always listed first. 

At any given iteration of A Star, we looked at the closest neighbor currently in the queue. We then checked if this neighbor was within a reasonable margin of the end point. If it was, we had successfully solved the problem, and could then return the trajectory, which was then published in Rviz. 

If the closest neighbor, or our current node, was still too far from the goal, we expanded our search by finding all neighbors to the current node that did not interfere with an obstacle. The process for this is shown in the image below.

<img src="https://drive.google.com/uc?export=view&id=1m7anzZFBixMRwIC5-cpLYWW0RIgwRMJJ” alt=”Finding Neighbors” height=”462” width="583">
**Figure 2: Process of Finding Neighbor Nodes**
To find the neighbors of a node, our implementation had a set distance away from the current node that it looked for new nodes. The program then swept the car’s feasible turning angles, and stepped through this angle sweep at the given radius, and set those points as the new neighbor nodes to explore.

 We then made sure these neighbors were not interfering with the locations of the obstacles, and once they had been determined to be safe, added the neighbors to the queue to be explored on a later iteration of A*. 

During each round, we also updated the cost of each node. This cost was how much distance the path had taken to get to some node in the trajectory. This cost was updated any time a node was reached in a shorter distance than we had initially set its cost to be. In this way, we were able to ensure a shortest path would be found by this algorithm. 

<img src="https://drive.google.com/uc?export=view&id=1CecYPDg6d_jOoD1XM-EUUR1p70w17WnQ” alt=”Path” height=”462” width="583">
**Figure 3: Path Found Using A***
Here, the trajectory that the algorithm found is seen as a blue line on the map, and the nodes that were explored to reach this trajectory are marked as green points along the way. 


## *Sample Planning*
### *RRT* ()

###*RRT star* ()
Because RRT does not produce optimal paths, we used the RRT* modification. RRT* samples points in the same way as RRT, but after it checks that there are no obstacles to the newly added point, RRT* checks if there is a better way to connect the nodes of the tree. It does this by looping through the nodes within a certain radius of the new node and finding which of these nodes gives the shortest obstacle free path to the new node. Then RRT* checks if any other nodes can be reconnected through the new node to make the path leading up to them shorter.

##*Pure Pursuit* (Eric)
Now that we have a path from where the robot is to where we want it to be, we need to program the robot to follow that path. 


## **Experimental Evaluation**

### *In Simulation*

### *Search Planning* (Nada)

Our search-planning approach was very good at finding shortest paths. Because our implementation explored the nodes closest to the goal first, we guaranteed that we would find the optimal path. However, it took a lot of time to come up with the path using this method. This implementation took about a minute of set up time in order to find all the obstacles in the map, and even then could sometimes take up to a minute to find a path. 

[insert video of path planning]


### *Sample Planning* 

The sample planner is able to generate acceptable paths that the car can follow successfully in significantly less time than the search planner. Though these paths are not guaranteed to be the optimal route as is the case for the search planner, the use of RRT* for the path planning algorithm means that the generated paths are typically shorter than they would with traditional RRT. It also allows for a tradeoff between calculation time and path length, because RRT* has parameters that can be changed to produce better paths given more time. 

For routes between most points in the Stata basement map, the sample planner is able to find a path in well under a minute


### *On the Racecar*

## **Lessons Learned**



