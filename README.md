# CST8919 - Flask Auth0 Azure Web App - Lab Report

This README documents the steps to deploy a Flask web app with Auth0 authentication to Azure App Service using **GitHub Actions**. It also includes Log Analytics integration and alert rules for suspicious access.

---

##  Features

- Auth0 OAuth2 login flow
- Flask session management
- Protected routes with user logging
- Deployment to Azure App Service (Linux)
- CI/CD via GitHub Actions
- Diagnostic logs to Log Analytics
- KQL queries to detect suspicious behavior
- Azure Monitor Alerts

---

##  Project Structure

```bash
├── app.py
├── requirements.txt
├── templates/
└── .env.example
```

---

## Local Development

 Clone your repo:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

 Install dependencies:

```bash
 pip install -r requirements.txt
```

Create your .env

```bash
AUTH0_CLIENT_ID=YOUR_CLIENT_ID
AUTH0_CLIENT_SECRET=YOUR_SECRET
AUTH0_DOMAIN=YOUR_DOMAIN
AUTH0_CALLBACK_URL=http://localhost:5000/callback
FLASK_SECRET_KEY=YOUR_SECRET
APP_BASE_URL=http://localhost:5000
```

 Run locally with Gunicorn:

```bash
flask run
```

# Deploy to Azure via GitHub Actions
Steps:

Push your code to GitHub.

In Azure Portal > App Service > Deployment Center:

Source: GitHub

Repo: Your GitHub repo

Branch: main

Build provider: GitHub Actions

Azure creates .github/workflows/azure-webapp.yml in your repo

```bash
name: Build and deploy Python app to Azure Web App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.10'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: YOUR_AZURE_APP_NAME
        publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE }}
        package: .
```

Download the Publish Profile from Azure:

Azure Portal > App Service > Overview > Get Publish Profile

Add the secret to GitHub:

Name: AZUREAPPSERVICE_PUBLISHPROFILE

Value: Paste the full publish profile XML

Result: Pushing to main automatically deploys your app.

# Download the Publish Profile from Azure:

Azure Portal > App Service > Overview > Get Publish Profile

Add the secret to GitHub:

Name: AZUREAPPSERVICE_PUBLISHPROFILE

Value: Paste the full publish profile XML

Result: Pushing to main automatically deploys your app.

```bash
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

App Settings (Environment Variables in Azure):

```bash
AUTH0_CLIENT_ID
AUTH0_CLIENT_SECRET
AUTH0_DOMAIN
AUTH0_CALLBACK_URL
FLASK_SECRET_KEY
APP_BASE_URL
```

# Diagnostic Logs to Log Analytics
In Azure Portal:

App Service > Diagnostic Settings

Enable Console Logs

Send logs to your Log Analytics Workspace (e.g., CST8919Workspace)

Confirm logs in Log Analytics:

```bash
INFO:app:LOGIN_ATTEMPT
INFO:app:LOGIN_SUCCESS
INFO:app:PROTECTED_ACCESS
```

# Example KQL Query
To detect suspicious repeated access to /protected:

AppServiceConsoleLogs
| where ResultDescription has "PROTECTED_ACCESS"
| where TimeGenerated > ago(15m)
| parse ResultDescription with * "user_id=" user_id "," *
| summarize AccessCount = count() by user_id
| where AccessCount > 10
| project user_id, AccessCount

# Azure Alerts
Alert Rule:

Scope: Log Analytics Workspace

Condition: Custom Log Search (your KQL)

Threshold: Number of records > 0

Evaluation window: 5-15 minutes

Action Group: Email or SMS

Result: Get notified when a user abuses /protected route.


# Demo Video 
https://youtu.be/2PtuQz6XrHo
