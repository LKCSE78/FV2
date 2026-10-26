FV2 — DIABETIC FOOT ULCER (DFU) DETECTION
============================================================

Repository
----------
https://github.com/LKCSE78/FV2

OVERVIEW
--------
FV2 is a Streamlit-based prototype for classifying foot images into five
diabetic foot ulcer (DFU) stages using a PyTorch convolutional neural network
(CNN). It includes image upload, prediction confidence, a Grad-CAM heatmap,
SQLite-backed user/prediction history, and PDF/CSV export features.

IMPORTANT MEDICAL NOTICE
-----------------------
This project is a research/educational decision-support prototype. Predictions
are not a diagnosis and must not replace assessment by a qualified healthcare
professional. The model's reliability depends on its training data and
validation. Do not use it as the sole basis for treatment or urgent-care
decisions.

FEATURES
--------
- Streamlit web dashboard.
- Upload JPG, JPEG, or PNG foot images.
- Five output classes:
  Stage 0 - Healthy
  Stage 1 - Superficial
  Stage 2 - Deep
  Stage 3 - Infected
  Stage 4 - Gangrene
- CNN inference using PyTorch.
- Grad-CAM visualization for model interpretability.
- User sign-up/login and prediction history using SQLite.
- PDF report generation and CSV history export.
- Password-reset OTP workflow through Gmail SMTP (requires configuration).

REPOSITORY CONTENTS
-------------------
app.py                  Main Streamlit application.
architecture.md         CNN architecture documentation.
design.md               System design notes.
dfu_cnn_train.py        Model training script.
dfu_cnn_train_pytorch.py Training script variant.
dfu_cnn_train_pytorch1.py Training script variant.
dfu_report.pdf          Example/generated DFU report.
bg.jpg                  Image asset.
model.pth               Required trained model weights; provide this file
                        locally if it is not included in the repository.

REQUIREMENTS
------------
- Python 3.10 or a compatible Python version.
- pip.
- A compatible PyTorch and torchvision installation.
- A trained model checkpoint named model.pth that matches SimpleCNN in app.py.

The application imports Streamlit, PyTorch, torchvision, Pillow, NumPy,
OpenCV, and fpdf. SQLite, smtplib, hashlib, random, string, os, and datetime
are part of the Python standard library. Install package dependencies in the
steps below. Exact package versions are not pinned in the repository; for
reproducible deployment, validate and pin versions in a requirements file.

SETUP (WINDOWS CMD)
-------------------
1. Clone the repository:

   git clone https://github.com/LKCSE78/FV2.git
   cd FV2

2. Create a virtual environment:

   py -m venv .venv
   .venv\Scripts\activate

3. Upgrade pip:

   python -m pip install --upgrade pip

4. Install application dependencies:

   pip install streamlit torch torchvision pillow numpy opencv-python fpdf

   If PyTorch installation fails or you need a specific CPU/GPU build, follow
   the official installation selector at https://pytorch.org/get-started/locally/.

5. Add the trained model checkpoint:

   Place the compatible model weights file at:
   FV2\model.pth

   The application expects model.pth in the current working directory. If the
   checkpoint is missing or does not match the model architecture, prediction
   will not work.

6. Start the application:

   streamlit run app.py

7. Open the local URL printed by Streamlit in your browser (usually
   http://localhost:8501).

SETUP (macOS / LINUX)
---------------------
   git clone https://github.com/LKCSE78/FV2.git
   cd FV2
   python3 -m venv .venv
   source .venv/bin/activate
   python -m pip install --upgrade pip
   pip install streamlit torch torchvision pillow numpy opencv-python fpdf
   streamlit run app.py

ENVIRONMENT VARIABLES AND SECRETS
---------------------------------
The current app.py does not read configuration from environment variables.
The Gmail sender address and SMTP password are currently written directly
in the source code; this is not safe for a public repository or deployment.

For password-reset email to work, configure Gmail SMTP with an app password
and refactor the code to read secrets from environment variables or Streamlit
secrets. Suggested variable names for that future configuration are:

   DFU_SMTP_EMAIL=your-sender@gmail.com
   DFU_SMTP_APP_PASSWORD=your-gmail-app-password

These variables are suggestions only and will NOT work until app.py is updated
to read them. Never commit real credentials, API keys, or passwords. If a
credential has already been committed, revoke/rotate it.

MODEL AND IMAGE PROCESSING
--------------------------
The application resizes images to 224 x 224 pixels, converts them to tensors,
and applies ImageNet normalization. The CNN produces five class scores, which
are converted to probabilities with softmax. Grad-CAM creates a heatmap from
the second convolutional layer.

DATABASE
--------
SQLite creates users.db in the working directory and stores user and prediction
history. Keep this file private because it may contain personal data. Back it
up securely and do not commit it to a public repository.

USAGE
-----
1. Start the Streamlit application.
2. Create an account and log in.
3. Upload a JPG, JPEG, or PNG foot image.
4. Review the predicted stage, confidence score, and optional Grad-CAM heatmap.
5. Generate a PDF report or export prediction history as CSV.

TROUBLESHOOTING
---------------
- "model.pth not found": place the correct checkpoint in the project root.
- Model loading error: confirm that the checkpoint was trained with a model
  architecture compatible with SimpleCNN in app.py.
- Import error: activate the virtual environment and install the dependencies.
- OTP email failure: SMTP credentials are not configured through environment
  variables yet; refactor app.py and use a Gmail app password.
- OpenCV/PyTorch installation issues: install a compatible wheel for your
  Python version and operating system.

SECURITY AND LIMITATIONS
------------------------
- The current password hashing uses SHA-256 without a password-specific salt.
  For production, use a password-hashing algorithm such as Argon2id or bcrypt.
- The current code contains SMTP credentials/configuration in source and should
  be refactored before public deployment.
- SQLite is suitable for a prototype, but production deployments need stronger
  access controls, backups, and privacy safeguards.
- Validate model performance on representative, independently labelled data
  before any clinical use.

PROJECT DOCUMENTATION
---------------------
See architecture.md and design.md in the repository for existing model and
system-design notes. A more complete Technical Design Document is included
as Technical_Design_Document.txt.

LICENSE
-------
No open-source license was visible in the repository when this document was
prepared. Unless the repository owner adds a license, users should not assume
permission to copy, modify, redistribute, or use the code. See LICENSE.txt.
