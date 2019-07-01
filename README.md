# *RoboND_Search_and_Sample_Return*

This project covers the implementation of all three essential elements of robotics: **Perception, Decision Making, and Actuation** for a **Mars Rover**'s autonomous navigation, mapping, collecting sample rocks and return to the original start point in a simulated environment.

**Unity** are used for simulating the environment in this project and **Jupyter Notebook with Python** is choosed to write most of the code.

This is a `README` that includes all the key points and how I addressed each one.

**Steps to complete the project:**  


1. Download the simulator appropriate for your OS ([MacOS](https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Mac_Roversim.zip), [Linux](https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Linux_Roversim.zip), [Windows](https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Windows_Roversim.zip)).
2. Get setup with Python using the [Python Starter Kit](https://github.com/udacity/RoboND-Python-StarterKit/blob/master/doc/configure_via_anaconda.md).
3. Fork, download or clone the [project repository](https://github.com/udacity/RoboND-Rover-Project) and have a look at the `README`.
4. Experiment with the simulator and take some data ( explained in the Telemetry and Record Data lesson).
5. Run through the Jupyter notebook and fill in the `process_image()` function.
6. Run `drive_rover.py` and experiment with autonomous mapping (details in the More Decisions lesson).
7. Fill in the `perception_step()` and `decision_step()` functions and map the environment!

# 1 Notebook Analysis
## 1.1 Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

The first step is to do a perspective transformation as shown below:

![Perspective_Transformation_demo](image/Perspective_Transformation_demo.PNG)

After several experiments, *rgb_thresh=(160, 160, 160)* is selected to be a suitable threshold for selecting **navigable terrain**.

As for the **obstacles**, the equation is *obstacle = np.absolute(np.float32(navigable) - 1) * mask* where mask is the section of the image after perspective transformation as shown below:

![mask](image/mask.PNG)

As for the **rock**, since the rock appears yellow in the simulated environment, a *yellow_thresh = (110 < img[:,:,0]) & (img[:,:,0]< 220) & (110 < img[:,:,1]) & (img[:,:,1]< 220) & (0 < img[:,:,2]) & (img[:,:,2]< 50)* is used to filter out the rock samples.

The original input frame with its perspective transformation and results after the thresholding (navigable_terrain, obstacles and rock) are shown below (indicated by white pixels):

![Combined](image/Combined.jpg)
![threshold_res](image/threshold_res.PNG)

After getting the navigable terrain, the forward direction of the rover can be determined.

![Getting_directions](image/Directions.PNG)

## 1.2 Populate the process_image() function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap. Run process_image() on your test data using the moviepy functions provided to create video output of your result.

Coordinates rotation, translation and scaling are used to transform the rover-centric map into world map as shown below. Details are shown in `perception.py`.

![To_world_1](image/To_world_1.PNG)
![To_world_1](image/To_world_2.PNG)

As for the mapping out video, please refer to `test_mapping.mp4`. One sample frame of the output mapping video is shown below. The bule, red and yellow indicate navigable terrain, obstacles and rock accordingly.

![Sample_output](image/Sample_output.png)

# 2 Autonomous Navigation and Mapping
## 2.1 Fill in the perception_step() (at the bottom of the perception.py script) and decision_step() (in decision.py) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

### Two modifications are introduced in perception_step():
1. Optimizing Map Fidelity: Your perspective transform is technically only valid when roll and pitch angles are near zero. If you're slamming on the brakes or turning hard they can depart significantly from zero, and your transformed image will no longer be a valid map. Setting thresholds near zero in roll and pitch to determine which transformed images are valid for mapping. i.e. Adding one conditional statement : *if abs(Rover.pitch) < 1 and abs(Rover.roll) < 1:* To avoiding mapping when the rover is pitching or rolling heavily which can help to optimize map fidelity.
2. Optimizing for Finding All Rocks Tip: The rocks always appear near the walls. Think about making your rover a "wall crawler" that explores the environment by always keeping a wall on its left or right. If done correctly, this optimization can help all the aforementioned metrics. i.e. Adding *Rover.nav_angles += 0.3*. But to remember reduce this 0.3 in the `decision.py` when determining the rover's forward direction.

### As for decision_step():

