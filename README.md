Estimating the speed of the ISS using image feature matching
-
This project was developed for the Astro Pi Mission Space Lab competition and achieved Flight Status, meaning that it was selected to be run aboard the International Space Station (ISS)

How it works:
- Capture 5 consecutive images using the Astro pi camera on the ISS
- Read the capture time from each image's EXIF metadata
- Detect ORB key points and descriptors
- Match features between consecutive images
- calculate the average pixel displacement
- Convert pixel displacement into ground distance using the camera's ground sample distance
- Estimate the ISS's speed using the distance time equation
