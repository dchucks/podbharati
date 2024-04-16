+++
title = 'How to Build an AI NSFW Content Checker with Python'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

In today’s digital age, monitoring and filtering Not Safe For Work (NSFW) content is crucial for maintaining professional and age-appropriate environments. Whether you’re a developer looking to integrate content moderation into your app, or just curious about how AI can assist in distinguishing safe from unsafe content, building an AI NSFW content checker can be a fascinating project. This blog post will guide you through creating an AI NSFW content checker using Python, Streamlit, and OpenNSFW2. We'll break down each step of the code, making it accessible for both beginners and seasoned programmers.

Alright, let's dive into the nitty-gritty of building an AI NSFW content checker with Python!

### Setting the Stage

First, we need to understand the tools and libraries we're going to use:

- **Python:** The backbone of our project, known for its simplicity and power in data science and machine learning tasks.
- **Streamlit:** A fantastic library that lets us create web apps for our machine learning projects with ease.
- **OpenNSFW2:** A Python library built on neural networks to detect NSFW content in images.

Now, let’s dissect the code and see how these elements come together to form our NSFW content checker.

### The Code Breakdown

#### Preliminaries and Setup

```python
import locale
def getpreferredencoding(do_setlocale = True):
    return "UTF-8"
locale.getpreferredencoding = getpreferredencoding

!pip install --upgrade opennsfw2 streamlit
!npx install localtunnel
```

Here, we're setting the preferred encoding to UTF-8 to ensure that our app handles text encoding reliably. Then, we install the necessary Python packages: `opennsfw2` for NSFW content detection, and `streamlit` for web app creation. `localtunnel` is used to expose our local server to the internet, making our app accessible publicly.

#### Building the Web App

```python
%%writefile app.py

import streamlit as st
import opennsfw2 as n2
import requests
from PIL import Image
import io
```

We start by writing our app script `app.py` with `%%writefile`, a magic command in Jupyter notebooks. We import our required libraries: `streamlit` for the app, `opennsfw2` for NSFW detection, `requests` for handling web requests, `PIL` (Python Imaging Library) for image processing, and `io` for input/output operations.

#### Configuring the Streamlit Interface

```python
st.title("NSFW Probability Checker")
st.write("Upload an image to check its NSFW probability.")
```

We set up the title and description of our web app using Streamlit’s functions `st.title` and `st.write`.

#### The NSFW Prediction Function

```python
def predict_nsfw(image_path):
    return n2.predict_image(image_path)
```

This function uses `opennsfw2` to predict the NSFW probability of the given image. It’s straightforward, with `predict_image` doing the heavy lifting of analyzing the image.

#### Handling Image Upload

```python
uploaded_file = st.file_uploader("Choose an image...", type=["jpg", "jpeg", "png"])
if uploaded_file is not None:
    image_bytes = uploaded_file.getvalue()
    image = Image.open(io.BytesIO(image_bytes))
```

We use Streamlit’s `file_uploader` to allow users to upload an image. If an image is uploaded, it's read into memory, and we prepare it for analysis.

#### Displaying and Processing the Image

```python
temp_file_path = "temp_image.jpg"
with open(temp_file_path, "wb") as f:
    f.write(image_bytes)

st.image(image, caption='Uploaded Image.', use_column_width=True)
```

The uploaded image is saved temporarily and displayed on the web app. We’re getting everything ready for the magic to happen!

#### Predicting and Displaying NSFW Probability

```python
probability = predict_nsfw(temp_file_path)
probability_percentage = round(probability * 100, 2)

if probability_percentage < 20:
    color = "green"
elif 20 <= probability_percentage < 80:
    color = "orange"
else:
    color = "red"

st.markdown(f'<h2 style="color:{color};">NSFW Probability: {probability_percentage}%</h2>', unsafe_allow_html=True)
```

We call our `predict_nsfw` function and process the probability, displaying it in a color-coded manner: green for safe, orange for questionable, and red for likely NSFW.

#### Running the App and Exposing It Online

```python
!streamlit run /content/app.py &>/content/logs.txt & curl ipv4.icanhazip.com
!npx localtunnel --port 8501
```

Finally, we run the Streamlit app and use `localtunnel` to make our local web server publicly accessible. This lets us share our NSFW checker with the world.

### Conclusion

Voilà! You've just built an AI-powered NSFW content checker. This tool can be incredibly useful for content moderation, ensuring that digital spaces remain professional and age-appropriate. By understanding the code and technology behind it, you're now equipped to customize and integrate this functionality into larger projects or platforms.

We've walked through the installation of necessary libraries, crafting a Streamlit app, handling image uploads, and making the NSFW predictions. This project not only showcases the power of Python and machine learning but also highlights the ease with which you can create interactive web applications with Streamlit.

Embracing these technologies can open doors to innovative solutions for digital content moderation and beyond. So, go ahead, tinker with the code, tweak the parameters, and maybe even enhance the model's accuracy or user interface. The digital world is your oyster!

This blog post aimed to demystify the process of creating an AI NSFW content checker. Hopefully, it sparked your interest in AI and machine learning and showed you just how approachable these technologies can be. Happy coding and content checking!