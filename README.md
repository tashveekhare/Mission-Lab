# Estimating the speed of the ISS using image feature matching

This project was developed for the Astro Pi Mission Space Lab competition and achieved **Flight Status**, meaning that it was selected to be run aboard the **International Space Station (ISS)**


**How it works:**
- Capture 5 consecutive images using the Astro Pi camera on the ISS
- Read the capture time from each image's EXIF metadata
- Detect ORB key points and descriptors
- Match features between consecutive images
- calculate the average pixel displacement
- Convert pixel displacement into ground distance using the camera's ground sample distance
- Estimate the ISS's speed using the distance time equation

**Technologies**
- Python
- OpenCV
- NumPy
- Astro Pi/ Raspberry Pi
- ORB feature detection and matching
- EXIF image metadata

**Computer vision techniques**
- detecting ORB key points
- extracting ORB descriptors
- converting to grayscale
- Gaussian blurring

**Hardware**

The program was designed to run on the Astro Pi computer aboard the International Space Station, using its camera to capture consecutive images of Earth's surface.

**Results**

The project successfully achieved Flight Status in the Astro Pi Mission Space Lab competition and was selected for execution aboard the ISS. The captured images were used to estimate the displacement of features on Earth's surface and, from this, calculate the ISS's speed.

**Astro Pi Mission Space Lab Competition**

The aim of the competition was to estimate the speed of the International Space Station using computer vision and image analysis
