## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

[//]: # (Image References)

[image1]: ./misc_images/exercise_1_outliers.JPG
[image2]: ./misc_images/exercise_1_inliers.JPG
[image3]: ./misc_images/exercise_2_objects.JPG
[image4]: ./misc_images/exercise_2_cluster.JPG
[image5]: ./misc_images/exercise_3_capture.JPG
[image6]: ./misc_images/exercise_3_train_graph.JPG
[image7]: ./misc_images/exercise_3_object_recognition.JPG
[image8]: ./misc_images/train_svm_graph.JPG
[image9]: ./misc_images/recognition_rviz_1.JPG
[image10]: ./misc_images/recognition_rviz_2.JPG
[image11]: ./misc_images/recognition_rviz_3.JPG

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.
Implemented filtering and RANSAC plane fitting to obtain the table and items as seen in the **Figure** **1** and **2** below. I played with the leaf size, the passthrough max and min axes values, as well as the RANSAC max_distance and applied the outlier removal. All iterated efforts until a better outcome was achieved.

![alt text][image1]
###### **Figure**  **1** : Outliers / Objects

![alt text][image2]
###### **Figure**  **2** : Inliers / Tabletop

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  
Implemented the segmentation code. Here I played to some extent again with the filtering and RANSAC values from Exercise 1 to obtain an even better result. 

The following were the values that worked for the segmentation and clustering of the objects:

    ec.set_ClusterTolerance(0.06)
    ec.set_MinClusterSize(100)
    ec.set_MaxClusterSize(1200)

The following **Figure** **3** and **4** show the RVIZ simulation displaying the objects and the clustered objects, respectively.

![alt text][image3]
###### **Figure**  **3** : Plain Objects

![alt text][image4]
###### **Figure**  **4** : Clustered Objects


#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
Here, implementing the code applied for Exercises 1 & 2, adding color and normals histograms on the cluster that was obtained on Exercise 2, along with some training (train_svm.py), object recognition was achieved.

For the capture_features.py (see example in **Figure** **5**) code I used a 15 iteration to obtain better training_set.sav data which would improve the training Confusion Matrixes outcome by the train_svm.py code (see training results in **Figure** **6**). 

![alt text][image5]
###### **Figure**  **5** : Run capture_features in progress

![alt text][image6]
###### **Figure**  **6** : Training Confusion Matrix Results

This process produced accurate model.sav data which led to an overall successful object recognition result (see **Figure**  **7**); although you'll notice an object drifting away for some reason.

![alt text][image7]
###### **Figure**  **7** : Exercise 3: Object Recognition Achieved


### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

For the project, I reset the values for the filtering, RANSAC, and segmentation, given that the values from the previous exercises were not on par with the new setup. After recalibrating those values, I proceeded on capturing the features (capture_features.py) and training the model (see train_svm.py results in **Figure**  **8**).

![alt text][image8]
###### **Figure**  **8** : Project Confusion Matrix Results

Due the project requiring to recognize more objects for pick_list_3, I decided to perform the capture_features.py job using a 50 iterations loop. This made the training_set.sav data to be more accurate and producing better model.sav data for object recognition efforts. 

The following images display the 3 object recognition tasks with the 3 different pick_list yaml files.

![alt text][image9]
###### **Figure**  **9** : Object Recognition with pick_list_1.yaml

![alt text][image10]
###### **Figure**  **10** : Object Recognition with pick_list_2.yaml

![alt text][image11]
###### **Figure**  **11** : Object Recognition with pick_list_3.yaml


Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  

Most of the effort was already performed for the Perception Exercises, therefore; aside from testing out new filter, RANSAC, and segmentation values; the output files were the main concern. To obtain the pick_pose I followed the instructions from the [Output Yaml Files] Lesson (https://classroom.udacity.com/nanodegrees/nd209/parts/586e8e81-fc68-4f71-9cab-98ccd4766cfe/modules/e5bfcfbd-3f7d-43fe-8248-0c65d910345a/lessons/e3e5fd8e-2f76-4169-a5bc-5a128d380155/concepts/f816b260-335a-40ec-883b-18ab27067205) by obtaining the object cloud centroid. 

For the place_pose and arm_name, I used the following code which is based on the group (color) from the object_list and then deciding which arm to use based on the dropbox where each object was supposed to be placed.

    # Used the groups color information to select which arm to use and which location to place the object
    if object_group == 'green':
        arm_name.data = 'right'
        place_pose.position.x = dropbox_param[1]['position'][0]
        place_pose.position.y = dropbox_param[1]['position'][1]
        place_pose.position.z = dropbox_param[1]['position'][2]
    else:
        arm_name.data = 'left'
        place_pose.position.x = dropbox_param[0]['position'][0]
        place_pose.position.y = dropbox_param[0]['position'][1]
        place_pose.position.z = dropbox_param[0]['position'][2]

One of the troubles I came across, was that the model was mistakenly recognizing the colored edges of the boxes as objects, therefore, I decided to use another passthrough filter to narrow it's vision. 

    passthrough = cloud_filtered.make_passthrough_filter()

    filter_axis = 'y'
    passthrough.set_filter_field_name(filter_axis)
    axis_min = -0.5
    axis_max = 0.5
    passthrough.set_filter_limits(axis_min,axis_max)
    cloud_filtered = passthrough.filter()
	
Given more time, I would pursue a full pick_place_routine by using techniques learned from the Kinematics Lessons. This is a very interesting topic and I'm hoping to look further into other possible techniques within computer vision and more complex object recognition.







