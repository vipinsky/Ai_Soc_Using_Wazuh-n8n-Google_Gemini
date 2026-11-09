# AI SOC Using Wazuh + n8n + Gemini

**Author:** Vipin Kumar  
**LinkedIn:** https://www.linkedin.com/in/vipin-kumar-530a55283/  
**GitHub:** https://github.com/vipinsky

An AI-assisted SOC lab project that connects Wazuh alert data to an n8n automation workflow and uses Google Gemini to produce readable alert summaries.

## Project overview

Security analysts often need to inspect many SIEM events, identify priority alerts, notice repeated rules and explain what the activity may indicate. This project demonstrates a workflow in which an analyst can ask an alert-related question through a chat-style SOC Assistant Console. n8n receives the request, queries Wazuh alert data and uses Gemini to generate a concise summary.

The solution is designed as an **investigation assistant**. AI-generated explanations should be checked against the original Wazuh events before an analyst makes a security decision.

## Lab architecture

The environment is split across two virtual machines:

| Environment | Components | Purpose |
|---|---|---|
| VM 1 - Wazuh server | Wazuh Manager, Wazuh Dashboard, Wazuh Indexer | Hosts the SIEM services, monitoring interface and indexed alert data. |
| VM 2 - automation host | Wazuh Agent, Docker, n8n container | Runs a monitored endpoint and hosts the n8n automation workflow. |
| AI service | Google Gemini API / Gemini model node | Produces a human-readable summary from alert information supplied by the workflow. |
| User interface | Chat-style SOC Assistant Console | Accepts natural-language questions and displays the workflow response. |

## Workflow

1. An analyst submits a question in the SOC Assistant Console.
2. The request is sent to the n8n webhook.
3. The n8n AI Agent interprets the request using the configured SOC system instructions.
4. The agent uses an HTTP Request tool configured with the POST method, the Wazuh URL and Wazuh authentication to request alert data.
5. The instructions direct the agent to classify and group the returned alerts.
6. The Gemini model generates a concise summary and highlights repeated rules, patterns or anomalies.
7. The webhook response node returns the result to the interface.

### Main n8n components

- **Webhook:** receives the incoming request.
- **AI Agent:** coordinates the alert-analysis workflow.
- **HTTP Request:** retrieves alert data from Wazuh.
- **Gemini model:** summarizes the retrieved information.
- **Simple Memory:** connected to the AI Agent for context handling.
- **Webhook Response:** returns the generated result.

## Alert analysis logic

The AI Agent system message specifies this project-specific severity mapping using the Wazuh `rule.level` value:

| `rule.level` | Severity |
|---|---|
| 0-3 | Low |
| 4-7 | Medium |
| 8-11 | High |
| 12 and above | Critical |

The agent is instructed to:

- Query the Wazuh index using user-provided parameters, such as alert count or time range.
- Use the latest **100 alerts from the past 24 hours** when no parameters are provided.
- Group and categorize alerts by severity, type and frequency.
- Highlight repeated rules, notable patterns and anomalies.
- Use Gemini to generate a clear, concise summary.

> **Severity mapping note:** The Wazuh Dashboard screenshot supplied with this project displays different bands: Low 0-6, Medium 7-11, High 12-14 and Critical 15+. These differ from the custom mapping in the AI Agent prompt. Before using this design operationally, align the assistant with the dashboard mapping or formally document and label the custom mapping. Otherwise, the same event may receive different severity labels in the two views.

## Demonstrated project elements

The supplied project evidence includes:

- A two-VM Wazuh and automation setup.
- Docker-based n8n deployment.
- An n8n workflow containing the webhook, AI Agent, Gemini, Simple Memory, HTTP Request and response components.
- An HTTP POST configuration for retrieving Wazuh alert data using authentication.
- A Wazuh monitoring dashboard and alert/event view.
- A chat-style SOC Assistant Console and a sample generated alert breakdown.

One sample assistant output in the screenshots displays 10 alerts: 6 Low (60%), 4 Medium (40%), and 0 High or Critical. It also shows a File Integrity Monitoring category example. These are values from a captured example, not benchmark results or a guarantee of detection accuracy.

## Technology stack

- Wazuh SIEM
- n8n workflow automation
- Docker
- Google Gemini API / Gemini model node
- HTTP Request / REST API integration
- Webhooks
- Virtual machines
- SOC alert triage and log analysis concepts

## Security and reliability considerations

Before exposing this workflow beyond a controlled lab:

- Store Wazuh credentials and Gemini API keys in n8n Credentials or an approved secret manager. Never commit secrets to GitHub.
- Protect the webhook with HTTPS, authentication or signed requests, network restrictions and rate limits.
- Use a dedicated read-only Wazuh account with access limited to the required alert data.
- Review which alert fields are sent to Gemini and remove unnecessary personal or sensitive information.
- Treat log contents as untrusted data, not as instructions for the AI Agent.
- Require analyst verification of AI-generated findings before taking response actions.
- Add timeouts, appropriate retries, clear error handling, pagination limits and workflow execution logging.
- Test with known benign events and controlled test alerts, then document false positives, missed alerts, latency and API failures.

## Current scope and documentation limits

The supplied implementation notes and screenshots describe the architecture and show example screens. They do not provide the exact Wazuh index endpoint, query body, full n8n workflow export, deployment scripts, credential-storage method, webhook security configuration, formal test cases or performance measurements. Those details should be added to the repository once they are available and have been safely redacted.

## Resume-ready summary

Built a two-VM lab integrating Wazuh Manager, Dashboard and Indexer with a Wazuh Agent and Docker-hosted n8n. Designed a webhook-driven AI Agent workflow that retrieves Wazuh alert data through an authenticated HTTP POST request, applies a project-specific `rule.level` severity mapping, groups events by severity, type and frequency, and uses Google Gemini to generate analyst-readable summaries.

## Conclusion

This project demonstrates how SIEM alert data can be connected to workflow automation and AI-assisted analysis to make security events easier to query and understand. It combines Wazuh monitoring, n8n orchestration and Gemini summarization while keeping analyst validation central to the investigation process. Further work on severity consistency, secret management, webhook security, repeatable testing and operational reliability will strengthen the project for broader use.
