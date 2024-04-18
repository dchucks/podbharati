+++
title = 'Master MNIST Image Classification with Ludwig'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

MNIST dataset, containing images of handwritten digits, is a cornerstone in the machine learning community for benchmarking image classification algorithms. Ludwig, a flexible tool created by Uber, makes deep learning simpler by allowing you to train a model without needing extensive coding. In this tutorial, we'll go through a complete setup of using Ludwig for MNIST image classification, from installation to visualization of results.

### Setting Up Your Environment

#### Install Ludwig
First, ensure your environment is ready to run Ludwig. This tutorial is designed for Google Colab, but you can adapt it for any Python environment. Begin by uninstalling previous TensorFlow versions and installing Ludwig directly from the repository:

```python
!pip uninstall -y tensorflow
!python -m pip install git+https://github.com/ludwig-ai/ludwig.git --quiet
```

### Loading the Dataset

#### Import Required Libraries
After installation, import necessary libraries and set up your environment:

```python
from ludwig.datasets import mnist
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Configure pandas display options
pd.set_option('display.max_colwidth', 80)
np.random.seed(31)  # For reproducibility
```

#### Load MNIST Data
Use Ludwig’s built-in function to load the MNIST dataset:

```python
train_df, test_df, _ = mnist.load(split=True)
```

### Exploring the Data
It’s crucial to understand the data you’re working with. Here’s how you can peek into the datasets:

```python
print(f"Training Dataset Sample:\n{train_df.sample(n=5)}")
print(f"\nTest Dataset Sample:\n{test_df.sample(n=5)}")
```

### Visualizing the Data

#### Plot Sample Images
Visual feedback is vital in image classification tasks. Let’s plot some digit images from the dataset:

```python
def plotDigitGrid(X, y, idxs, y_hat=None):
    plt.figure(figsize=(14,12))
    plt.rcdefaults()

    for i in range(len(idxs)):
        plt.subplot(5, 6, i+1)
        plt.imshow(X[idxs[i]], cmap='Greys')
        title = f"Label: {y[idxs[i]]}"
        if y_hat is not None:
            title += f"  Pred: {y_hat[idxs[i]]}"
        plt.title(title)
        plt.axis('off')
    plt.subplots_adjust(hspace=0.5)
    plt.show()

# Sample and display images
sample_df = train_df.sample(n=30)
sample_df['image'] = sample_df['image_path'].apply(plt.imread)
plotDigitGrid(sample_df['image'], sample_df['label'], sample_df.index)
```

### Configuring the Ludwig Model

#### Define the Model Configuration
Set up the configuration dictionary for the model. This includes defining the type of input and output features:

```python
config = {
  'input_features': [{'name': 'image_path', 'type': 'image', 'encoder': 'stacked_cnn',
                      'conv_layers': [{'num_filters': 32, 'filter_size': 3, 'pool_size': 2, 'pool_stride': 2},
                                      {'num_filters': 64, 'filter_size': 3, 'pool_size': 2, 'pool_stride': 2, 'dropout': 0.4}],
                      'fc_layers': [{'output_size': 128, 'dropout': 0.4}],
                      'preprocessing': {'num_processes': 4}}],
  'output_features': [{'name': 'label', 'type': 'category'}],
  'trainer': {'epochs': 5}
}
```

#### Train the Model
Initialize and train your Ludwig model using the configuration:

```python
from ludwig.api import LudwigModel
import logging

model = LudwigModel(config, logging_level=logging.INFO)
train_stats, preprocessed_data, output_directory = model.train(dataset=train_df)
```

### Evaluating the Model

#### Perform Evaluation
Evaluate the model on the test dataset and visualize the performance:

```python
test_stats, predictions, output_directory = model.evaluate(test_df, collect_predictions=True, collect_overall_stats=True)
from ludwig.visualize import confusion_matrix, learning_curves

confusion_matrix([test_stats], model.training_set_metadata, 'label', top_n_classes=[5], model_names=[''], normalize=True)
learning_curves(train_stats, output_feature_name='label')
```

#### Predict and Display Test Samples
Finally, let’s use the model to predict test samples and visualize them:

```python
predictions, _ = model.predict(test_df)
predictions = pd.merge(test_df, predictions, left_index=True, right_index=True)
sample_test_df = predictions.sample(n=30)
sample

_test_df['image'] = sample_test_df['image_path'].apply(plt.imread)
plotDigitGrid(sample_test_df['image'], sample_test_df['label'], sample_test_df.index, y_hat=sample_test_df['label_predictions'])
```

### Conclusion
Congratulations! You’ve just completed a full cycle of training and evaluating an image classification model using Ludwig for the MNIST dataset. This tutorial covered everything from setting up Ludwig, loading and preparing data, configuring and training the model, to evaluating and visualizing the results. Dive deeper by adjusting the model's architecture or preprocessing steps to achieve better accuracy.

