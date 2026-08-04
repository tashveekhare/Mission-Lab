# Mission-Lab

from exif import Image
from datetime import datetime
import numpy as np
import cv2
import math
from sense_hat import SenseHat
import time
from picamzero import Camera

sense = SenseHat()
sense.clear()

# Distances in meters
horizontal_distance = 8000.0
radius = 6783700.0

def get_time(image):
    with open(image, 'rb') as image_file:
        img = Image(image_file)
        time_str = img.get("datetime_original")
        if time_str is None:
            print(f"Warning: No EXIF timestamp found in {image}. Using system time.")
            return datetime.now()
        time = datetime.strptime(time_str, '%Y:%m:%d %H:%M:%S')
    return time

def get_time_difference(image_1, image_2):
    time_1 = get_time(image_1)
    time_2 = get_time(image_2)
    return (time_2 - time_1).seconds

def convert_to_cv(image_path):
    return cv2.imread(image_path, 0)  # Convert to grayscale

def removing_noise(image):
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    equalised = cv2.equalizeHist(gray)
    blurred = cv2.GaussianBlur(equalised, (5, 5), 0)
    return blurred

def filter_keypoints_by_ground(keypoints, center, radius, tolerance=0.1):
    return [kp for kp in keypoints if radius * (1 - tolerance) <= np.sqrt((kp.pt[0] - center[0])**2 + (kp.pt[1] - center[1])**2) <= radius * (1 + tolerance)]

def calculate_features(image, feature_number=1000):
    orb = cv2.ORB_create(nfeatures=feature_number)
    return orb.detectAndCompute(image, None)

def calculate_matches(descriptors_1, descriptors_2):
    brute_force = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
    matches = brute_force.match(descriptors_1, descriptors_2)
    return sorted(matches, key=lambda x: x.distance)

def display_matches(image_1, keypoints_1, image_2, keypoints_2, matches):
    match_img = cv2.drawMatches(image_1, keypoints_1, image_2, keypoints_2, matches[:100], None)
    resize = cv2.resize(match_img, (1600, 600), interpolation=cv2.INTER_AREA)
    cv2.imshow('matches', resize)
    cv2.waitKey(1000)  
    cv2.destroyAllWindows()

def find_matching_coordinates(keypoints_1, keypoints_2, matches):
    return ([(keypoints_1[m.queryIdx].pt) for m in matches], [(keypoints_2[m.trainIdx].pt) for m in matches])

def calculate_mean_distance(coordinates_1, coordinates_2):
    distances = [math.hypot(c1[0] - c2[0], c1[1] - c2[1]) for c1, c2 in zip(coordinates_1, coordinates_2)]
    return sum(distances) / len(distances) if distances else 0

def calculate_speed_in_kmps(feature_distance, GSD, time_difference):
    return (feature_distance * GSD / 100000) / time_difference if time_difference else 0

# Capture images
cam = Camera()
images = [cam.take_photo(f'image{i:03d}.jpg') for i in range(5)]
time.sleep(5)

total_speed = 0

for i in range(4):
    image_1, image_2 = images[i], images[i+1]   
    time_difference = get_time_difference(image_1, image_2)
    image_1_cv, image_2_cv = convert_to_cv(image_1), convert_to_cv(image_2)

    keypoints_1, descriptors_1 = calculate_features(image_1_cv)
    keypoints_2, descriptors_2 = calculate_features(image_2_cv)
    
    if descriptors_1 is None or descriptors_2 is None:
        print(f"Skipping pair {i}: No descriptors found.")
        continue  # Skip this loop iteration


    image_height, image_width = image_1_cv.shape[:2]
    center = (image_width // 2, image_height // 2)
    radius = image_width // 2

    keypoints_1 = filter_keypoints_by_ground(keypoints_1, center, radius)
    keypoints_2 = filter_keypoints_by_ground(keypoints_2, center, radius)

    if not keypoints_1 or not keypoints_2:
        print(f"Skipping pair {i}: No keypoints after filtering.")
        continue
    matches = calculate_matches(descriptors_1, descriptors_2)
    display_matches(image_1_cv, keypoints_1, image_2_cv, keypoints_2, matches)

    coordinates_1, coordinates_2 = find_matching_coordinates(keypoints_1, keypoints_2, matches)

    avg_feature_distance = calculate_mean_distance(coordinates_1, coordinates_2)
    speed = calculate_speed_in_kmps(avg_feature_distance, 12648, time_difference)
    if time_difference == 0:
        print(f"Skipping pair {i}: Time difference is zero.")
        continue
    total_speed += speed

speed = total_speed / 4

with open("result.txt", "w") as file:
    file.write(str(speed))
file.close()
