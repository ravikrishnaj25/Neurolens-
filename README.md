# NeuroLens

**Smartphone-based Retinal Imaging and AI-driven Prediction System for Early Stroke Risk Screening**

---


## 🔭 Project Overview

NeuroLens is a low-cost stroke-risk **screening** concept built around three signals:

1. **Retinal imaging** – a fundus image captured with a smartphone and a low-cost 3D-printed fundus camera attachment (or an existing fundus photograph).
2. **Retinal occlusion analysis** – an AI model that classifies a fundus image as normal or as a retinal vascular occlusion pattern (RAO, CRVO, BRVO).
3. **Vascular indicators** – an AI-estimated carotid intima-media thickness (CIMT) category derived from the fundus image, and an estimated pulse wave velocity (ePWV) computed from age and blood pressure.

The outputs are combined into a weighted **stroke risk score** (Low / Moderate / High) and presented through a web application intended for primary health centres (PHCs) and rural clinics.

---

## 🎥 Demo Video

**Unlisted YouTube demo video (15–20 minutes total):**

- [NeuroLens Demo](https://youtu.be/nJfN_ATpa88)
- [NeuroLens Project Overview](https://youtu.be/XsVhbIZbKus)

---

## 📽️ Project Presentation

**Presentation covering project details and outcomes:** [View the NeuroLens presentation](https://drive.google.com/file/d/1giKTM3io_IMnwrheeUGbPpEgVHnt-tXM/view?usp=drive_link)

## ❗ Problem Statement

As stated in the project documents (figures are cited from the project's research protocol and presentation and have not been independently re-verified for this submission):

- Stroke is a leading cause of disability in India, with an estimated 1.8 million cases/deaths annually (the project documents use both wordings).
- About 70% of cases are reported to occur in rural or low-resource settings with limited access to neurologists.
- Patients may travel more than 50 km to reach CT/MRI, delaying diagnosis and treatment beyond the narrow therapeutic window.
- The documents report that many districts lack stroke specialists, many rural hospitals lack basic stroke imaging, and most PHCs lack retinal screening tools.
- Diagnostic imaging costs are described as unaffordable for a large share of rural patients.

**Problem:** There is no affordable, portable, early stroke-risk screening option at the primary-care level in underserved regions.

---

## 🏥 Healthcare Use Case & Intended Users

**Use case:** Non-invasive, early **stroke-risk screening** at the point of primary care, to help identify people who may need further evaluation by a clinician.

**Intended users**

| User | Role |
|------|------|
| Health workers / nurses / technicians at PHCs and rural clinics | Capture fundus image, record age and blood pressure, run screening |
| Doctors | Review risk output, history and reports; decide on referral and care |

**Intended role of the system:** supportive risk stratification only. It does **not** diagnose stroke, does not replace CT/MRI or clinical examination, and all clinical decisions remain with qualified clinicians.

---

## 💡 Proposed Solution & Key Features

- Smartphone-compatible retinal image acquisition, including a custom 3D-printed fundus camera prototype.
- Fundus image input as an uploaded image, or extracted from a fundus video (frame selection, crop, enhancement – standalone tool designed for integration).
- AI inference for retinal occlusion class, CIMT category and a brain-stroke probability.
- ePWV calculation from age and blood pressure.
- Weighted multi-modal risk score with Low / Moderate / High categories.
- Clinic dashboard, patient registration with medical history, downloadable PDF report, patient history and follow-up comparison.

---

## 🧪 Implementation Status

Statuses are assigned conservatively from what was verified in the project source and from project documentation.

| Component |
|-----------|
| Backend inference code for eye-stroke, CIMT and brain-stroke models (`backend/models.py`) |
| Trained model checkpoint files bundled in repo (`brain_stroke.pth`, `cimt_model.pth`, `eye_stroke.h5`) |
| ePWV calculation and age-based risk scaling |
| Weighted stroke-risk score combining the four components |
| 3D-printed smartphone fundus camera | 🟡 Prototype shown in project presentation; imaging-quality validation not documented |
| Web frontend (React), clinic login, dashboard, patient registration |
| Cloud database / authentication (Supabase) |
| Fundus video → image extractor integrated in the web app |
| PDF report generation |
| Patient history & longitudinal comparison |
| Prospective clinical data collection at JIPMER |
| Fundus-image-only (Phase 2) model |
| Offline mode, multi-language and voice guidance |


---

### Models verified in the backend source

| Task | Checkpoint | Architecture | Input | Output used |
|------|-----------|-------------------------------|-------|-------------|
| Retinal occlusion ("eye stroke") | `eye_stroke.h5` | Keras CNN: 3 conv/batch-norm/max-pool stages, 256-unit dense layer, 4-way softmax | 128×128 RGB | Class: NORMAL / RAO / CRVO / BRVO |
| CIMT estimation | `cimt_model.pth` | Siamese `seresnext50_32x4d` with regression head (PyTorch) | Fundus image(s), resized | CIMT value → risk bucket |
| Brain-stroke indicator | `brain_stroke.pth` | EfficientNet-B0 with 2-class linear head (PyTorch) | 224×224 RGB, ImageNet normalization | Softmax probability of class index 1 |

### Component scaling used by the risk score

**Retinal occlusion class → score**

| Class | Score |
|-------|------:|
| NORMAL | 0.1 |
| BRVO | 0.5 |
| CRVO | 0.8 |
| RAO | 1.0 |

**CIMT → score** (thresholds in mm): below 0.6 → 0.2; 0.6–0.8 → 0.55; above 0.8 → 0.9.

**ePWV** is estimated from age and mean blood pressure (`MBP = DBP + 0.4 × (SBP − DBP)`) using a published polynomial ePWV equation (coefficients in `calculate_ePWV_scale`, `backend/models.py`). The estimate is mapped to an age-dependent scale:

| Age group | Low (0.2) | Medium (0.55) | High (0.9) |
|-----------|-----------|---------------|------------|
| < 40 | < 7 m/s | 7–8 m/s | > 8 m/s |
| 40–59 | < 9 m/s | 9–10 m/s | > 10 m/s |
| ≥ 60 | < 11 m/s | 11–12 m/s | > 12 m/s |


### Risk score

```
Stroke Risk Score = 0.22 × EyeStroke + 0.35 × CIMT + 0.10 × BrainStroke + 0.33 × ePWV
```

Risk categories: **Low 0–0.4 · Moderate 0.4–0.7 · High 0.7–1.0**.

---

## 🧰 Technical Stack

| Layer | Technology |
|-------|-----------|
| Frontend  | React.js |
| Visualisation  | Chart.js |
| Backend API  | FastAPI (Python) |
| ML frameworks | PyTorch (`.pth` models), TensorFlow/Keras (`.h5` model); XGBoost planned/unverified |
| Backbones used | SE-ResNeXt-50 (32×4d) · EfficientNet-B0 · custom CNN |
| Database  | Supabase (PostgreSQL) |
| Authentication  | Clinic email/password login |
| Deployment  | Vercel (frontend); backend host **[ADD]** |
| Model tracking  | Hugging Face |
| Security  | HTTPS; AES encryption (documented for data handling) |
| Hardware | 3D-printed (PLA) fundus camera, 20D double aspheric lens, PVC pipe, smartphone (≥12 MP camera, Android 8.0+ / iOS 12+), BP monitor |

---

## 🔄 Application Workflow

The intended end-to-end flow (design documentation):

1. **Landing page** – what NeuroLens is, why early detection matters, Login/Register/About/Contact.
2. **Login** – clinics/PHCs authenticate with email and password.
3. **Dashboard** – screening statistics, risk distribution, monthly counts, quick actions.
4. **Patient registration** – name, age, gender, phone, auto-generated patient ID, optional address, auto visit date, and five yes/no history questions (hypertension, diabetes, previous stroke/TIA, heart disease, smoking/alcohol).
5. **Fundus input** – either upload a fundus **video** (select best frame → crop/auto-crop → brightness, contrast, sharpness, saturation → final centred retina image) or upload a fundus **image**.
6. **Clinical inputs** – age, systolic BP, diastolic BP (feed the ePWV calculation).
7. **Analyse** – backend runs retinal occlusion, CIMT, brain-stroke and ePWV components and combines them.
8. **Results** – risk category, component results, final score, fundus image and risk gauge.
9. **PDF report** – patient details, date, image, BP/age, component results, final risk and recommendation.
10. **History & follow-up** – search patients, compare previous scans, track progression.

---

## 📊 Results & Evaluation

**No measured performance results are reported in this repository.**

- No accuracy, sensitivity, specificity, AUC-ROC, F1, MAE or held-out test results are available for the bundled models.
- "≥85% accuracy" in the project documents is a **target/expected outcome**, not a measured result.
- Impact figures on cost, speed and detection rate in the presentation are **projections/estimates**, not outcomes of a completed study. The documents give differing cost figures (e.g. ₹100 vs ₹100–150 vs ₹500–1,000 per screening), so no cost figure is claimed here.

---

## 🚧 Limitations & Future Scope

**Current limitations**

- Not clinically validated; no prospective or external validation completed.
- Training data and procedure for the bundled checkpoints are undocumented.
- The CIMT model is a Siamese (left/right) architecture, but the current inference code passes the **same uploaded image to both branches**; the `eye_side` argument is unused.
- The brain-stroke model's output class semantics are assumed (index 1 treated as stroke-positive).
- Risk-score weights are heuristic and differ between project documents.
- Retinal occlusion class scores are fixed lookup values, not calibrated probabilities.
- CIMT is predicted from the fundus image; the protocol plans ultrasound-measured CIMT for ground truth, which is not yet collected.
- Image quality from a handmade 3D-printed camera has not been quantified in the available documentation.
- Population bias is possible; Indian-population performance is unknown.

**Future scope (planned)**

- JIPMER-based prospective data collection and validation after ethics approval.
- Fundus-image-only (Phase 2) model removing the need for CIMT/ePWV inputs.
- Explainability (heatmaps), bias assessment by age, sex and skin tone.
- Offline operation, multi-language support and voice guidance.
- Integration with national digital health programmes.
- Patent / publication.

---

## 👥 Team

**Submitting team**

| Name | Role | Contribution |
|------|------|--------------|
| Nishitha R | **Team Leader** | Project coordination and planning; research protocol and ethics documentation;3D-printed fundus camera prototyping and image acquisition; dataset preparation and testing |
| Ravikrishna J | **AI/ML Developer** | Model integration and inference pipeline (eye-stroke, CIMT and brain-stroke models); risk-score and ePWV implementation in the backend |
| Thirupathi P | **Full-Stack Developer** | Web application (frontend and backend API); database and authentication; dashboard and report features |
| Sudharshan R | **Backend Developer** | Backend API development (FastAPI) and connecting it to the model files; Supabase database and authentication setup |

**Faculty:** Dr. Kayalvizhi M – Head, Centre of Excellence of Biomedical Research and Innovation; Research Professor, Department of Biomedical Engineering, Chennai Institute of Technology.

---

## 🏫 College, Collaboration & Institutional Support

- **College:** Chennai Institute of Technology, Sarathy Nagar, Kundrathur, Chennai – 600069, Tamil Nadu, India
- **Department:** Biomedical Engineering
- **Principal Investigator:** Dr. Kayalvizhi M, Chennai Institute of Technology
- **Collaborating institution:** JIPMER (Jawaharlal Institute of Postgraduate Medical Education and Research), Puducherry
  - Dr. Karthikeyan Mahalingam, Assistant Professor, Department of Ophthalmology, JIPMER (co-investigator)


---

### Clone

```bash
git clone https://github.com/ravikrishnaj25/Neurolens-.git
```

### Backend

```bash
cd backend
python -m venv venv
# Windows:  venv\Scripts\activate
# Linux/macOS:  source venv/bin/activate
pip install -r requirements.txt
```

### Frontend

```bash
cd [FRONTEND FOLDER]
npm install
npm run dev     # or: npm start
```

## 📁 Repository Structure

```
backend/
├── app.py
├── models.py
├── requirements.txt
├── runtime.txt
└── models/
    ├── brain_stroke.pth
    ├── cimt_model.pth
    └── eye_stroke.h5
```
---


## 🙏 Acknowledgements & References

- Dr. Kayalvizhi M, Chennai Institute of Technology, for guidance.
- Dr. Karthikeyan Mahalingam and JIPMER, for the planned clinical collaboration.
- Chennai Institute of Technology.

References:

- Degerli A. et al. (2025). *Advanced Assessment of Stroke in Retinal Fundus Imaging with Deep Multi-view Learning.* Scientific Reports, 15, 2198.
- Shurrab M. et al. (2025). *Multimodal Deep Learning for Stroke Prediction and Detection.* arXiv:2505.02677.
- Gu H. et al. (2025). *Estimated Pulse Wave Velocity and Stroke Among Middle-Aged and Older Populations.* J Am Heart Assoc, 14(3), e038376.
- Shashikant R. C. & Das P. J. (2024). *First ischemic stroke: role of ultrasonographic assessment of increased carotid intima-media thickness.* Eur J Cardiovasc Med, 14, 874–877.
- Carretero E. et al. (2025). *Retinal Ischemic Perivascular Lesions as Potential Biomarkers for Cardiovascular Disease.* J Clin Med, 14(11), 3837.
- Fallico M. et al. (2020). *Risk of acute stroke in patients with retinal artery occlusion: a systematic review and meta-analysis.* Eye, 34(4), 683–689.
- Zhang W. et al. (2025). *Association Between ePWV and Mortality in Acute Ischemic Stroke Patients.* Scientific Reports, 15, 7082.

---

## ⚕️ Healthcare Disclaimer

NeuroLens is an academic **research prototype** for stroke-risk **screening support**. It is **not** a certified medical device, has **not** been clinically validated, and has **not** received regulatory authorization. It **does not diagnose** stroke or any other condition and **must not** be used to make or delay treatment decisions. A "Low" result does not rule out stroke; a "High" result is not a diagnosis. Anyone with stroke symptoms (face drooping, arm weakness, speech difficulty, sudden vision loss) must seek **emergency medical care immediately**. All outputs must be interpreted by a qualified clinician in the context of clinical examination and standard stroke protocols.
