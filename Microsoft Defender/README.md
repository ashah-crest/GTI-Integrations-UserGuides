# Google Threat Intelligence → Microsoft Defender Integration

## Overview

This solution enables integration between **Google Threat Intelligence (GTI)** and **Microsoft Defender for Endpoint (MDE)** using Azure Functions.

It automates ingestion of threat intelligence indicators (IOCs) from GTI into Defender, allowing security teams to operationalize external intelligence and strengthen endpoint protection.

---

## Solution Overview

The integration performs the following:

- Retrieves threat intelligence data from GTI
- Transforms data into Microsoft Defender-compatible format
- Ingests indicators into Defender as custom indicators
- Runs on a scheduled basis using Azure Functions

This enables near real-time enrichment of Defender with global threat intelligence.

---

## Use Cases

- Automate ingestion of external threat intelligence into Defender
- Reduce manual IOC management
- Enhance endpoint detection with high-fidelity threat signals
- Improve SOC response time with continuously updated indicators

---

## Prerequisites

Before deployment, ensure you have:

- GTI API Key
- Azure Subscription
- Microsoft Defender for Endpoint (E5)
- Microsoft Defender Security Administrator access
- Microsoft Intune Admin access _(optional)_

### Required Credentials

You will need:

- Client ID
- Client Secret
- Tenant ID
- User Object ID

> Steps to generate Client ID, Client Secret, and Tenant ID are covered in the **App Registration** section below.

---

## Deployment

### Step 1: App Registration (Microsoft Entra)

Create and configure an application for authentication.

#### Create App Registration

- Navigate to: `Microsoft Entra Admin Center → Identity → Applications → App registrations`
- Click **New registration**
- Choose **Single tenant**
- Leave Redirect URI blank

#### Collect Required IDs

From the Overview page:

- Application (Client) ID → **Client ID**
- Directory (Tenant) ID → **Tenant ID**

#### Create Client Secret

- Go to: `Certificates & secrets`
- Click \*\*New client secret`
- Copy the value immediately

#### Configure API Permissions

- Navigate to: `API permissions → Add permission`
- Select: `WindowsDefenderATP`

Add:

- `Ti.ReadWrite.All` (Application)
- `Ti.ReadWrite` (Delegated)

#### Grant Admin Consent

- Click **Grant admin consent**

---

### Step 2: Get User Object ID

- Navigate to: `Microsoft Entra Admin Center → Users`
- Select your user
- Copy the **Object ID**

---

### Step 3: Deploy to Azure

Click **Deploy to Azure**.

You have two deployment options based on the Azure function plan:



#### Plan Selection 

| Plan | Description | Action |
|----------|----------|----------|
| Consumption | Cost-efficient, auto-scaling, suitable for most workloads | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FVirusTotal%2Fgti-Microsoft-Defender%2Frefs%2Fheads%2Fmain%2Fgti.json) |
| Premium    | Better performance, no cold starts, longer execution duration    | [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FVirusTotal%2Fgti-Microsoft-Defender%2Frefs%2Fheads%2Fmain%2Fgti-premium.json)    |




📌 Reference:

- Azure Functions Pricing: https://azure.microsoft.com/en-us/pricing/details/functions/
- Pricing Calculator: https://azure.microsoft.com/en-us/pricing/calculator/

---

### Deployment Parameters

Provide the following values:

| Parameter      | Description           |
| -------------- | --------------------- |
| App Name       | Function App name     |
| GTI API Key    | Authentication key    |
| Threat List    | Optional GTI list IDs |
| Lookback Days  | Max 7 days            |
| Threat Score   | 0–100                 |
| Timer Schedule | CRON expression       |
| Client ID      | From App Registration |
| Client Secret  | From App Registration |
| Tenant ID      | From App Registration |
| User Object ID | From Entra            |

---

### Step 4: (Optional) Enable Defender–Intune Integration

This step is optional and only required for device compliance and mobile protection scenarios.

- Navigate to: `Intune Admin Center → Endpoint Security → Microsoft Defender for Endpoint`
- Open Defender portal:
  - `Settings → Endpoints → Advanced features`
- Enable:
  - Microsoft Intune connection

---

## Configuration

### Data Ingestion

- **Threat Lists** – Select specific GTI feeds
- **Lookback Days** – Historical ingestion window (max 7 days)

### Filtering

- Verdicts
- Severities
- Threat Score (0–100)

### Scheduling

- Configured via CRON expression
- Recommended: hourly execution

---

## Viewing Results

### Ingested Indicators

- Navigate to: `https://security.microsoft.com`
- Go to:
  - `Settings → Endpoints → Rules → Indicators`

---

### Failed Indicators

Failed IOCs are stored in Azure Table Storage:

- Table Name: `FailedIOCs`

---

## Limitations

- Maximum **15,000 indicators** supported in Defender
- Ingestion API may return inconsistent counts
- Consumption plan timeout: **10 minutes**
  - Remaining data is processed in subsequent runs

---
