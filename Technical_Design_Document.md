TECHNICAL DESIGN DOCUMENT (TDD)
FV2 — DIABETIC FOOT ULCER DETECTION USING DEEP LEARNING
============================================================

1. DOCUMENT PURPOSE
-------------------
This document describes the visible implementation and design of the FV2
repository: https://github.com/LKCSE78/FV2

The application is a prototype that uses a Streamlit interface and a PyTorch
CNN to classify uploaded foot images into one of five DFU-related classes. It
also offers model-explanation visualization, prediction history, and report
generation.

2. SCOPE AND STATUS
-------------------
In scope:
- User interface and image upload.
- Image preprocessing and model inference.
- Five-class output and confidence display.
- Grad-CAM heatmap visualization.
- SQLite user and prediction-history storage.
- PDF report and CSV export.

Out of scope:
- Clinical diagnosis or treatment recommendations.
- Medical-device certification.
- A guarantee of model accuracy or generalization.
- Production-grade identity management or multi-user database operations.

This document describes the current repository implementation where visible.
Items identified as recommendations are not claims that those capabilities are
already implemented.

3. HIGH-LEVEL ARCHITECTURE
--------------------------
User
  |
  v
Streamlit Web Interface (app.py)
  |
  +--> Account / Login / Password Reset
  |       |
  |       +--> SQLite database (users.db)
  |       +--> Gmail SMTP for OTP (requires safe credential configuration)
  |
  +--> Upload foot image
          |
          v
      PIL image decoding and RGB conversion
          |
          v
      Resize to 224 x 224, tensor conversion,
      ImageNet normalization
          |
          v
      PyTorch SimpleCNN
          |
          +--> Softmax probabilities and predicted class
          |
          +--> Grad-CAM heatmap (optional)
          |
          +--> Display result / confidence / chart
          |
          +--> Save prediction history to SQLite
          |
          +--> Generate PDF report / export CSV

4. TECHNOLOGY STACK
-------------------
Language: Python
Web interface: Streamlit
Deep learning: PyTorch and torchvision
Image handling: Pillow, NumPy, OpenCV
Database: SQLite (Python sqlite3)
PDF generation: fpdf
Email delivery: smtplib with Gmail SMTP
Data export: Python CSV functionality

5. APPLICATION MODULES
----------------------

5.1 Presentation Module
The Streamlit application provides sign-up, login, password reset, image
upload, prediction display, confidence chart, optional Grad-CAM visualization,
history table, feedback/notes, and report/export controls.

5.2 Authentication Module
User records are stored in SQLite. The current implementation hashes passwords
using SHA-256. This is not the recommended approach for production password
storage because it lacks a password-specific salt and adaptive work factor.
Replace it with Argon2id or bcrypt and add appropriate rate limiting and
session protections before production use.

5.3 Image Input and Preprocessing
Accepted file types are JPG, JPEG, and PNG. The uploaded image is decoded with
Pillow and converted to RGB. The preprocessing pipeline resizes it to 224 x
224 pixels, converts it to a tensor, and normalizes channels using:
Mean = [0.485, 0.456, 0.406]
Std  = [0.229, 0.224, 0.225]

Preprocessing must remain consistent with the process used during training.

5.4 CNN Inference Module
The SimpleCNN model contains:
- Conv2D: 3 input channels to 16 output channels, 3 x 3 kernel, padding 1.
- ReLU activation and 2 x 2 max pooling.
- Conv2D: 16 input channels to 32 output channels, 3 x 3 kernel, padding 1.
- ReLU activation and 2 x 2 max pooling.
- Flatten: 32 x 56 x 56 = 100,352 features.
- Fully connected layer: 100,352 to 128.
- ReLU activation.
- Output layer: 128 to 5 logits.

The application loads model.pth with CPU mapping and switches the model to
evaluation mode. Softmax converts logits into class probabilities; argmax
selects the predicted class.

Output classes:
0. Stage 0 - Healthy
1. Stage 1 - Superficial
2. Stage 2 - Deep
3. Stage 3 - Infected
4. Stage 4 - Gangrene

The class names represent the application's output labels. They should not be
treated as medically verified staging without clinical validation.

5.5 Explainability Module
Grad-CAM uses the second convolutional layer (model.features[3]) to generate
a heatmap indicating image regions influential to the selected class. The
heatmap is resized and overlaid on the input image. A heatmap is an explanation
aid, not proof that the model's prediction is correct.

5.6 Persistence Module
SQLite creates users.db and maintains user and prediction-history tables.
Prediction history includes email, result, confidence, stage, and timestamp.
The current implementation stores metadata, not a complete audit-grade
clinical record. Protect the database and uploaded images as sensitive data.

5.7 Reporting Module
The app can generate a PDF report containing the user, predicted result,
confidence, stage, timestamp, and optionally the input image. Prediction
history can be exported as CSV.

5.8 Email / OTP Module
The password-reset workflow generates a six-digit OTP and attempts to send it
through Gmail SMTP. The current source has sender configuration embedded in
the code. Move credentials to environment variables or Streamlit secrets,
avoid logging secrets, and add OTP expiry, attempt limits, and secure reset
handling before deployment.

6. DATA FLOW
-------------
1. User signs in to the dashboard.
2. User uploads a foot image.
3. The app decodes and preprocesses the image.
4. PyTorch loads/uses the trained SimpleCNN checkpoint.
5. The model returns five logits.
6. Softmax produces class probabilities and argmax selects the predicted label.
7. The UI displays the class and confidence score.
8. If enabled, Grad-CAM creates an explanatory overlay.
9. The app writes prediction metadata to SQLite.
10. The user may generate a PDF or export history as CSV.

7. CONFIGURATION
----------------
Required model artifact:
- model.pth in the application working directory.
- It must match the SimpleCNN architecture and expected state_dict format.

Current environment variables:
- None are read by app.py at the time this document was prepared.

Recommended future configuration:
- DFU_SMTP_EMAIL: Gmail sender account.
- DFU_SMTP_APP_PASSWORD: Gmail app password.

These names are proposed for a future code change and are not active until
app.py reads them. Never commit real credentials. Prefer environment variables
or Streamlit secrets for deployment.

8. SETUP AND EXECUTION
----------------------
From a terminal:

  git clone https://github.com/LKCSE78/FV2.git
  cd FV2
  python -m venv .venv

Windows:
  .venv\Scripts\activate

macOS/Linux:
  source .venv/bin/activate

Install dependencies:
  python -m pip install --upgrade pip
  pip install streamlit torch torchvision pillow numpy opencv-python fpdf

Place the compatible model checkpoint at ./model.pth, then run:
  streamlit run app.py

9. DATA AND PRIVACY
------------------
The database may contain account identifiers and prediction history. Apply
least-privilege filesystem access, avoid committing users.db, and establish
retention and deletion practices. If this prototype is tested with real
patient information, obtain appropriate consent and follow applicable privacy,
security, and healthcare requirements.

10. SECURITY REVIEW AND RECOMMENDATIONS
---------------------------------------
Priority improvements before deployment:
- Replace SHA-256 password storage with Argon2id or bcrypt.
- Move SMTP credentials out of source code and rotate any exposed credential.
- Add OTP expiration, attempt limits, and abuse prevention.
- Add input validation and safe error handling for uploaded files.
- Avoid exposing personal data in logs or downloadable reports.
- Add authorization checks around history and report access.
- Pin dependency versions and scan dependencies for vulnerabilities.
- Add automated tests for preprocessing, model loading, prediction, and exports.
- Add database backups and a documented retention policy.
- Review whether the model checkpoint and training data can legally be shared.

11. MODEL VALIDATION
--------------------
Before clinical use, evaluate with an independently labelled dataset that is
representative of intended users and capture conditions. Report per-class
precision, recall, F1-score, confusion matrix, sensitivity, specificity, and
calibration where appropriate. Test for data leakage, class imbalance, device
variation, and distribution shift. Do not infer real-world performance from
confidence scores alone.

12. LIMITATIONS
---------------
- The CNN is relatively simple and uses a large flattened feature vector.
- The model expects 224 x 224 RGB input.
- The available repository files do not establish clinical validation.
- Performance depends on training data, checkpoint quality, and image capture.
- A Grad-CAM heatmap may be misleading and does not establish causality.
- SQLite and the current authentication logic are prototype-level.

13. TEST PLAN
-------------
- Start-up: app launches when dependencies and model are present.
- Missing model: app displays a useful error.
- Image validation: valid JPG/JPEG/PNG uploads are accepted.
- Preprocessing: output tensor has shape (1, 3, 224, 224).
- Inference: model returns five logits and a valid predicted class index.
- Probability: softmax scores are finite and sum approximately to 1.
- Grad-CAM: heatmap generation completes for a valid image.
- Persistence: prediction history is stored and retrieved for the correct user.
- Export: PDF report and CSV history can be generated.
- Security: authentication, OTP expiry/rate limiting, access control, and secret
  handling are tested before any production deployment.

14. FUTURE ENHANCEMENTS
----------------------
- Add a pinned requirements.txt and automated test suite.
- Improve authentication, configuration, and database access controls.
- Add model versioning and traceability in reports.
- Add monitoring for input quality and prediction drift.
- Evaluate stronger backbones only after establishing a sound baseline.
- Consider deployment packaging and a documented model/data governance process.

15. REFERENCES
--------------
Source repository:
https://github.com/LKCSE78/FV2

Existing repository documentation:
- architecture.md
- design.md

This document is an engineering summary based on the publicly visible source
files. Verify all implementation details against the current code before
deployment.
