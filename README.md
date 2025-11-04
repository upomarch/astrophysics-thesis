# **Celestial Choreography: Turbulence’s Influence on Dust Clouds in the Heart of NGC 628**

### **Overview**

This project analyzes galactic images from the James Webb Space Telescope (JWST) to study the distribution of gas and dust clouds in spiral galaxies. These clouds play a key role in star formatio, collapsing under gravity and forming stars as turbulence, radiation, and motion within the galaxy shape their structure.

The goal is to measure how complex these clouds are by calculating their fractal dimension, a mathematical measure that quantifies how detailed or “clumpy” a structure appears at different scales. By examining how the number of visible cloud features changes as the image is blurred, we can estimate how matter is distributed and infer the physical processes driving that structure.

A complete PDF of the thesis is available in this repository under Unda_March_Thesis. Don't worry, it's mostly charts and there are some very cool photos!

---

## **Data Description**

Each dataset consists of:

* A **2D grayscale image array** (in `.fits` format), where each pixel represents light intensity captured by JWST.
* Metadata including image scale, wavelength, and spatial calibration.
* Output tables containing the **number of clouds** detected at each blur scale and the corresponding **fractal dimension**.

All images are preprocessed to ensure consistent units and aligned pixel grids before analysis.

---

## **Scientific Context**

### **Why Fractal Dimension Matters**

Astrophysical structures often display *self-similarity* — meaning their shapes look similar at multiple scales, from small filaments to large spiral arms. The **fractal dimension (D)** quantifies this property:

* **D ≈ 1**: Linear, filament-like structures (e.g., thin gas filaments).
* **D ≈ 2**: Plane-filling, complex regions (e.g., dense molecular clouds).

A higher fractal dimension implies more chaotic and turbulent environments, often linked to **active star formation**. Measuring D allows astronomers to connect visible structure in images to underlying physical processes like turbulence, gravity, and radiation feedback.

---

## **Methods Overview**

The code performs the following major steps:

1. **Load and prepare galactic image data.**
   Each image is read into a NumPy array using Astropy and normalized for consistent brightness scaling.

2. **Smooth the image using Gaussian convolution.**
   This creates progressively blurred versions of the image to simulate viewing the galaxy at different levels of resolution.

3. **Identify and count cloud structures.**
   Using threshold-based segmentation from `skimage`, the program counts how many separate bright regions (“clouds”) are visible at each blur level.

4. **Calculate fractal dimension.**
   The relationship between blur scale (r) and the number of detected clouds (N) follows:
   [
   N(r) \propto r^{-D}
   ]
   Taking logarithms gives a linear relation:
   [
   \log(N) = -D \log(r) + c
   ]
   The slope of this line, obtained via linear regression (`sklearn.linear_model`), gives the fractal dimension D.

---

## **Technical Exmaples**

### **1. Image Smoothing Using Astropy**

Below is a simplified excerpt showing how the code visualizes multiple levels of Gaussian smoothing:

```python
fig, ax = plt.subplots(nrows=2, ncols=2, figsize=(8, 8))
plt.subplots_adjust(left=0.05, bottom=0.05, right=0.95, top=0.95,
                    wspace=0.3, hspace=0.3)
ax = ax.flatten()
for axis in ax:
    axis.axis('off')

# Original image
im = ax[0].imshow(img, vmin=20, vmax=40.0, origin='lower',
                  interpolation='nearest', cmap='gray')
y, x = np.where(np.isnan(img))
ax[0].plot(x, y, 'rx', markersize=4)
ax[0].set_title("Input Data")

# Gaussian-blurred images
im = ax[1].imshow(astropy_conv8, vmin=20, vmax=40.0, origin='lower',
                  interpolation='nearest', cmap='gray')
ax[1].set_title("G8")

im = ax[2].imshow(astropy_conv10, vmin=20, vmax=40.0, origin='lower',
                  interpolation='nearest', cmap='gray')
ax[2].set_title("G10")

im = ax[3].imshow(astropy_conv24, vmin=20, vmax=40.0, origin='lower',
                  interpolation='nearest', cmap='gray')
ax[3].set_title("G24")

plt.tight_layout()
plt.show()
```

### **Definitions**

* **Gaussian convolution:** Each `G#` (G8, G10, G24) refers to the width of the Gaussian kernel used to blur the image.
  A *small kernel (G8)* keeps fine details, while a *large kernel (G24)* smooths over them to show broader structures.
* **Purpose:** This lets us study how visible “clouds” merge or disappear as we zoom out or reduce resolution.
* **NaN handling:** The code marks invalid data points (e.g., gaps in telescope imaging) with red X’s so they’re excluded from analysis.

Astropy’s convolution tools are used here because they handle NaN values gracefully — essential for real astronomical data where parts of the image may be missing or saturated.

<img width="790" height="592" alt="image" src="https://github.com/user-attachments/assets/513b3c2e-d448-440d-8611-88f0d64d1744" />

---

### **2. Cloud Counting**

After smoothing, each image is thresholded to isolate bright regions (dense clouds):

```python
from skimage.measure import label
from skimage.filters import threshold_otsu

threshold_value = threshold_otsu(image)
binary_mask = image > threshold_value
labeled_clouds, count = label(binary_mask, return_num=True)
```

* **`threshold_otsu`** automatically selects a brightness cutoff separating cloud regions from background noise.
* **`label`** assigns a unique identifier to each contiguous bright region.
* The total number of labels = the number of distinct “clouds.”

This is repeated for each blur level to observe how the number of clouds changes with scale.

---

### **3. Fractal Dimension Estimation**

For each blur scale ( r ) and corresponding number of clouds ( N ), the code stores:

```python
scales = np.array([8, 10, 24])
counts = np.array([N8, N10, N24])
```

Then it performs a **log–log regression**:

```python
from sklearn.linear_model import LinearRegression

X = np.log(scales).reshape(-1, 1)
y = np.log(counts)
model = LinearRegression().fit(X, y)
fractal_dimension = -model.coef_[0]
```

This slope (`-model.coef_[0]`) represents the **fractal dimension D**, summarizing how the galaxy’s structure scales with resolution.

---

### **4. Interpretation of Results**

* A **steeper slope** (larger D) means the number of clouds decreases rapidly with smoothing — indicating a **more complex, fragmented** structure.
* A **shallower slope** (smaller D) means the structure remains relatively intact — **smoother or more cohesive** gas regions.

Researchers can compare D values across galaxies or regions within the same image to study differences in star-forming activity.

<img width="1253" height="1570" alt="newfft" src="https://github.com/user-attachments/assets/69a12a7a-efee-4ff6-a0c9-789d831bce18" />


---

## **Software Dependencies**

* **Astropy** – image I/O and convolution operations
* **NumPy / SciPy** – numerical array handling and log scaling
* **Scikit-image** – segmentation and labeling of cloud regions
* **Scikit-learn** – regression for fractal dimension fitting
* **Matplotlib** – visualization of processed images and regression plots

All analysis was performed in a Jupyter notebook using Python 3.10.

---

## **Summary**

This workflow transforms raw telescope images into quantitative descriptors of galactic structure.
By:

1. Blurring images to simulate different viewing scales,
2. Counting how clouds merge or fragment, and
3. Calculating the fractal dimension from this relationship,

the code captures the underlying complexity and organization of matter in NGC 628. 


