# Intelligent Document Processing (IDP) System

An end-to-end document AI application for extracting structured data from PDFs, images, and DOCX files. The system combines document classification, OCR and digital-PDF text extraction, schema-based field mapping, validation, a FastAPI backend, and a React human-review interface.

The project is designed around a practical IDP workflow:

```text
Upload document
  -> Detect file type
  -> Extract text / run OCR
  -> Classify document
  -> Map structured fields
  -> Validate extracted values
  -> Human review and correction
  -> Save reviewed JSON
```

## Key Capabilities

- Content-based file type detection
- Digital PDF extraction with `pypdf`, `PyMuPDF`, `pdfminer`, and `pdfplumber`
- OCR for scanned PDFs and images using multiple OCR engines
- DOCX extraction with `python-docx`
- Schema-based document classification and field extraction
- Structured extraction for employee forms, Aadhaar cards, PAN cards, bank passbooks, and invoices
- Post-extraction validation for required fields and common data formats
- FastAPI backend for document processing
- React review interface for human-in-the-loop correction
- Reviewed JSON output
- Batch evaluation with ground-truth comparison
- Accuracy and processing-time reports exported to Excel

## Architecture

```text
                         +-------------------+
                         |   React Frontend  |
                         +---------+---------+
                                   |
                                   v
                         +-------------------+
                         |   FastAPI API     |
                         +---------+---------+
                                   |
                                   v
                  +----------------+----------------+
                  | Document Processing Pipeline    |
                  +----------------+----------------+
                                   |
             +---------------------+---------------------+
             |                     |                     |
             v                     v                     v
      File Detection        Text / OCR Layer      DOCX Extraction
                                  |
                                  v
                       Document Classification
                                  |
                                  v
                         Schema Field Mapping
                                  |
                                  v
                              Validation
                                  |
                                  v
                         Human Review + JSON
```

## Supported Inputs

- Digital `.pdf`
- Scanned `.pdf` (requires Poppler for image conversion)
- `.png`
- `.jpg` / `.jpeg`
- `.docx`

## Supported Document Types

### Employee Enrollment Forms
Extracts fields such as employee name, employee ID, date of birth, date of joining, department, designation, mobile number, email, address, and pincode.

### Aadhaar Cards
Supports front- and back-side fields including Aadhaar number, VID, name, date/year of birth, gender, address, pincode, and relationship fields.

### PAN Cards
Extracts PAN number, names, father's name, date of birth, signature presence, and card issue text.

### Bank Passbooks
Extracts banking information such as bank name, branch details, IFSC, MICR, account holder, account number, PAN, address, account opening date, and issue date.

### Invoices
Extracts structured fields such as company, date, address, receipt number, subtotal, tax, discount, total, and currency.

## Extraction Strategy

The application selects an extraction path based on the uploaded document:

- **Digital PDFs:** tries `pypdf`, `PyMuPDF`, `pdfminer`, and `pdfplumber`
- **Scanned PDFs:** converts pages to images and sends them through the OCR pipeline
- **Images:** uses available OCR engines according to the model policy
- **DOCX:** reads paragraphs and table cells directly

OCR engines available in the project include:

- docTR
- PaddleOCR
- EasyOCR
- Tesseract
- RapidOCR

The parser also handles common OCR/layout issues such as split labels, values appearing on the next line, separator-only fields, and common OCR email errors.

## Human-in-the-Loop Review

The React interface provides a review workbench before data is saved:

```text
Upload -> Extract -> Review/Edit -> Save Reviewed JSON
```

Uploaded originals are temporary. After extraction, the uploaded file is removed and reviewed JSON is saved only when the user explicitly chooses to save it.

## Validation Layer

Extracted data passes through post-processing and validation checks including:

- Required fields
- Email format
- Mobile number format
- Pincode format
- Date validity
- Employee ID format

Missing values remain empty and are surfaced through validation errors or warnings instead of being replaced with invented values.

## Model Evaluation

The project includes a batch-testing framework for comparing OCR and PDF extraction approaches against matching ground truth.

Evaluation covers:

- Per-document accuracy
- Field-level accuracy
- Processing time
- Model failures
- Complete-record rate
- Image-quality conditions such as blur, skew, crop, low light, overexposure, and low resolution

Scanned PDF batches compare OCR engines, while digital PDF batches compare native PDF text extractors without converting pages to images.

Reports are saved as JSON and Excel files under `testing/test-outputs/`.

## Project Structure

```text
intelligent-document-processing-system/
├── Backend/
│   ├── api.py
│   ├── pipeline.py
│   ├── model_policy.py
│   ├── output.py
│   ├── processors/
│   └── extractors/
├── frontend/
│   └── src/
├── testing/
│   ├── testing.py
│   ├── reporting.py
│   ├── test-CLI/
│   ├── test-data/
│   └── test-outputs/
├── uploads/
├── outputs/
├── main.py
├── pyproject.toml
├── uv.lock
└── README.md
```

## Tech Stack

**Backend & APIs**
- Python 3.12+
- FastAPI
- Uvicorn

**Frontend**
- React
- Vite

**Document Processing**
- pypdf
- PyMuPDF
- pdfminer
- pdfplumber
- python-docx
- pdf2image
- OpenCV

**OCR**
- docTR
- PaddleOCR
- EasyOCR
- Tesseract
- RapidOCR
- ONNX Runtime

**Testing & Reporting**
- Python unittest
- Excel reporting
- Ground-truth batch evaluation

## Setup

Clone the repository:

```bash
git clone https://github.com/avdhanda44/intelligent-document-processing-system.git
cd intelligent-document-processing-system
```

Install Python dependencies with `uv`:

```bash
uv sync
```

### Run the FastAPI backend

```bash
uv --cache-dir .uv-cache run uvicorn Backend.api:app --reload --host 127.0.0.1 --port 8000
```

### Run the React frontend

In another terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at:

```text
http://127.0.0.1:5173
```

and proxies `/api/*` requests to the FastAPI backend at `http://127.0.0.1:8000`.

## Run Evaluation Batches

Examples:

```bash
uv --cache-dir .uv-cache run python testing/testing.py --aadhaar-batch
uv --cache-dir .uv-cache run python testing/testing.py --pan-batch
uv --cache-dir .uv-cache run python testing/testing.py --passbook-batch
uv --cache-dir .uv-cache run python testing/testing.py --invoice-batch
```

Digital PDF evaluation examples:

```bash
uv --cache-dir .uv-cache run python testing/testing.py --aadhaar-digital-pdf-batch
uv --cache-dir .uv-cache run python testing/testing.py --pan-digital-pdf-batch
uv --cache-dir .uv-cache run python testing/testing.py --passbook-digital-pdf-batch
```

Run the test suite:

```bash
uv --cache-dir .uv-cache run python -m unittest discover -s testing/test-CLI
```

## Current Limitations / Roadmap

Planned improvements include:

- Confidence scores for extracted fields
- Excel input extraction
- Database-backed persistence
- Expanded document schemas
- Deployment and production observability

## Why This Project

Real document-processing systems must handle more than clean OCR. They need to work across file formats, choose appropriate extraction methods, convert noisy text into structured fields, validate results, support human correction, and measure extraction quality.

This project brings those pieces together in one practical Intelligent Document Processing workflow.
