# 3. Video Processing & AI Analytics

Once we have successfully "carved" out the raw video data from the disk sectors, we face the next problem: playing it and analyzing it.

## The Video Container Problem
When we carve data out of a Dahua or Hikvision DVR, it is often wrapped in a weird proprietary format (like a `.dav` file). Standard web browsers (like Chrome or Safari) cannot play `.dav` files. They only understand universal formats like `.mp4`.

## FFmpeg to the Rescue
**FFmpeg** is an extremely powerful, open-source tool that is the undisputed king of video processing. 
In our backend system, we will send the carved `.dav` file to FFmpeg. FFmpeg will decode the raw, proprietary video stream and convert it into a standard, web-playable `.mp4` file so it can be viewed in our React dashboard.

## How Video Works (Frames)
A video isn't magic; it is just a fast sequence of pictures (called Frames), usually playing at 25 or 30 frames per second (fps).
To use AI for analysis, we need to feed it individual pictures, not a continuous video stream.
FFmpeg will extract specific frames (for example, taking 1 picture every second) from the video and hand them over to our AI model.

## AI & Computer Vision (YOLO)
**YOLO (You Only Look Once)** is a highly popular, open-source AI model designed for **Object Detection**.
Our Python backend will feed the extracted picture frames into YOLO. 
YOLO will draw invisible bounding boxes around things it recognizes and output data like: *"Person (98% confidence)", "Red Car (85% confidence)"*. 

We take this AI text output and save it in our PostgreSQL database alongside the timestamp of that frame. 
Now, when an investigator types *"Find all red cars"* in our React Web UI, our database instantly searches its records and points the user to the exact timestamp on the video where YOLO saw the car!
