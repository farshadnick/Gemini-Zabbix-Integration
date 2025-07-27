**An Engineer's Tool: Smart Alerts for Zabbix with Gemini AI**

---

## Table of Contents

- [Introduction](#introduction)
- [What It Does (Features)](#what-it-does-features)
- [Before You Start (Prerequisites)](#before-you-start-prerequisites)
- [How to Set It Up](#how-to-set-it-up)  
  1. [Get Your Gemini API Key](#1-get-your-gemini-api-key)  
  2. [Configure Zabbix Media Type](#2-configure-zabbix-media-type)  
  3. [Set Up Zabbix User Media](#3-set-up-zabbix-user-media)  
  4. [Create a Zabbix Action](#4-create-a-zabbix-action)
- [Fine-Tuning (Customization)](#fine-tuning-customization)
- [If Something Goes Wrong (Troubleshooting)](#if-something-goes-wrong-troubleshooting)
- [Want to Help? (Contributing)](#want-to-help-contributing)
- [License](#license)
- [Contact](#contact)

---

## Introduction

As engineers, we deal with Zabbix alerts constantly. Sometimes, it's hard to quickly understand what's happening or what to do next. This project, **Zabbix-Gemini-AI-Webhook**, is a simple but powerful tool I built to make our lives easier.

It connects Zabbix with Google's Gemini AI. When Zabbix sends an alert, this webhook takes the alert details, sends them to Gemini, and Gemini gives us quick suggestions: what might be causing the problem, how to fix it, useful commands to check things, and even ideas to stop it from happening again.

Think of it as having a smart assistant for every alert. It helps us understand and fix issues faster, which means less downtime and less stress.

---

## What It Does (Features)

- **Smart Troubleshooting**: Get immediate AI-driven ideas for alert causes, solutions, debug commands, and prevention steps.
- **Language Choice**: Choose between English or Spanish for Gemini’s responses.
- **Control Response Length**: Get short (~10 lines) or detailed answers.
- **Focus the AI**: Set focus areas like "root cause," "network issues," or "performance."
- **Adjust Gemini's Brain**: Customize temperature, maxOutputTokens, topP, and topK from Zabbix.
- **Better Error Messages**: Clear error messages for easier debugging.

---

## Before You Start (Prerequisites)

Make sure you have these ready:

- **Zabbix Server**: Version 5.0 or newer.
- **Internet Access**: Server must connect to `https://generativelanguage.googleapis.com`.
- **Google Account**: To get your Gemini API key.

---

## How to Set It Up

### 1. Get Your Gemini API Key

- Go to [Google AI Studio](https://aistudio.google.com/).
- Create a new API key.
- **Important**: Save this key. It’s like a password for Gemini.

---

### 2. Configure Zabbix Media Type

- Log into Zabbix as an admin.
- Go to `Administration -> Media types`.
- Click **Create media type**.
- Use these details:
  - **Name**: `Gemini AI Webhook`
  - **Type**: `Webhook`
  - **Parameters**:
    ```text
    api_key (String)
    alert_subject (String)
    temperature (Number, Optional, Default: 0.7)
    maxOutputTokens (Number, Optional, Default: 200)
    topP (Number, Optional, Default: 0.9)
    topK (Number, Optional, Default: 40)
    prompt_language (String, Optional, Default: English)
    prompt_length (String, Optional, Default: concise)
    prompt_focus (String, Optional, Default: causes, solutions, debug commands, mitigation)
    ```

- **Script**: Paste the entire JavaScript code provided [here](#).

---

### 3. Set Up Zabbix User Media

- Go to `Administration -> Users`.
- Select the user(s) for AI alerts.
- Go to the **Media** tab → **Add**.
- Options:
  - **Type**: `Gemini AI Webhook`
  - **Send to**: Leave blank or fill in as needed.
  - **Use if severity**: Select desired alert levels.
  - **Enabled**: ✅
  - **Parameters**:
    ```json
    {
      "api_key": "YOUR_GEMINI_API_KEY_HERE",
      "temperature": 0.7,
      "maxOutputTokens": 250,
      "prompt_language": "English",
      "prompt_length": "concise",
      "prompt_focus": "causes, solutions, debug commands, mitigation"
    }
    ```
<img width="953" height="614" alt="image" src="https://github.com/user-attachments/assets/a1e4fe3b-51b1-490b-97b9-b368f0504bff" />
<img width="789" height="326" alt="image" src="https://github.com/user-attachments/assets/0efb1428-f6b4-4b88-95c0-3351bbb6f719" />


> 🔐 Tip: Use Zabbix global macro like `{$GEMINI_API_KEY}` instead of hardcoding your API key.

---

### 4. Create a Zabbix Action

- Go to `Configuration -> Actions -> Trigger actions`.
- Click **Create action**.

**Action Tab**:

- **Name**: `Send Smart Alert (Gemini)`
- **Conditions**: Set based on server/group/severity.

**Operations Tab**:

- Click **Add**:
  - **Operation type**: `Send message`
  - **Send to Users**: Choose user(s)
  - **Send only to**: `Gemini AI Webhook`
  - **Default message**: ❌
  - **Custom message**: ✅

**Subject**:

AI Insight for: {EVENT.NAME}

Always show details

**Message**:
```json
{
  "api_key": "{$GEMINI_API_KEY}",
  "alert_subject": "{EVENT.NAME}",
  "temperature": 0.7,
  "maxOutputTokens": 250,
  "prompt_language": "English",
  "prompt_length": "detailed",
  "prompt_focus": "root cause, solution steps, impact, troubleshooting commands"
}
