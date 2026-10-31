# Fair Organ Allocation System

A full-stack academic prototype for managing organ donors, patients, and allocation records. The application combines a React frontend, a Java Spring Boot backend, and a separate Python Flask image-scoring service.

> **Important medical-safety note:** This project is an educational prototype, not a clinical decision-support system. The current Python service creates a DenseNet-121 model without loading trained weights, so its returned score is not a validated medical severity measurement. Do not use it to make real organ-allocation or treatment decisions.

## Overview

The system provides a web interface for registering donors and patients, submitting a patient image for experimental scoring, requesting a donor-to-patient match, and viewing saved allocation records.

## Main features

- **Donor registration** – records a donor's name, blood group, and organ type.
- **Patient registration** – records a patient's name, blood group, required organ, waiting days, and an uploaded image.
- **Allocation workflow** – filters patients by matching blood group and organ type, then ranks eligible patients by the stored score and waiting time.
- **Allocation history** – displays saved allocation records.
- **Help chatbot** – provides rule-based guidance on using the application.
- **Separate services** – keeps the React UI, Spring Boot API, and Flask image-scoring endpoint in separate folders.

## Technology stack

| Layer | Technologies |
|---|---|
| Frontend | React, React Router, Axios, CSS |
| Backend | Java 17, Spring Boot, Spring Data JPA, Maven |
| Database | H2 (configured in the current application properties) |
| Image-scoring service | Python, Flask, PyTorch, Torchvision, Pillow |
| Containerization | Docker |

## Architecture

```text
React frontend
     |
     | HTTP requests
     v
Spring Boot REST API --------> H2 database
     |
     | image upload request
     v
Python Flask image-scoring service
```

### Repository structure

```text
FairOrganAllocation/
├── frontend/                 # React application
│   └── src/components/       # Donor, patient, allocation, list and chatbot UI
├── organ-allocation-system/  # Spring Boot REST API and persistence
│   ├── src/main/java/        # Controllers, entities, repositories and services
│   ├── src/main/resources/   # Application configuration
│   └── Dockerfile
└── organ-aiService/          # Flask image-scoring API
    ├── app.py
    └── requirements.txt
```

## How allocation currently works

The backend's allocation service currently:

1. Finds patients whose blood-group string exactly matches the donor's blood group.
2. Keeps patients whose required organ matches the donor's organ type.
3. Selects the eligible patient with the highest stored image score; waiting days are used as the tie-breaker.
4. Saves an allocation-history record when a patient is selected.

This is a simplified demonstration rule. It is **not** a complete medical compatibility standard or a clinically validated allocation policy.

## Run locally

You will need Java 17, Maven (or the included Maven wrapper), Node.js/npm, and Python installed.

### 1. Start the Spring Boot backend

Open a terminal:

```bash
cd organ-allocation-system
./mvnw spring-boot:run
```

On Windows Command Prompt:

```bat
mvnw.cmd spring-boot:run
```

The backend uses the H2 in-memory database configuration currently committed in `src/main/resources/application.properties`. Data in an in-memory database is temporary and is cleared when the application stops.

### 2. Start the Flask image-scoring service

Open another terminal:

```bash
cd organ-aiService
python -m venv .venv
```

Activate the environment, then install and run:

Windows:
```bat
.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

macOS/Linux:
```bash
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

The Flask app reads the `PORT` environment variable and otherwise uses port `5000`.

**Model limitation:** The current service initializes DenseNet-121 with `pretrained=False` and does not load a trained checkpoint. Its output is therefore an untrained model score, not a meaningful prediction. A trained, evaluated model and suitable validation are required before describing this as a working medical AI model.

### 3. Start the React frontend

Open another terminal:

```bash
cd frontend
npm install
npm start
```

The development server normally opens at `http://localhost:3000`.

### Service configuration

The frontend and Java service currently contain deployed service URLs in their source code. For a local setup or a different deployment, update those API base URLs to point to your local backend and Flask service. For a production-quality setup, move service URLs into environment-based configuration rather than hard-coding them.

## Current scope and future improvements

- Load a properly trained model checkpoint and evaluate it on a held-out dataset.
- Replace the experimental image score with a clearly defined, validated scoring approach; do not interpret softmax confidence as clinical severity.
- Implement clinically reviewed compatibility and allocation rules before any real-world use.
- Add input validation, authentication, authorization, and audit logging.
- Move from the in-memory H2 database to a persistent database for deployment.
- Add automated backend and frontend tests.
- Configure service URLs and secrets through environment variables.

## Development and attribution

This repository is a personal fork of the original project:

- **Upstream repository:** [narigelabhargavi/FairOrganAllocation](https://github.com/narigelabhargavi/FairOrganAllocation)
- **Maintainer of this fork:** [Malreddy-Manogna](https://github.com/Malreddy-Manogna)

This project originated as a team project. The commit history in this fork records changes pushed from the maintainer's GitHub account. Project work and individual responsibilities should be described accurately when presenting the project.

## Disclaimer

This software is for learning and demonstration only. It is not intended for diagnosis, treatment, organ matching in clinical practice, or any other real patient-care decision.
