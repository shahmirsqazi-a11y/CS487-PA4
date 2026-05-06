<div align="center">

# PA4 Submission: TaskFlow Pipeline

<img alt="GitHub only" src="https://img.shields.io/badge/Submit-GitHub%20URL%20Only-10b981?style=for-the-badge">
<img alt="Total points" src="https://img.shields.io/badge/Total-100%20points-7c3aed?style=for-the-badge">

</div>

## Student Information

| Field | Value |
|---|---|
| Name | Shahmir Sher Qazi |
| Roll Number | 26100204 |
| GitHub Repository URL | https://github.com/shahmirsqazi-a11y/CS487-PA4 |
| Resource Group | `rg-sp26-26100204` |
| Assigned Region | `swedencentral` |

## Evidence Rules

- Use relative image paths, for example: `![AKS nodes](docs/aks-nodes.png)`.
- Every image must have a 1-3 sentence description below it.
- Azure Portal screenshots must show the resource name and enough page context to identify the service.
- CLI screenshots must show the command and output.
- Mask secrets such as function keys, ACR passwords, and storage connection strings.


## Task 1: App Service Web App (15 points)

### Evidence 1.1: Forked Repository

![Forked Repository](task1.1.png)

Description: This is the working fork of the PA4 starter repository under the account shahmirsqazi-a11y. It contains the full PA4 starter structure including webapp/, function-app/, validate-api/, and report-job/ directories.

### Evidence 1.2: App Service Overview

![App Service Overview](task1.2.png)

Description: The Web App pa4-26100204 is deployed in resource group rg-sp26-26100204 in Sweden Central, running on Node 20 LTS with status Running. The public URL is pa4-26100204-bjfne5c0gpgse8dr.swedencentral-01.azurewebsites.net.

### Evidence 1.3: Deployment Center / GitHub Actions

![Deployment Center](task1.3.png)

Description: The Web App is connected to the GitHub fork via GitHub Actions. The workflow file main_pa4-26100204.yml automatically builds and deploys the webapp/ folder on every push to main.

### Evidence 1.4: Live Web UI

![Live Web UI](task1.4.png)

Description: The TaskFlow dashboard is successfully served by the App Service over HTTPS, displaying the order submission form and status panel.

---

## Task 2: Azure Container Registry (15 points)

### Evidence 2.1: ACR Overview

![ACR Overview](task2.1.png)

Description: The Azure Container Registry pa426100204 is provisioned in resource group rg-sp26-26100204 with Basic SKU and admin user enabled.

### Evidence 2.2: Docker Builds

![Docker Builds](task2.2.png)

Description: All three images were built successfully using `az acr build`. validate-api was built from ./validate-api, report-job from ./report-job, and func-app from ./function-app.

### Evidence 2.3: ACR Repositories

![ACR Repositories](task2.3.png)

Description: The ACR repository list confirms validate-api:v1, report-job:v1, and func-app:v1 were all pushed successfully to pa426100204.azurecr.io.

### Evidence 2.4: Local Validator Test

![Local Validator Test](task2.4.png)

Description: The validate-api container was tested locally on port 8080. A POST to /validate with a valid order returned `{"valid":true,"reason":"ok","order_id":"LOCAL-1"}`.

### Evidence 2.5: ACR Push Output

![ACR Push](task2.5.png)

Description: Successful push output for all three images to ACR showing layers pushed and digest confirmed.

### Evidence 2.6: ACR Portal View

![ACR Portal](task2.6.png)

Description: The Azure Portal shows the ACR resource pa426100204 with Succeeded provisioning state.

### Evidence 2.7: Image Tags

![Image Tags](task2.7.png)

Description: The ACR repositories view shows validate-api, report-job, and func-app each tagged with v1.

### Evidence 2.8: ACR Build Output

![ACR Build](task2.8.png)

Description: Output of `az acr build` showing the cloud build and push completing successfully for all three images.

---

## Task 3: Durable Function Implementation (12 points)

### Evidence 3.1: Completed Function Code

[function_app.py](function-app/function_app.py)

Description: The orchestrator chains two activities: validate_activity POSTs the order to the AKS validator and returns a valid/invalid result. If valid, report_activity uses the Azure SDK to create an ACI running the report-job image, polls until Succeeded, then returns the blob URL of the generated PDF.

### Evidence 3.2: Local Function Handler Listing

![func start](task2.8.png)

Description: Running `func start` locally shows all four Durable handlers registered: http_starter (HTTP POST), my_orchestrator (orchestrationTrigger), validate_activity (activityTrigger), and report_activity (activityTrigger).

---

## Task 4: Function App Container Deployment (8 points)

### Evidence 4.1: Function App Container Configuration

![Function App Container](task4.1.png)

Description: The Function App pa4-26100204-fn is configured to pull the container image pa426100204.azurecr.io/func-app:v1 from ACR using admin credentials.

### Evidence 4.2: Orchestration Smoke Test

![Smoke Test](task4.2.png)

Description: A curl POST to the HTTP starter endpoint returns an instance id and statusQueryGetUri, proving the Function App deployed successfully and the orchestrator started.

### Evidence 4.3: Expected Failed Status

![Expected Failure](task4.2.png)

Description: Polling the statusQueryGetUri shows runtimeStatus: Failed with an error about VALIDATE_URL not being reachable. This is expected at this stage since the AKS validator was not yet wired.

---

## Task 5: AKS Validator (15 points)

### Evidence 5.1: AKS Cluster

![AKS Cluster](task5.1.png)

Description: The AKS cluster pa4-26100204 is deployed in resource group rg-sp26-26100204 in Sweden Central with 1 node of size Standard_B2s and provisioningState Succeeded.

### Evidence 5.2: Kubernetes Nodes and Pods

![Nodes and Pods](task5.2.png)

Description: `kubectl get nodes` shows one node in Ready state. `kubectl get pods` shows validate-deployment pod in 1/1 Running state, confirming the validator is scheduled and healthy.

### Evidence 5.3: Kubernetes Service

![Kubernetes Service](task5.3.png)

Description: `kubectl get service validate-service` shows the LoadBalancer service with EXTERNAL-IP 4.166.10.86 exposed on port 8080, provisioned by Azure.

### Evidence 5.4: Validator API Tests

![Validator Tests](task5.4.png)

Description: GET /health returns `{"status":"ok"}`. POST /validate with qty=2 returns `{"valid":true}`. POST /validate with qty=999 returns `{"valid":false,"reason":"quantity exceeds limit"}`, demonstrating the qty > 100 rejection rule.

### Evidence 5.5: Function App VALIDATE_URL

![VALIDATE_URL](task5.5.png)

Description: The Function App environment variable VALIDATE_URL is set to http://4.166.10.86:8080/validate, allowing validate_activity to reach the AKS validator over HTTP.

### Evidence 5.6: AKS Idle Behavior

![AKS Idle](task5.5.png)

Description: The AKS node continues running even when no orders are being processed. Unlike ACI, AKS keeps the node alive at all times, incurring continuous compute cost while providing a stable always-on endpoint.

---

## Task 6: ACI Report Job (15 points)

### Evidence 6.1: Blob Container

TODO: Embed screenshot of the `reports` blob container.

Description: TODO

### Evidence 6.2: Manual ACI Run

TODO: Embed screenshot of `az container show` for `ci-report-test`.

Description: TODO

### Evidence 6.3: ACI Logs

TODO: Embed screenshot of `az container logs`.

Description: TODO

### Evidence 6.4: Generated PDF

TODO: Embed screenshot showing `TEST-001.pdf` in Blob Storage.

Description: TODO

### Evidence 6.5: Function App Managed Identity and IAM

TODO: Embed screenshots of managed identity attached to Function App.

Description: TODO

### Evidence 6.6: Report App Settings

TODO: Embed screenshot of REPORT_*, ACR_*, STORAGE_CONN, and SUBSCRIPTION_ID settings.

Description: TODO

---

## Task 7: End-to-End Pipeline (15 points)

### Evidence 7.1: Web App Wiring

TODO: Embed screenshot showing FUNCTION_START_URL and FUNCTION_STATUS_URL configured on the Web App.

Description: TODO

### Evidence 7.2: Happy Path UI

TODO: Embed screenshots of the form before submit, Running status, and Completed status.

Description: TODO

### Evidence 7.3: Backend Participation

TODO: Embed screenshots showing Function App invocation, AKS validator evidence, ACI evidence, and Blob PDF evidence.

Description: TODO

### Evidence 7.4: Reject Path UI

TODO: Embed screenshot of an order with qty > 100 being rejected.

Description: TODO

---

## Task 8: Write-up and Architecture Diagram (5 points)

### Evidence 8.1: Architecture Diagram

![Architecture Diagram](architecturediagram.png)

Description: The diagram shows all components: GitHub connecting to App Service via CI/CD, App Service calling the Durable Function, the orchestrator calling the AKS validator and spawning ACI per run, ACI writing PDFs to Blob Storage, and ACR serving images to all three compute services. The managed identity relationship between the Function App and resource group is also shown.

### Question 8.2: Service Selection

**App Service** is the right choice for the TaskFlow web frontend because it provides a managed, always-on PaaS environment with native GitHub Actions CI/CD integration, making it easy to deploy a Node.js Express app without managing infrastructure. It supports custom domains, HTTPS, and environment variables natively.

**Durable Functions** coordinate the multi-step async workflow because they provide stateful orchestration with automatic checkpointing between activities. If the report step fails after validation succeeds, the runtime can replay from the checkpoint without re-running validation — something plain HTTP functions cannot do.

**AKS** hosts the validator because it is a long-lived HTTP microservice that must respond to every order with low latency. AKS provides a stable LoadBalancer endpoint, declarative pod management, and production-grade container orchestration — appropriate for a service that runs continuously.

**ACI** runs the report generator because it is a short-lived batch job that starts, generates a PDF, uploads it, and exits. ACI bills only while the container is alive, making it cost-efficient for per-order workloads that run for ~20 seconds. Keeping this as a persistent AKS workload would waste compute.

### Question 8.3: ACI vs AKS

When the AKS cluster is idle for 10 minutes, the node continues running and billing at the Standard_B2s rate — there is no scale-to-zero for a standard AKS node pool. The validator pod stays scheduled and ready to serve requests instantly. For ACI in this pipeline, "idle" is meaningless — the container only exists during a run. The orchestrator creates it, it does its work, and report_activity deletes it. There is zero idle cost. If a malicious user spammed the Submit button 1000 times in a minute, ACI would incur the most cost because the Function App would attempt to spawn 1000 container instances, each billing from creation to deletion.

### Question 8.4: Durable Functions vs Plain HTTP

If the same flow were implemented as two plain HTTP functions calling each other, there would be at least two concrete problems. First, the report step takes up to 60 seconds, which would exceed the default HTTP timeout of many clients and intermediate proxies, causing the request to fail even if the work completed successfully. Second, there is no state persistence — if the second function crashes mid-execution, there is no way to know whether validation already succeeded, so the entire flow must restart from scratch. Durable Functions solve both problems: the orchestrator checkpoints after each activity, so replays skip already-completed steps, and the client polls a status URL asynchronously rather than waiting on a single long HTTP connection.

### Question 8.5: Cost Review

TODO: Embed Cost Management screenshot scoped to your resource group.

Description: TODO: Identify the most expensive resource and explain why.

### Question 8.6: Challenges Faced

**Challenge 1: Key-based storage authentication blocked by subscription policy.** The auto-created storage account for the Function App had `allowSharedKeyAccess` disabled by a subscription-level policy, causing the Function runtime to crash with a 403 on every startup. The fix was to switch to managed identity-based storage authentication using `AzureWebJobsStorage__accountName`, `AzureWebJobsStorage__credential`, and `AzureWebJobsStorage__clientId` pointing to the pre-provisioned managed identity `mi-pa4-26100204`.

**Challenge 2: GitHub Actions workflow not finding webapp/package.json.** The account has ran out of credits to make a workflow so we had to make a new account.

**The task 6 error due to no access to storage**: Couldnt do task 6 because of this error `The request may be blocked by network rules of storage account. Please check network rule set using az storage account show -n accountname --query networkRuleSet.If you want to change the default action to apply when no rule matches, please use az storage account update.`

**Task 7** wasnt working since it was a pipeline and mine was broken in alot of places in the middle