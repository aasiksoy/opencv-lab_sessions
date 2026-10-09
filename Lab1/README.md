# Computer Vision Setup Guide

Welcome! This is a quick-start guide to setting up your local Python development environment using a **Virtual Environment (`venv`)** in **Visual Studio Code (VS Code)**. 

By the end of this guide, you will be able to run Jupyter Notebooks (`.ipynb`) and execute code using **OpenCV**, **NumPy**, and **Matplotlib**.

---

## rerequisites

Before starting, ensure you have the following installed on your computer:
1. **Python 3.8+** – [Download Python](https://www.python.org/).
2. **Visual Studio Code** – [Download VS Code](https://code.visualstudio.com/).
3. **VS Code Extensions** – Open VS Code, click on the Extensions icon on the left sidebar, and install:
   * **Python** (by Microsoft)
   * **Jupyter** (by Microsoft)

---

## step-by-Step Setup

### Step 1: Open Your Project Folder
1. Create a new folder on your computer for your project (e.g., `opencv-lab_sessions`).
2. Open VS Code, go to `File > Open Folder...`, and select your new folder.

### Step 2: Create a Virtual Environment (`venv`)
A virtual environment keeps your project's libraries isolated so they don't interfere with other projects on your computer.

1. Open the built-in terminal in VS Code (`Terminal > New Terminal`).
2. Run the following command depending on your Operating System:

* **Windows:**
  ```bash
  python -m venv .venv
  ```
* **macOS / Linux:**
  ```bash
  python3 -m venv .venv
  ```

*A folder named `.venv` will appear in your project directory.*

### Step 3: Activate the Virtual Environment
You must activate the environment so that any libraries you install stay inside this project.

* **Windows (PowerShell):**
  ```powershell
  .venv\Scripts\Activate.ps1
  ```

* **macOS / Linux:**
  ```bash
  source .venv/bin/activate
  ```

 **How to know it worked:** You will see `(.venv)` appear at the very beginning of your terminal prompt line.

### Step 4: Install Required Packages
With your virtual environment activated, copy and paste the following command into your terminal to install **OpenCV, NumPy, Matplotlib**, and the background tools needed to run **Jupyter Notebooks**:

```bash
pip install --upgrade pip
pip install opencv-python numpy matplotlib ipykernel
```

* **`opencv-python`**: Computer vision library.
* **`numpy`**: Advanced math and multi-dimensional array operations.
* **`matplotlib`**: Data visualization and plotting graphs/images.
* **`ipykernel`**: The engine required to run Jupyter Notebook (`.ipynb`) files inside your virtual environment.

---

## Running a Jupyter Notebook (`.ipynb`)

1. In VS Code, create a new file (in your project folder) and name it `test_notebook.ipynb`.
2. Click **Select Kernel** in the top-right corner of the notebook window.
3. Choose **Python Environments...** and select the interpreter that points to your **`.venv`** folder (it is usually recommended with a star, do not use 'global').
4. Create a new code cell (by clicking on + code), paste the following snippet, and press `Shift + Enter` (or click the Play button) to test everything:

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt

print("NumPy Version:", np.__version__)
print("OpenCV Version:", cv2.__version__)

# Create a simple synthetic image (gradient) using NumPy
gradient_img = np.linspace(0, 255, 10000).reshape(100, 100).astype(np.uint8)

# Display the image using Matplotlib
plt.imshow(gradient_img, cmap='gray')
plt.title("Environment Setup Success!")
plt.axis('off')
plt.show()
```

If the code runs without errors and displays a gray gradient square, your development environment is officially ready to go! 




---

## Troubleshooting Common Issues

As a beginner, it is completely normal to run into a few bumps during your first setup. Here are the most common issues and how to fix them:

### 1. Error: "Execution of scripts is disabled on this system" (Windows)
* **The Problem:** Windows security blocks your terminal from running the activation script.
* **The Fix:** Before running the activate command, run this line in your terminal to temporarily bypass the restriction for this session:
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
  ```
  Then, try activating your `.venv` again.

### 2. Error: `ModuleNotFoundError: No module named 'cv2'` (or `numpy`/`matplotlib`)
* **The Problem:** Your Jupyter Notebook or terminal is running outside of your virtual environment.
* **The Fixes:**
  * **In your Terminal:** Look at the left side of your prompt. If you don't see `(.venv)`, you forgot to activate it. Run the activation command from **Step 3**.
  * **In your Notebook (`.ipynb`):** Look at the top-right corner of your file. If it says `Python 3.x...` instead of `(.venv) (Python 3.x...)`, click on it, select **Python Environments...**, and switch it to your `.venv`.

### 3. Missing Jupyter "Select Kernel" Option
* **The Problem:** VS Code doesn't know how to handle your notebook because the extension didn't load properly.
* **The Fix:** Press `Ctrl+Shift+P` (Windows) or `Cmd+Shift+P` (Mac) to open the VS Code Command Palette. Type `Developer: Reload Window` and hit Enter. This will safely restart VS Code and refresh your extensions.

### 4. Code runs forever or hangs indefinitely
* **The Problem:** The Jupyter background engine (`ipykernel`) crashed or got stuck.
* **The Fix:** Look at the top menu of your `.ipynb` file and click the **Restart** button (the curved arrow icon). This resets your variables and restarts the kernel without deleting your code.
