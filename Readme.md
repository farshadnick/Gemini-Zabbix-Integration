Zabbix-Gemini-AI-Webhook
An Engineer's Tool: Smart Alerts for Zabbix with Gemini AI
Table of Contents

    Introduction

    What It Does (Features)

    Before You Start (Prerequisites)

    How to Set It Up

        1. Get Your Gemini API Key

        2. Configure Zabbix Media Type

        3. Set Up Zabbix User Media

        4. Create a Zabbix Action

    Fine-Tuning (Customization)

    If Something Goes Wrong (Troubleshooting)

    Want to Help? (Contributing)

    License

    Contact

Introduction

As engineers, we deal with Zabbix alerts constantly. Sometimes, it's hard to quickly understand what's happening or what to do next. This project, Zabbix-Gemini-AI-Webhook, is a simple but powerful tool I built to make our lives easier.

It connects Zabbix with Google's Gemini AI. When Zabbix sends an alert, this webhook takes the alert details, sends them to Gemini, and Gemini gives us quick suggestions: what might be causing the problem, how to fix it, useful commands to check things, and even ideas to stop it from happening again.

Think of it as having a smart assistant for every alert. It helps us understand and fix issues faster, which means less downtime and less stress.
What It Does (Features)

    Smart Troubleshooting: Get immediate AI-driven ideas for alert causes, solutions, debug commands, and prevention steps.

    Language Choice: You can choose if Gemini responds in English or Spanish.

    Control Response Length: Get short, to-the-point answers (around 10 lines) or more detailed explanations.

    Focus the AI: Tell Gemini what to focus on. For example, "root cause," "network issues," or "performance."

    Adjust Gemini's Brain: You can tweak how Gemini thinks using temperature (for creativity), maxOutputTokens (for length), topP, and topK directly from Zabbix.

    Better Error Messages: If something breaks, the system gives clearer messages, so it's easier to find the problem.

Before You Start (Prerequisites)

Make sure you have these ready:

    Zabbix Server: Version 5.0 or newer.

    Internet Access: Your Zabbix server needs to be able to connect to https://generativelanguage.googleapis.com.

    Google Account: You'll need one to get your Gemini API key.

How to Set It Up

Here's how to get this working in your Zabbix environment.
1. Get Your Gemini API Key

    Go to Google AI Studio.

    Follow the steps to create a new API key.

    Important: Copy this key and keep it safe. It's like a password for Gemini.

2. Configure Zabbix Media Type

    Log into Zabbix as an administrator.

    Go to Administration -> Media types.

    Click Create media type.

    Fill in these details:

        Name: Gemini AI Webhook (or something similar you'll remember)

        Type: Webhook

        Parameters: Add these parameters. These are just names; the actual values come later.

            api_key (Type: String)

            alert_subject (Type: String)

            temperature (Type: Number, Optional, Default: 0.7)

            maxOutputTokens (Type: Number, Optional, Default: 200)

            topP (Type: Number, Optional, Default: 0.9)

            topK (Type: Number, Optional, Default: 40)

            prompt_language (Type: String, Optional, Default: English)

            prompt_length (Type: String, Optional, Default: concise)

            prompt_focus (Type: String, Optional, Default: causes, solutions, debug commands, mitigation)

        Script: Copy and paste the entire JavaScript code block below into this field.

    var Gemini = {
        params: {},
        setParams: function(params) {
            if (typeof params !== 'object') {
                throw 'Invalid parameters provided to Gemini.setParams.';
            }
            if (typeof params.api_key !== 'string' || params.api_key === '') {
                throw 'API key for Gemini is required and cannot be empty.';
            }
            Gemini.params = params;
            // Default URL, can be overridden if needed in params
            Gemini.params.url = Gemini.params.url || 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent';
            Zabbix.log(4, '[ Gemini Webhook ] Gemini parameters set: ' + JSON.stringify(Gemini.params));
        },
        request: function(data) {
            if (!Gemini.params.api_key) {
                throw 'Gemini API key is missing. Call Gemini.setParams first.';
            }

            var request = new HttpRequest();
            request.addHeader('Content-Type: application/json');

            var urlWithKey = Gemini.params.url + '?key=' + Gemini.params.api_key;

            // Add optional model parameters if they exist
            if (Gemini.params.temperature !== undefined) {
                if (!data.generationConfig) data.generationConfig = {};
                data.generationConfig.temperature = Gemini.params.temperature;
            }
            if (Gemini.params.maxOutputTokens !== undefined) {
                if (!data.generationConfig) data.generationConfig = {};
                data.generationConfig.maxOutputTokens = Gemini.params.maxOutputTokens;
            }
            if (Gemini.params.topP !== undefined) {
                if (!data.generationConfig) data.generationConfig = {};
                data.generationConfig.topP = Gemini.params.topP;
            }
            if (Gemini.params.topK !== undefined) {
                if (!data.generationConfig) data.generationConfig = {};
                data.generationConfig.topK = Gemini.params.topK;
            }

            Zabbix.log(4, '[ Gemini Webhook ] Sending request to: ' + urlWithKey + '\nPayload: ' + JSON.stringify(data));
            var response = request.post(urlWithKey, JSON.stringify(data));

            Zabbix.log(4, '[ Gemini Webhook ] Received response with status code ' + request.getStatus() + '\nResponse: ' + response);

            if (request.getStatus() < 200 || request.getStatus() >= 300) {
                var errorMessage = 'Gemini API request failed with status code ' + request.getStatus() + '.';
                try {
                    var errorResponse = JSON.parse(response);
                    if (errorResponse && errorResponse.error && errorResponse.error.message) {
                        errorMessage += ' Error: ' + errorResponse.error.message;
                    }
                } catch (e) {
                    Zabbix.log(4, '[ Gemini Webhook ] Failed to parse error response: ' + e);
                }
                throw errorMessage;
            }

            try {
                response = JSON.parse(response);
            } catch (error) {
                Zabbix.log(4, '[ Gemini Webhook ] Failed to parse JSON response from Gemini: ' + error);
                throw 'Invalid JSON response received from Gemini API.';
            }
            return response;
        }
    };

    try {
        var params = JSON.parse(value),
            data = {},
            result = "",
            required_params = ['api_key', 'alert_subject'];

        Object.keys(required_params).forEach(function(key) {
            if (params[key] === undefined || params[key] === null || params[key] === '') {
                throw 'Missing or empty required parameter: "' + key + '".';
            }
        });

        var geminiConfig = {
            api_key: params.api_key,
            temperature: params.temperature !== undefined ? parseFloat(params.temperature) : 0.7,
            maxOutputTokens: params.maxOutputTokens !== undefined ? parseInt(params.maxOutputTokens) : 200,
            topP: params.topP !== undefined ? parseFloat(params.topP) : 0.9,
            topK: params.topK !== undefined ? parseInt(params.topK) : 40
        };

        var prompt_language = params.prompt_language || 'English';
        var prompt_length = params.prompt_length || 'concise';
        var prompt_focus = params.prompt_focus || 'causes, solutions, debug commands, mitigation';

        var basePrompt = "";
        if (prompt_language === 'Spanish') {
            basePrompt = "La alerta: " + params.alert_subject + " ocurrió en Zabbix. ";
            if (prompt_length === 'concise') {
                basePrompt += "Sugiere posibles " + prompt_focus + " para resolver este problema. No hagas un texto grande, " +
                              "solamente unas 10 líneas.";
            } else {
                basePrompt += "Proporciona un análisis detallado de las posibles " + prompt_focus + " para resolver este problema. " +
                              "Incluye pasos detallados y ejemplos si es posible.";
            }
        } else {
            basePrompt = "The alert: " + params.alert_subject + " occurred in Zabbix. ";
            if (prompt_length === 'concise') {
                basePrompt += "Suggest possible " + prompt_focus + " to resolve this issue. Keep it concise, " +
                              "around 10 lines.";
            } else {
                basePrompt += "Provide a detailed analysis of possible " + prompt_focus + " to resolve this issue. " +
                              "Include detailed steps and examples where applicable.";
            }
        }

        data = {
            contents: [{
                parts: [{
                    text: basePrompt
                }]
            }]
        };

        Gemini.setParams(geminiConfig);

        var response = Gemini.request(data);

        if (response && response.candidates && response.candidates.length > 0 && response.candidates[0].content && response.candidates[0].content.parts && response.candidates[0].content.parts.length > 0) {
            result = response.candidates[0].content.parts[0].text.trim();
        } else {
            Zabbix.log(3, '[ Gemini Webhook ] Gemini response did not contain expected content: ' + JSON.stringify(response));
            throw 'Gemini did not return a valid response or content.';
        }

        return result;

    } catch (error) {
        Zabbix.log(3, '[ Gemini Webhook ] ERROR: ' + error);
        throw 'Sending failed: ' + error;
    }

3. Set Up Zabbix User Media

    Go to Administration -> Users.

    Pick the user(s) who should get these smart alerts.

    Go to the Media tab and click Add.

    Set these options:

        Type: Choose Gemini AI Webhook (the one you just made).

        Send to: Leave empty or fill if needed.

        Use if severity: Select the alert severities for which you want AI help.

        Enabled: Tick this box.

        Parameters: Here's where your Gemini API key and other settings go. For security, it's a good idea to use a Zabbix global macro for the API key.

            Example:

            {
                "api_key": "YOUR_GEMINI_API_KEY_HERE",
                "temperature": 0.7,
                "maxOutputTokens": 250,
                "prompt_language": "English",
                "prompt_length": "concise",
                "prompt_focus": "causes, solutions, debug commands, mitigation"
            }

            Replace YOUR_GEMINI_API_KEY_HERE with your actual API key. You can also define {$GEMINI_API_KEY} as a global macro in Zabbix for better security.

4. Create a Zabbix Action

    Go to Configuration -> Actions -> Trigger actions.

    Click Create action.

    Action tab:

        Name: Send Smart Alert (Gemini) (or similar)

        Conditions: Define when this action should run (e.g., for specific servers, alert levels, etc.).

    Operations tab:

        Click Add under "Operations".

        Operation type: Send message

        Send to Users: Select the user(s) you set up in Step 3.

        Send only to: Choose Gemini AI Webhook.

        Default message: Uncheck this.

        Custom message: Check this.

        Subject: AI Insight for: {EVENT.NAME}

        Message: This is important. It's a JSON text that goes to the webhook.

        {
            "api_key": "{$GEMINI_API_KEY}",
            "alert_subject": "{EVENT.NAME}",
            "temperature": 0.7,
            "maxOutputTokens": 250,
            "prompt_language": "English",
            "prompt_length": "detailed",
            "prompt_focus": "root cause, solution steps, impact, troubleshooting commands"
        }

            Crucial: api_key and alert_subject are a must. The other settings (temperature, maxOutputTokens, prompt_language, prompt_length, prompt_focus) are optional. You can set them here or in the user media/script. Using Zabbix macros like {$GEMINI_API_KEY} and {EVENT.NAME} is highly recommended.

    Recovery operations / Update operations: Set these up if you want specific messages when problems are resolved or updated (maybe without AI analysis).

    Enable the action.

Fine-Tuning (Customization)

This tool has several settings to change how Gemini works and what kind of answers it gives. You can set these in the Zabbix User Media or directly in the Zabbix Action message.

    api_key (Required): Your Google Gemini API key.

    alert_subject (Required): The main topic of the Zabbix alert (e.g., {EVENT.NAME}).

    temperature (Optional, default: 0.7): Controls how "creative" or "random" Gemini's answer is. Higher numbers (like 0.9) mean more varied answers; lower numbers (like 0.2) mean more direct and focused answers.

    maxOutputTokens (Optional, default: 200): The maximum length of Gemini's response (roughly how many words). Good for keeping answers short.

    topP (Optional, default: 0.9): A technical setting for how Gemini picks words.

    topK (Optional, default: 40): Another technical setting for word selection.

    prompt_language (Optional, default: "English"): Sets the language for your question to Gemini and its answer. Options: "English", "Spanish".

    prompt_length (Optional, default: "concise"): How long the answer should be. Options: "concise" (around 10 lines), "detailed".

    prompt_focus (Optional, default: "causes, solutions, debug commands, mitigation"): A list of keywords (separated by commas) to tell Gemini what specific information you need. Examples:

        "root cause, solution steps, impact"

        "network issues, firewall rules, connectivity tests"

        "performance bottlenecks, resource utilization, optimization tips"

If Something Goes Wrong (Troubleshooting)

If you face any issues, here's how to check:

    Zabbix Server Logs: Look in your Zabbix server logs (usually /var/log/zabbix/zabbix_server.log). You can set DebugLevel to 4 in zabbix_server.conf for more details about the webhook.

    JSON Response Error: If you see "Invalid JSON response received from Gemini API," it might be a problem with your API key, internet connection, or how the request was built.

    "Gemini did not return a valid response or content": This means the API call worked, but Gemini didn't send back a useful answer. Check the raw response in the Zabbix logs (DebugLevel 4).

    API Key Check: Double-check your Gemini API key for any typos.

    Network Check: Make sure your Zabbix server can connect to https://generativelanguage.googleapis.com on port 443.

Want to Help? (Contributing)

If you have ideas to make this better, find bugs, or want to add new features, please help!

    Fork this project (make your own copy).

    Create a new branch for your changes (git checkout -b feature/YourNewIdea).

    Make your changes.

    Commit your changes (git commit -am 'Add new feature').

    Push your changes to your branch (git push origin feature/YourNewIdea).

    Open a Pull Request here.

License

This project is open-source under the MIT License. You can find the full details in the LICENSE file.
Contact

For any questions or feedback, please open an issue on this GitHub repository. I'll do my best to help.

Please remember to replace YOUR_USERNAME in the badge URLs and YOUR_GEMINI_API_KEY_HERE with your actual GitHub username and API key/macro.
