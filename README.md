## 1. Overview

The Maya Chatbot Backend is a Python FastAPI application providing robust services for a multilingual, therapeutic AI chatbot. It features a multi-agent architecture, integrating OpenAI (GPT-4.1, Whisper) and ElevenLabs APIs, with Rhubarb Lip Sync for avatar animation. The system prioritizes modularity, asynchronous processing, and carefully engineered AI agent interactions.

## 2. Core Architecture & Technologies

*   **Framework:** FastAPI (Async)
*   **AI Services:**
    *   **OpenAI GPT-4.1:** Powers distinct agents for (1) Conversational Logic (Maya Persona) and (2) User Persona Management (structured JSON updates).
    *   **OpenAI GPT-4o-mini-search-preview:** Drives a specialized, action-oriented Recommendation Agent with real-time web search capabilities.
    *   **OpenAI Whisper:** Speech-to-Text (STT).
    *   **ElevenLabs API:** Text-to-Speech (TTS) with language-specific voices.
*   **Lip Sync:** Rhubarb Lip Sync (subprocess) for viseme generation from TTS audio.
*   **Data Management:**
    *   **Active Session:** `userpersona.json` (dynamically updated JSON object).
    *   **Archival:** User persona data from completed sessions is archived to a **database**.
    *   **Context:** In-memory lists for short-term conversation history.
*   **Configuration:** `.env` for API keys; `LANGUAGE_MAPPING` for language/voice settings.
*   **Geocoding:** `geopy` for location context in recommendations.

## 3. Key Agentic Flows

The system employs distinct **agentic flows**, where specialized AI agents, guided by engineered prompts and guardrails, perform specific tasks:

*   **A. Main Interaction Flow (`POST /api/transcribe`):**
    1.  **Input:** User audio (WAV).
    2.  **STT Agent (Whisper):** Audio -> Transcribed Text.
    3.  **Parallel Processing:**
        *   **Conversational Agent (GPT-4.1 "Maya Persona"):** Manages the dialogue flow. Transcribed Text + Chat History -> Bot Response (text, expression, animation cues). *Prompt engineering defines persona, empathetic engagement, conversational goals, and style guardrails.*
        *   **Persona Update Agent (GPT-4.1 "Profile Manager"):** Transcribed Text + Current Persona -> Updated `userpersona.json`. *Prompt engineering ensures structured, additive updates in English and protects system fields.*
    4.  **TTS Agent (ElevenLabs) & Lip Sync Processor (Rhubarb):** Bot Text -> Audio (WAV) -> Lip Sync JSON.
    5.  **Output:** Structured JSON (bot messages, base64 audio, lip-sync data) for frontend rendering.

*   **B. Recommendation Agent Flow (`POST /api/recommendation`):**
    1.  **Input:** Current `userpersona.json` (skills, location, needs).
    2.  **Location Processing (Geopy):** Lat/Long -> City/State.
    3.  **Recommendation Agent (GPT-4.1-mini):** This dedicated agent **takes action** by performing a live web search based on the user's profile and the implicit or explicit need for information. *Its prompt engineering focuses on effective search strategies, synthesizing relevant information, and formatting actionable results.* This flow operates with a higher degree of agency in information retrieval.

## 4. API Endpoints Summary

*   **`/api/transcribe` (POST):** Core conversational interaction.
*   **`/api/recommendation` (POST):** Invokes the Recommendation Agent for web-searched information.
*   **`/api/set-language` (POST):** Configures session language and TTS voice.
*   **`/api/get-user-persona` (GET):** Retrieves current user profile.
*   **`/api/update-user-persona` (POST):** Allows direct persona data modification.
*   **`/api/end-chat` (POST):** Archives persona to database, clears session data.

## 5. Technical Highlights

*   **Modular AI Agents:** Specialization of AI models (conversational vs. action-oriented recommendation) enhances control and task-specific performance.
*   **Prompt Engineering:** Critical for defining agent behavior, capabilities, output consistency, and implementing guardrails for each distinct agent.
*   **Orchestrated Agency:** The system distinctly separates conversational management from the more proactive, information-retrieval agency of the recommendation component.
*   **Asynchronous Operations:** Maximizes I/O efficiency and responsiveness.
*   **Structured Data Exchange:** JSON for API communication and persona management.
*   **Frontend Decoupling:** Backend provides data; frontend handles rendering, audio playback, and animation.
