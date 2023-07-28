# ComputerVision-Object-Tracking
Welcome to my Computer Vision project! This project was developed during summer of 2023 (May - August). The object_tracking.py file takes as an input a video file (.mp4 only) and uses a pre-trained detection model to identify cars and track them along the video. It assigns a unique ID to each identified car and destroys that ID once the car exits the frame of the video. 

## Pre-requisites
To successfully deploy the object tracking you fill need the following. 
* Python 3.8.2 or above. Download here: https://www.python.org/downloads/
* If you're using MacOS or Linux you will need Homebrew. Download here: https://brew.sh/ and follow instructions for isntallation (<5 minutes)
* You will also need OpenCV for the video handling and framing. Download using the following command:

 For Windows: ```pip install opencv-python```
 
 For MacOS/Linux:  ```pip3 install opencv-python```

## Import Information
As the object detection model exceeds Githubs upload size (>25MB) it has not been included in this repository. As a result if you download the repository and attempt to execute the object tracking, it will not work. To see the full project in action, please email me at alextsaha@gmail.com and I will happily SecureTransfer the object detection model for you to use. 

## Launching the tracker 
Using your preferred IDE (e.g. VisualStudio Code) run the object_tracking.py file.

This should launch a new window showing the following video with the cars being identified by the system and tracked with unique IDs. 
![image](https://github.com/alextsah/ComputerVision-Object-Tracking/assets/98911345/3d17473d-9506-4e96-8aa4-795dd30786df)

## Development Strategy 

### 1. Object Detection
For convenience, I have already written this part and you find everything in the object_detection.py file. For this tutorial, I used Yolo v4 with its own pre-trained model for object detection. To use it just a call in the main file:
```
from object_detection import ObjectDetection
# Initialize Object Detection
od = ObjectDetection()
while True:
    # Detect objects on frame
    (class_ids, scores, boxes) = od.detect(frame)
    for box in boxes:
        (x, y, w, h) = box
        cx = int((x + x + w) / 2)
        cy = int((y + y + h) / 2)
        center_points_cur_frame.append((cx, cy))
```
Here is the result in the video frame.

![image](https://github.com/alextsah/ComputerVision-Object-Tracking/assets/98911345/e6417412-6221-4edf-ad11-99397312b8ad)


### 2. Object Tracking 
By saving the position of the center point of each object, you can trace the previous position of the objects and predict what the immediate next will be. 

### 3. Finding the point and assigning the ID
We don’t need the history of all the tracking but only the last points so I initialize an array to keep track of the previous points and then we need to calculate the distance between the points to make sure they all belong to the same object. The closer the points are, the greater the probability that we are tracking the same object.

```
# Initialize count
count = 0
center_points_prev_frame = []
tracking_objects = {}
track_id = 0
    # Only at the beginning we compare previous and current frame
    if count <= 2:
        for pt in center_points_cur_frame:
            for pt2 in center_points_prev_frame:
                distance = math.hypot(pt2[0] - pt[0], pt2[1] - pt[1])
                if distance < 20:
                    tracking_objects[track_id] = pt
                    track_id += 1
```
As you can see from the portion of code above with the math.hypot() function the distance of the two points is calculated and if the distance is less than 20, an ID is associated with the position of the point.

### 4. Assigning univocal ID
In this part of the code, we have to make sure to compare the previous object with the current one and update the position of the ID. In this way, the same object remains with the same ID for its entire path. When the object is no longer recognized, it loses the ID.

```
    else:
        tracking_objects_copy = tracking_objects.copy()
        for object_id, pt2 in tracking_objects_copy.items():
            object_exists = False
            for pt in center_points_cur_frame_copy:
                distance = math.hypot(pt2[0] - pt[0], pt2[1] - pt[1])
                # Update IDs position
                if distance < 20:
                    tracking_objects[object_id] = pt
                    object_exists = True
                    continue
            # Remove IDs lost
            if not object_exists:
                tracking_objects.pop(object_id)
```
Now the tracking works quite well and as you can see from the image below, the white car has lost track because the object has not been identified anymore.

![image](https://github.com/alextsah/ComputerVision-Object-Tracking/assets/98911345/84256c3a-ecac-433a-a3e5-68f726a160bd)

### 5. Adding new IDs to new cars
If a new object is identified, the list of points must also be updated. So here are the changes of the previous code that allow you to delete the old and add the points of the new cars identified.
```
    else:
        tracking_objects_copy = tracking_objects.copy()
        center_points_cur_frame_copy = center_points_cur_frame.copy()
        for object_id, pt2 in tracking_objects_copy.items():
            object_exists = False
            for pt in center_points_cur_frame_copy:
                distance = math.hypot(pt2[0] - pt[0], pt2[1] - pt[1])
                # Update IDs position
                if distance < 20:
                    tracking_objects[object_id] = pt
                    object_exists = True
                    if pt in center_points_cur_frame:
                        center_points_cur_frame.remove(pt)
                    continue
            # Remove IDs lost
            if not object_exists:
                tracking_objects.pop(object_id)
        # Add new IDs found
        for pt in center_points_cur_frame:
            tracking_objects[track_id] = pt
            track_id += 1
```
As can be seen from the video, the id is kept until the object is recognized. In fact, in the central part of the video, it traces very well

## Further Information 
This project was developed under guidance from Professor Marwan Kannan at McGill University during the summer of 2023. For more information on further applications please reach out to me at alextsaha@gmail.com 
