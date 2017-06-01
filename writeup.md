## Project: Search and Sample Return
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

I modified the existing thresholding function to take in both a lower and upper bound. It selects locations where all 3 rgb values lie within the specified ranges and sets the values at those positions to 1. The result is a binary image Where only the pixels within the allowed range are white. 

I used (160,160,160) as the lower bound to detect path pixels as suggested. For the obstacles, I used the same threshold as the upper bound because anthing that isn't part of the path is considered an obstacle. Then I took an example image of a rock and played around with the numbers until it was able to filter out just the rock. On the color wheel, yellow is furthest from blue, so I started with high red and green values and calibrated from there.

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

I first measured the source and destination points for a perspective transform. In order to measure the source, I used plt.imshow to display the example rover image and  manually recorded the points of a tile in the image. The destination is just the coordinates of a 10x10 square in the center of the bottom row. 

After taking the perspective transform, I applied my updated color_thresh function for each feature(path, rock and obstacle). I then converted the binary images to rover coordinates where the pixels locations are given relative to the rover being at (0,0). Finally I took into account the yaw and location of the rover to convert those rover coordinates onto the world map.

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

I was able to reuse most of the logic in the process_image function from the notebook for the perception step. The only changes are updating a new array, prev_pos, that keeps track of the prev positions of the rover. The new state variable is used to detect when the rover is stuck and needs to reset itself. Also I added a check to the pitch and roll to make sure not to record bad data since the perspective transform is most accurate at 0 for both.

I added logic in the decision function for picking up which just detects if its near an objective, turns on the brakes and sets the pickup flag. I also added a new state 'reset' which triggers when the std deviation prev_pos is less than a certain threshold. Basically if we're seeing little to no change in the positions, stop and turn for a set duration. I make sure to reset prev_pos afterwards and only trigger 'reset' mode when the array is full capacity so that it doesn't get stuck in reset.

I've updated the steering logic to take a wall hugging approach. I take the mean of the angles and also the mean of the bottom group of angles. Then my output angle is the weighted sum of these two values.
The weight is a dynamic weight based on range between the values. The greater the range, the more emphasis is put on the lower avg compared to the standard one. This is to account for the case where the rover has a big range of directions to choose from but has a lot path pixels to the left. In that case, I want to weaken the influence from the standard mean in order to have the rover still favor the right wall. 

I've also updated all the stop and reset turns to be to the left as most of the time, the right back wheel gets caught or we run directly into the wall when hugging the right side.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

The rover takes a right wall hugging approach when navigating the world. From my runs, it can map out 100% of the map with around 70% fidelity while picking up 2-3 samples by chance.

I would add a new weight on the steering based on the position of the rocks so that it would pick up nearby rocks while mapping.

I want to further extend this by adding a function that can take a coordinate and navigate to it while avoiding obstacles. That way I can find navigate back to the home point after collecting all the samples.

Sometimes, the rover will see 2 paths and will try to right in the middle causing it to get stuck on small obstacles. I would like to add a clustering algorithm to figure out if multiple paths are available and only run the steering calculations on the rightmost path. 

