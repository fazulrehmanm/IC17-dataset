import numpy as np
import cv2

# Example image and polygon (replace with your own)
image = cv2.imread("your_image.jpg")  # Shape: (height, width, 3)
height, width = image.shape[:2]
polygons = np.random.rand(1, 20, 2) * [width, height]  # Example: 1 polygon, 20 points, scaled to image size

# Sine augmentation parameters
amplitude = 20  # Max pixel shift (adjustable)
frequency = 0.05  # Controls wave frequency (adjustable)

# Create mesh grid for image warping
x, y = np.meshgrid(np.arange(width), np.arange(height))
sine_offset = amplitude * np.sin(frequency * x)  # Vertical sine wave

# Warp the image
warped_image = np.zeros_like(image)
for i in range(height):
    for j in range(width):
        new_y = int(i + sine_offset[i, j])  # Apply sine to y-coordinate
        new_x = j  # Optionally: new_x = j + amplitude * np.sin(frequency * i)
        if 0 <= new_y < height and 0 <= new_x < width:
            warped_image[new_y, new_x] = image[i, j]

# Warp the polygons
warped_polygons = np.copy(polygons)
for i in range(polygons.shape[0]):  # For each polygon
    for j in range(polygons.shape[1]):  # For each vertex
        x, y = polygons[i, j]
        new_y = y + amplitude * np.sin(frequency * x)  # Same sine offset as image
        new_x = x  # Optionally: new_x = x + amplitude * np.sin(frequency * y)
        warped_polygons[i, j] = [np.clip(new_x, 0, width-1), np.clip(new_y, 0, height-1)]

# Visualize (optional)
cv2.imwrite("warped_image.jpg", warped_image)
print("Original polygon:\n", polygons[0])
print("Warped polygon:\n", warped_polygons[0])

# If you want to draw polygons
for poly in [polygons, warped_polygons]:
    img_copy = warped_image.copy()
    pts = poly[0].reshape((-1, 1, 2)).astype(np.int32)
    cv2.polylines(img_copy, [pts], isClosed=True, color=(0, 255, 0), thickness=2)
    cv2.imwrite(f"poly_image_{id(poly)}.jpg", img_copy)
