# Convolutional Neural Networks (CNNs)

Convolutional Neural Networks (CNNs) are a class of deep neural networks primarily used for processing structured grid data, such as images. CNNs consist of three main components:

1. **Convolutional Layers** (Starts with)
2. **Pooling Layers**
3. **Fully Connected Layer** (Ends with)

## 1. Convolutional Layers

The convolutional layer is the foundational building block of a CNN. It involves:

### Components:
- **Input Data**: The raw data to be processed (e.g., an image matrix).
- **A Filter**: A small matrix (e.g., 3x3) used to extract features from the input.
- **A Feature Map**: The output of the convolution operation.

### Operation:
- A **dot product** is performed between the **Input Data** and the **Filter**, sliding over the input to generate the **Feature Map**.

### Key Factors:
- **Weights of the Filter**:
  - Initially random but adjusted during the training process using backpropagation and gradient descent.
- **Number of Filters**:
  - Determines the number of features extracted in a single convolutional layer.
- **Stride**:
  - Defines the number of pixels the filter moves during each step.
- **Zero Padding**:
  - Used when the filter size does not perfectly match the input dimensions. Common types include:
    - **Valid Padding**: No padding (results in reduced output size).
    - **Same Padding**: Pads the input so that the output size is the same as the input.
    - **Full Padding**: Adds padding to include every possible position of the filter.

### Non-Linearity:
- A **ReLU (Rectified Linear Unit)** is applied after the convolution to introduce non-linearity, ensuring the network can learn complex patterns.

---

## 2. Pooling Layer

The pooling layer is used to reduce the spatial dimensions of the feature maps, preserving important features while discarding less relevant details.

### Types of Pooling:
- **Max Pooling**: Retains the maximum value from a selected window.
- **Average Pooling**: Computes the average of values within the window.

### Benefits:
- **Reduces Complexity**: Smaller dimensions result in fewer computations.
- **Improves Efficiency**: Speeds up training and inference.
- **Limits Risk of Overfitting**: Reduces the chances of the model learning irrelevant patterns.

---

## 3. Fully Connected Layer

The fully connected layer connects the flattened output from previous layers to the final output, enabling task-specific predictions such as classification.

### Characteristics:
- **Task Classification**: Maps features to a specific label or output.
- **Softmax Activation Function**: Commonly used in the output layer for multi-class classification to generate probabilities for each class.

---

CNNs combine these components to efficiently process and classify structured data, making them a popular choice for computer vision tasks.

