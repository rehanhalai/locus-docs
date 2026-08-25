# 3. Video Processing & AI Analytics

Once we have successfully "carved" out the raw video data from the disk sectors, we face the next problem: playing it and analyzing it.

## The Video Container Problem
When we carve data out of a Dahua or Hikvision DVR, it is often wrapped in a weird proprietary format (like a `.dav` file). Standard web browsers (like Chrome or Safari) cannot play `.dav` files. They only understand universal formats like `.mp4`.

## PyAV & FFmpeg Remuxing
**FFmpeg / PyAV** is an extremely powerful open-source multimedia processing framework. 
In our backend system, after our custom parser strips the proprietary container wrapper (like `.dav`), we feed the clean video stream to PyAV/FFmpeg. PyAV remuxes the raw elementary stream into a standard, web-playable `.mp4` file (using zero-transcoding stream copy) so it can be viewed in our React dashboard.

## How Video Works (Frames)
A video isn't magic; it is just a fast sequence of pictures (called Frames), usually playing at 25 or 30 frames per second (fps).
To use AI for analysis, we need to feed it individual pictures, not a continuous video stream.
FFmpeg will extract specific frames (for example, taking 1 picture every second) from the video and hand them over to our AI model.

## AI & Computer Vision (YOLO)
**YOLO (You Only Look Once)** is a highly popular, open-source AI model designed for **Object Detection**.
Our Python backend will feed the extracted picture frames into YOLO. 
YOLO will draw bounding boxes around things it recognizes and output structured detection data: *"Person (98% confidence)", "Vehicle (85% confidence)"*. 

We take this AI output and save it in our local SQLite database (`forensics.db`) alongside the timestamp of that frame. 
Now, when an investigator types *"Find all vehicles"* in our React UI, our database instantly searches its records and points the user to the exact timestamp on the video where YOLO detected the vehicle!
