# Telegram AI Assistant for "Matrix - Trading Matematico"

This n8n workflow powers an AI assistant on Telegram designed to provide information and support for the "Matrix - Trading Matematico" project. It leverages a Notion database as its primary knowledge source and includes a user authorization layer via Google Sheets. The agent primarily communicates in Italian.

## Purpose & Goals

The main objectives of this workflow are:

* To provide timely and accurate answers to user queries on Telegram regarding the "Matrix - Trading Matematico" project.
* To serve as an automated information hub, drawing information from a structured Notion knowledge base.
* To handle both text and voice messages from users.
* To ensure that only authorized users can interact with the assistant.
* To offer a rich, pre-defined set of FAQs and initial information directly within its core logic.

## Key Features

* **Telegram Integration:** Triggers on incoming Telegram messages (text and voice).
* **Voice Message Processing:** Downloads voice notes and transcribes them using OpenAI (Whisper).
* **Content-Type Routing:** Differentiates between text, voice, and unsupported message types.
* **User Authorization:** Checks against a Google Sheet to verify if the interacting Telegram user is authorized. Unauthorized users receive a standard denial message.
* **AI-Powered Q&A:**
    * Uses a Langchain AI Agent powered by Google Gemini ("gemini-2.0-flash" model).
    * The Agent is primed with an extensive system prompt in Italian, detailing its role, behavior, error handling, output format, and a large amount of pre-loaded information about the "Matrix - Trading Matematico" project (welcome message, FAQs, how to start, pricing, strategies, tools, links).
* **Notion Knowledge Base Integration:** The AI Agent is equipped with tools to:
    * Search a specific Notion database ("Matrix Knowledge Base") by keyword or tag.
    * Retrieve content from specific Notion pages.
* **Conversational Memory:** Maintains conversation history for contextual interactions.
* **HTML Escaping for Responses:** Includes a step to format Telegram responses by escaping HTML special characters to prevent parsing errors.
* **Primary Language:** Italian (with capability for English if explicitly requested, as per prompt).

## Workflow Breakdown

1.  **Message Reception & Pre-processing (Telegram Trigger & Initial Nodes):**
    * The workflow starts when a message is received via the Telegram Trigger.
    * A "typing" action is sent to the user.
    * A Switch node ("Determine content type") routes the flow based on whether the message is text, voice, or another type.
    * Unsupported types receive an error message.

2.  **Voice Message Handling:**
    * If a voice message is detected, it's downloaded using the Telegram node.
    * The audio file is then sent to OpenAI for transcription.

3.  **Input Consolidation (Set Node - "Combine content and set properties"):**
    * The original text message or the transcribed text from a voice message is combined.
    * Properties like message type and whether the message was forwarded are set.

4.  **Notion Database Schema Preparation (Notion & Set Nodes):**
    * Details from the target Notion knowledge base (e.g., database ID, name, tag options) are fetched ("Get database details").
    * This information, along with the user's input and session ID (Telegram chat ID), is formatted ("Format schema") to be passed to the AI Agent.

5.  **AI Agent Interaction (Langchain AI Agent - "AI Agent1"):**
    * **Core Logic:** The formatted input is sent to the AI Agent.
    * **LLM:** Google Gemini ("gemini-2.0-flash") model.
    * **Memory:** Conversational history is maintained using the "Window Buffer Memory1" node.
    * **System Prompt:** A very detailed Italian prompt guides the agent's behavior, including its persona for the "Matrix - Trading Matematico" project, how to use its tools, the authorization flow, and a substantial amount of pre-defined project information.
    * **Tools:**
        * `Google Sheets - Reads`: Checks if the `message.from.id` exists in an "Utenti Autorizzati" Google Sheet. If not, the agent is instructed to reply with an access denied message.
        * `Search notion database`: Queries the Notion knowledge base.
        * `Search inside database record`: Fetches content from specific Notion pages.
    * The agent processes the user's query according to its prompt, potentially using its tools.

6.  **Response Formatting & Delivery:**
    * The agent's generated output is prepared for Telegram.
    * An attempt is made to extract a Notion URL if relevant ("Extract Notion URL" Set node - *the exact utility here might depend on specific agent tool responses*).
    * The response is first sent via "Send final reply" (Telegram node).
    * If this send operation encounters an error (often due to special characters in HTML mode), the error output is caught, and the "Correct errors" Telegram node attempts to resend the message after escaping common HTML entities (`&, <, >, "`).

## Key Nodes Used

* **Telegram Trigger:** Starts the workflow on new messages.
* **Telegram Node (Download file, Send message, Send chat action):** For various Telegram interactions.
* **Switch Node:** For content-type routing.
* **OpenAI Node (Transcribe):** For audio-to-text conversion.
* **Set Node:** For data preparation and manipulation.
* **Notion Node (Get database):** To fetch metadata about the Notion KB.
* **Langchain AI Agent:** The core conversational AI.
* **Langchain Google Gemini Chat Model:** The LLM for the agent.
* **Langchain Window Buffer Memory:** For conversation history.
* **Langchain Tool - HTTP Request (for Notion):** Custom tools to query Notion API.
* **Google Sheets Tool:** Used by the agent for user authorization.

## External Services & APIs Utilized

* Telegram Bot API
* OpenAI API (for Whisper transcription)
* Google Gemini API (for the chat model)
* Notion API (for the knowledge base)
* Google Sheets API (for user authorization)

## Authorization Mechanism

A critical part of this workflow is user authorization:
* The AI Agent's system prompt explicitly instructs it to use the "Google Sheets - Reads" tool.
* This tool checks if the Telegram user's ID (`message.from.id`) is present in a predefined list of authorized users stored in a Google Sheet (column "Utenti" in the "Utenti Autorizzati" sheet).
* If the user is not found, the agent is instructed to respond with a specific denial message in Italian: "Mi dispiace, non sei autorizzato a usare questo assistente." and provide no further assistance.

## Core AI Agent Functionality (Highlights from System Prompt)

* **Persona:** Expert AI assistant for "Matrix - Trading Matematico."
* **Language:** Responds in Italian by default.
* **Knowledge Source:** Primarily Notion database, supplemented by extensive pre-loaded FAQs and project info within the prompt.
* **Task:** Answer questions, provide information, guide users through project details.
* **Restrictions:** Does not give financial advice; focuses on technical/statistical aspects. Initially instructed to only accept text (though workflow handles voice).

## Setup Considerations (for context)

To make this workflow operational, the following would be required:

* A configured n8n instance.
* Telegram Bot API token.
* OpenAI API key.
* Google Cloud Platform credentials with Gemini API enabled.
* Notion API integration and the ID of the "Matrix Knowledge Base" database.
* Google Sheets API credentials and the ID of the "Utenti Autorizzati" spreadsheet, correctly formatted with a sheet named "Utenti" and a column for user IDs.
* The detailed system prompt for the AI Agent.

## Workflow Screenshot
![Image](https://github.com/user-attachments/assets/53b1fbf8-cee3-41c8-aa5c-f0ac4df31c0b)
