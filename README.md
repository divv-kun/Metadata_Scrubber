# SYSTEM://METADATA_SCRUBBER

> An automated, zero-trace privacy metadata remover tool powered by **n8n**, **ExifTool**, and **Docker**.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Docker](https://img.shields.io/badge/docker-automated-blue)
![n8n](https://img.shields.io/badge/n8n-workflow-orange)

---

## 📖 Overview

SYSTEM://METADATA_SCRUBBER is a privacy-focused backend service designed to strip sensitive metadata  
(EXIF, GPS, device details) from digital files **without compromising image quality**.

The system follows a **dual-mode architecture**:

1. **Scrubber Mode**  
   Removes metadata headers while preserving original pixel data (lossless).

2. **Analyzer Mode**  
   Extracts hidden metadata and returns it as a structured JSON report.

### Core Philosophy

- **Zero-Trace** – No database, no persistence, no logs
- **Stateless** – Each request is processed independently
- **Scalable** – Handles files up to **1 GB** (tested)

---

## 🏗️ Architecture

The system runs entirely inside a sandboxed **Docker container** based on **Alpine Linux**.

- **Ingestion** – Files received via HTTP Webhook (POST)
- **Processing** – Files staged temporarily in `/tmp`
- **Logic** – Bash scripts trigger ExifTool for analysis or scrubbing
- **Egress** – Clean file or JSON report returned to the user
- **Cleanup** – `/tmp` is wiped immediately after every transaction

> If it’s not stored, it can’t leak.

---

## 📂 Project Structure

```text
.
├── backend
│   ├── n8n-workflows
│   │   ├── analyse.json      # n8n workflow for metadata analysis
│   │   └── scrubber.json     # n8n workflow for metadata scrubbing
│   ├── docker-compose.yml    # Container orchestration
│   └── Dockerfile            # Custom Alpine image with ExifTool + n8n
├── Sample                    # Test files
│   ├── sample_image1.jpg
│   ├── sample_image2.jpg
│   ├── sample_image3.png
│   └── sample.pdf
└── index.html                # Simple frontend for testing
```

---

## ⚙️ Installation & Setup

### 1. Prerequisites

Ensure the following are installed:

- **Docker Desktop** (or Docker Engine + Docker Compose)
  [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- **Git**
  [https://git-scm.com/downloads](https://git-scm.com/downloads)

---

### 2. Clone the Repository

```bash
git clone https://github.com/divv-kun/Metadata_Scrubber.git
cd Metadata_Scrubber/backend
```

---

### 3. Start the Application

Build the custom image and start the services:

```bash
docker-compose up -d --build
```

- `--build` ensures Dockerfile changes are applied
- `-d` runs containers in detached mode

Once running, n8n will be available at:

```
http://localhost:5678
```

---

### 4. Initial n8n Setup (First Run)

1. Open `http://localhost:5678` in your browser
2. Create a local **Admin Account** (email & password)
3. Skip the usage survey if prompted

---

### 5. Import n8n Workflows

The analyzer and scrubber logic is provided as workflow JSON files.

1. Open the **n8n Dashboard**
2. Go to **Workflows** → **Add Workflow**
3. Select **Import from File**
4. Import:

   - `backend/n8n-workflows/scrubber.json`
   - `backend/n8n-workflows/analyse.json`

5. **Activate** each workflow using the toggle (top-right)

Webhook IDs will be generated automatically.

---

## 🔌 API Endpoints

### Metadata Analyzer

- **Endpoint:** `/webhook/analyze-file`
- **Method:** `POST`
- **Content-Type:** `multipart/form-data`
- **Field Name:** `data`

**Response:** JSON metadata extracted using ExifTool.

---

### Metadata Scrubber

- **Endpoint:** `/webhook/scrub-file`
- **Method:** `POST`
- **Content-Type:** `multipart/form-data`
- **Field Name:** `data`

**Response:** Cleaned file with metadata removed.

---

## 🧪 Testing the System

- Open `index.html` in a browser
- Upload files from the `Sample/` directory
- Monitor executions in the **n8n → Executions** tab
- Each upload should trigger an instant workflow execution

---

## 🔐 Privacy & Security Model

- Files exist only in `/tmp` during processing
- No database or persistent storage
- Automatic cleanup after every request
- Docker isolation limits attack surface

---

## 🎯 Use Cases

- Journalists protecting source location data
- OSINT and security professionals sanitizing shared files
- Photographers sharing images without metadata leakage
- Platforms integrating automatic metadata scrubbing

---

## ⚠️ Limitations

- Concurrency depends on container resources
- Very large files may increase processing time
- Supported formats depend on ExifTool compatibility
