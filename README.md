# Whisper Quickstart on OpenShift AI

Welcome to the whisper quickstart on OpenShift AI! This project helps you quickly set up the required components on Red Hat OpenShift AI and begin transcribing audio with minimal configuration.

To see how it's done, jump straight to [installation](#install). 


# Whisper Quickstart on OpenShift AI

This repository provides a Helm chart and supporting resources for a quick start with **speech-to-text transcription using Whisper** on **Red Hat OpenShift AI**.  
The deployment provisions all required components, including:

- A **Workbench** in OpenShift AI with preloaded example notebooks.  
- A deployed **Whisper-large-v3 model** served through a GPU-enabled runtime (`vLLM 0.9.2.2`).  
- Pre-cloned quickstart repository with data samples and example workflows.  

With this setup, users can start experimenting with **automatic audio transcription** using either:  
1. **Direct HTTP requests** to the deployed model.  
2. **The OpenAI Python client** interface.  

---

## Architecture Overview

The following diagram illustrates the components deployed by this Helm chart:
<p align="center">
  <img src="./assets/images/architecture-img.png" alt="Architecture" width="380"/>
</p>

- **Workbench**: Hosts Jupyter notebooks with example workflows and preloaded data samples.  
- **Whisper-large-v3 model**: Deployed through a GPU ServingRuntime (vLLM 0.9.2.2).  
- **Two alternatives to trigger transcription request**:  
  - **Option 1**: Send audio files via HTTP request.  
  - **Option 2**: Use the OpenAI Python client.  

---

## Requirements 

### Recommended hardware requirements 

- GPU required : +24GiB vRAM
- CPU cores: 16 cores   
- Memory: 64Gi+ RAM 
- Storage: 15Gi

### Minimum hardware requirements 

- GPU required : 1 x NVIDIA GPU with 24GiB vRAM
- CPU cores: 8+ cores 
- Memory: 32Gi+ RAM 
- Storage: 10Gi 

### Required software  

- Red Hat OpenShift Container Platform
- Red Hat OpenShift AI
    - KServe needs to be enabled
- Red Hat OpenShift Serverless
- Red Hat OpenShift Service Mesh 2
- Red Hat - Authorino Operator 
- Nvidia GPU Operator 
- Node Feature Discovery Operator 

### Required permissions

- Cluster admin permissions are required
---

## Install

**Please note before you start**

This example was tested on Red Hat OpenShift 4.19.9 & Red Hat OpenShift AI 2.22.1.  

### Clone the repository

```
git clone https://github.com/rh-ai-quickstart/basic-speech-to-text-with-whisper.git && cd basic-speech-to-text-with-whisper/
```

### Install with Helm
Deploy the Whisper Quickstart resources using Helm:

```bash
helm install whisper-quickstart whisper-quickstart-1.0.0.tgz \
  --namespace whisper-quickstart --create-namespace \
  --set namespaces="whisper-quickstart" \
  --set model.name="whisper-large-v3" \
  --set model.storageUri="oci://quay.io/redhat-ai-services/modelcar-catalog:whisper-large-v3"
```

This command will:  
- Create a namespace `whisper-quickstart`.  
- Deploy the **Workbench** and preconfigured **Whisper-large-v3 model**.  
- Clone the quickstart repository into the Workbench environment.  

---

## Verifying the Deployment

After installation, verify that the resources are running:

```bash
oc get all -n whisper-quickstart
```

Ensure that the following are present and ready:  
- **Deployment**: `whisper-large-v3-predictor`  
- **StatefulSet**: `whisper-workbench`  
- **Services**: including the predictor and Workbench endpoints  

![oc-get-all](./assets/images/oc-get-all.png)

---

## Accessing the Workbench

1. Log in to the **OpenShift AI dashboard**.  
2. Locate the **Workbench** named `whisper-workbench` under the `whisper-quickstart` data science project.  
3. Open the Workbench — the quickstart repository will already be cloned.  
4. Inside, you will find two example notebooks to start transcribing audio.  

![workbench](./assets/images/workbench.png)
![notebook-page](./assets/images/notebook-page.png)

---

## Finding the Model Endpoint

The deployed Whisper model can be accessed within the cluster.  
There are several ways to find the service endpoint:

### Option 1: Using OpenShift AI UI
- Navigate to **Models → Model deployments** in the OpenShift AI dashboard.  
- Find your deployed model and copy internal or external endpoint URL.  

![rhoai-endpoint](./assets/images/rhoai-endpoint.png)

### Option 2: Using the OpenShift Web Console
- Navigate to **Networking → Services → whisper-large-v3-predictor**.  
- Under **Service routing**, copy the **Hostname** (e.g., `whisper-large-v3-predictor.whisper-quickstart.svc.cluster.local`).  

![Service-Routing](./assets/images/serving-route.png)

### Option 3: Using the `oc` CLI
```bash
oc get svc whisper-large-v3-predictor -n whisper-quickstart \
  -o jsonpath='"http://{.metadata.name}.{.metadata.namespace}.svc.cluster.local:{.spec.ports[?(@.name=="http")].targetPort}{"\n"}"'
```

This will return a URL similar to:

```
http://whisper-large-v3-predictor.whisper-quickstart.svc.cluster.local:8080
```
![oc-get-svc](./assets/images/oc-get-svc.png)

---

## Running Transcription Examples

From the Workbench, open one of the provided notebooks:

- **HTTP request**: Demonstrates sending local audio files to the deployed service endpoint.  
- **OpenAI client**: Demonstrates using the OpenAI Python client library for transcription.  

Note: Both notebooks are preconfigured with demo endpoints — please update them before you run the cells.  

![example-transcribe](./assets/images/example-transcribe-1.png)
![example-transcribe](./assets/images/example-transcribe-2.png)

---

## Cleanup

To uninstall all deployed resources:

```bash
helm uninstall whisper-quickstart -n whisper-quickstart
oc delete namespace whisper-quickstart
```

---

