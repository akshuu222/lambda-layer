```md
# AWS Lambda Node.js Layer – CI/CD Pipeline

This project demonstrates how to **automatically build and package an AWS Lambda Layer for Node.js applications** using **AWS CodeBuild** and **AWS CodePipeline**.

The goal is to generate a versioned **Lambda Layer ZIP artifact** and store it in the pipeline’s S3 artifact bucket for further deployment or publishing.

---

## Overview

A **Lambda Layer** allows you to share common dependencies across multiple Lambda functions.  
Instead of bundling `node_modules` with every function, you package them once as a reusable layer.

This repository focuses on:

- Automated Layer Packaging
- Node.js Dependency Installation
- CI/CD Artifact Generation
- Versioned ZIP Creation
- Pipeline-Managed S3 Storage

---

## CI/CD Flow

### 1. Source Trigger
Code is pushed to the repository.

### 2. CodePipeline Trigger
Pipeline starts automatically.

### 3. CodeBuild Stage
- Installs Node.js runtime
- Creates Lambda layer folder structure
- Installs dependencies
- Packages ZIP file
- Exports artifact to Pipeline

### 4. Artifact Storage
The generated ZIP is uploaded to the **CodePipeline S3 Artifact Bucket** automatically.

---

## Lambda Layer Folder Structure

AWS Lambda requires a specific directory layout:

```

nodejs/
node24/
node_modules/

````

Your build process dynamically creates this structure.

---

## buildspec.yml Explanation

### Runtime Installation

```yaml
install:
  runtime-versions:
    nodejs: 24.13.1
````

Ensures the build environment uses the required Node.js version.

---

### Build Phase Steps

#### Step 1 – Create Folder Structure

```bash
mkdir -p nodejs/node24
```

Creates Lambda-compatible folder hierarchy.

---

#### Step 2 – Move Dependency Files

```bash
mv package-lock.json package.json ./nodejs/node24
```

Moves dependency manifests into layer directory.

---

#### Step 3 – Install Dependencies

```bash
npm install
```

Installs `node_modules` inside `nodejs/node24`.

---

#### Step 4 – Clean Up

```bash
rm package-lock.json package.json
```

Removes unnecessary files from final layer package.

---

#### Step 5 – Zip Layer

```bash
zip -rq layer-${CODEBUILD_BUILD_NUMBER}.zip nodejs
```

Creates a versioned ZIP artifact using the build number.

---

### Artifacts Section

```yaml
artifacts:
  files:
    - layer-${CODEBUILD_BUILD_NUMBER}.zip
```

This tells CodeBuild to export the ZIP file to CodePipeline.

---

## Artifact Storage Behavior

When run inside CodePipeline:

* CodeBuild **does not upload to its own S3 bucket**
* CodePipeline automatically stores artifacts in its **managed S3 artifact bucket**
* Artifacts are versioned and traceable per pipeline execution

---

## Benefits of This Approach

* Reusable dependencies across Lambda functions
* Reduced deployment package size
* Faster Lambda cold starts
* Automated versioning
* No manual zipping required
* Centralized artifact management

---

## Versioning Strategy

Each build produces:

```
layer-<build-number>.zip
```

Example:

```
layer-15.zip
layer-16.zip
layer-17.zip
```

This allows easy rollback and traceability.

---

## Deployment Options

After artifact creation, you can:

* Publish Layer Version manually
* Automate publishing using CLI or SDK
* Add another pipeline stage for deployment
* Share layer across multiple Lambda functions

---



