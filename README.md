### Step 1: Prepare the Dataset

Make sure you have your images and masks ready. The images should be in one directory and the masks in another.

```python
import os
import numpy as np
import cv2
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split

# Load images and masks
def load_data(image_dir, mask_dir):
    images = []
    masks = []
    for filename in os.listdir(image_dir):
        if filename.endswith('.png'):  # Assuming images are in PNG format
            img = cv2.imread(os.path.join(image_dir, filename), cv2.IMREAD_GRAYSCALE)
            mask = cv2.imread(os.path.join(mask_dir, filename), cv2.IMREAD_GRAYSCALE)
            images.append(img)
            masks.append(mask)
    return np.array(images), np.array(masks)

image_dir = 'path/to/images'
mask_dir = 'path/to/masks'
images, masks = load_data(image_dir, mask_dir)

# Normalize images
images = images.astype('float32') / 255.0
masks = masks.astype('float32') / 255.0

# Split dataset
X_train, X_val, y_train, y_val = train_test_split(images, masks, test_size=0.2, random_state=42)
```

### Step 2: Build the U-Net Model

```python
import tensorflow as tf
from tensorflow.keras import layers, models

def unet_model(input_size=(256, 256, 1)):
    inputs = layers.Input(input_size)
    
    # Encoder
    c1 = layers.Conv2D(64, (3, 3), activation='relu', padding='same')(inputs)
    c1 = layers.Conv2D(64, (3, 3), activation='relu', padding='same')(c1)
    p1 = layers.MaxPooling2D((2, 2))(c1)

    c2 = layers.Conv2D(128, (3, 3), activation='relu', padding='same')(p1)
    c2 = layers.Conv2D(128, (3, 3), activation='relu', padding='same')(c2)
    p2 = layers.MaxPooling2D((2, 2))(c2)

    c3 = layers.Conv2D(256, (3, 3), activation='relu', padding='same')(p2)
    c3 = layers.Conv2D(256, (3, 3), activation='relu', padding='same')(c3)
    p3 = layers.MaxPooling2D((2, 2))(c3)

    c4 = layers.Conv2D(512, (3, 3), activation='relu', padding='same')(p3)
    c4 = layers.Conv2D(512, (3, 3), activation='relu', padding='same')(c4)
    p4 = layers.MaxPooling2D((2, 2))(c4)

    # Bottleneck
    c5 = layers.Conv2D(1024, (3, 3), activation='relu', padding='same')(p4)
    c5 = layers.Conv2D(1024, (3, 3), activation='relu', padding='same')(c5)

    # Decoder
    u6 = layers.Conv2DTranspose(512, (2, 2), strides=(2, 2), padding='same')(c5)
    u6 = layers.concatenate([u6, c4])
    c6 = layers.Conv2D(512, (3, 3), activation='relu', padding='same')(u6)
    c6 = layers.Conv2D(512, (3, 3), activation='relu', padding='same')(c6)

    u7 = layers.Conv2DTranspose(256, (2, 2), strides=(2, 2), padding='same')(c6)
    u7 = layers.concatenate([u7, c3])
    c7 = layers.Conv2D(256, (3, 3), activation='relu', padding='same')(u7)
    c7 = layers.Conv2D(256, (3, 3), activation='relu', padding='same')(c7)

    u8 = layers.Conv2DTranspose(128, (2, 2), strides=(2, 2), padding='same')(c7)
    u8 = layers.concatenate([u8, c2])
    c8 = layers.Conv2D(128, (3, 3), activation='relu', padding='same')(u8)
    c8 = layers.Conv2D(128, (3, 3), activation='relu', padding='same')(c8)

    u9 = layers.Conv2DTranspose(64, (2, 2), strides=(2, 2), padding='same')(c8)
    u9 = layers.concatenate([u9, c1])
    c9 = layers.Conv2D(64, (3, 3), activation='relu', padding='same')(u9)
    c9 = layers.Conv2D(64, (3, 3), activation='relu', padding='same')(c9)

    outputs = layers.Conv2D(1, (1, 1), activation='sigmoid')(c9)

    model = models.Model(inputs=[inputs], outputs=[outputs])
    return model

model = unet_model()
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
```

### Step 3: Train the Model

```python
# Reshape data for training
X_train = X_train.reshape(-1, 256, 256, 1)
y_train = y_train.reshape(-1, 256, 256, 1)
X_val = X_val.reshape(-1, 256, 256, 1)
y_val = y_val.reshape(-1, 256, 256, 1)

# Train the model
history = model.fit(X_train, y_train, validation_data=(X_val, y_val), epochs=50, batch_size=16)
```

### Step 4: Evaluate the Model

```python
# Predict on a sample image
sample_image = X_val[0].reshape(1, 256, 256, 1)
predicted_mask = model.predict(sample_image)
predicted_mask = (predicted_mask > 0.5).astype(np.uint8)  # Binarize the output
```

### Step 5: Display the Results

```python
# Display the original image and the predicted mask
plt.figure(figsize=(12, 6))
plt.subplot(1, 2, 1)
plt.title('Original Image')
plt.imshow(X_val[0].reshape(256, 256), cmap='gray')

plt.subplot(1, 2, 2)
plt.title('Predicted Mask')
plt.imshow(predicted_mask.reshape(256, 256), cmap='gray')

plt.show()
```

### Notes:
- Ensure that the paths to your images and masks are correct.
- Adjust the input size of the U-Net model according to your image dimensions.
- You may need to install the required libraries if you haven't already:

```bash
pip install tensorflow opencv-python matplotlib
```

This code provides a complete pipeline for training a U-Net model for segmentation tasks and visualizing the results. Adjust the parameters and architecture as needed based on your specific dataset and requirements.