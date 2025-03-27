import numpy as np
import cv2

# Replace these with your actual image and bbox data
image = cv2.imread("your_image.jpg")  # Load your image
height, width = image.shape[:2]
bboxes = np.random.rand(1, 20, 2) * [width, height]  # Example: Replace with your (n, 20, 2) bbox data

# Sine augmentation parameters (tuned to image size)
amplitude = height * 0.05  # 5% of image height as max shift (adjustable)
frequency = 2 * np.pi / width  # One full wave across image width (adjustable)

# Create mesh grid for image warping
x, y = np.meshgrid(np.arange(width), np.arange(height))
sine_offset = amplitude * np.sin(frequency * x)  # Vertical sine distortion

# Warp the image using remap for efficiency
map_x = x.astype(np.float32)  # x-coordinates unchanged
map_y = (y + sine_offset).astype(np.float32)  # y-coordinates shifted by sine
warped_image = cv2.remap(image, map_x, map_y, cv2.INTER_LINEAR)

# Warp the bounding box polygons
warped_bboxes = np.copy(bboxes)
for i in range(bboxes.shape[0]):  # For each bbox
    for j in range(bboxes.shape[1]):  # For each vertex
        x, y = bboxes[i, j]
        # Apply the same sine offset based on x-position
        new_y = y + amplitude * np.sin(frequency * x)
        new_x = x  # Optionally: new_x = x + amplitude * np.sin(frequency * y)
        warped_bboxes[i, j] = [np.clip(new_x, 0, width-1), np.clip(new_y, 0, height-1)]

# Visualize the result
def draw_polygons(img, polygons, filename):
    img_copy = img.copy()
    for poly in polygons:
        pts = poly.reshape((-1, 1, 2)).astype(np.int32)
        cv2.polylines(img_copy, [pts], isClosed=True, color=(0, 255, 0), thickness=2)
    cv2.imwrite(filename, img_copy)

# Save original and warped images with bboxes
draw_polygons(image, bboxes, "original_with_bbox.jpg")
draw_polygons(warped_image, warped_bboxes, "warped_with_bbox.jpg")

# Output for verification
print("Original bbox (first sample):\n", bboxes[0])
print("Warped bbox (first sample):\n", warped_bboxes[0])
