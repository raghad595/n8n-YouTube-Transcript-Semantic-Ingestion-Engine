# 📺 YouTube Transcript Semantic Ingestion Engine (Vector ETL Pipeline)

An automated Vector ETL pipeline built in n8n that monitors YouTube channels, extracts raw video caption files via the YouTube API, partitions text into contextual chunks, and generates vector embeddings to populate a Supabase vector database[cite: 4]. 

This workflow serves as the foundational data ingestion layer for a Retrieval-Augmented Generation (RAG) ecosystem, enabling downstream AI agents (such as Telegram bots) to perform real-time semantic search across video catalogs.

## 🚀 Business Value & Motivation
Video content contains immense amounts of knowledge, but it is traditionally opaque to text-based search engines and AI models. Turning hours of video footage into a searchable, AI-ready knowledge base requires converting spoken text into mathematical vectors. This engine automates that entire lifecycle. By chunking text alongside dynamic, precision time-linked deep links, it enables users to ask an AI agent a question and immediately receive the exact second in a video where that answer is spoken.

## 🏗️ Core Architecture & Data Flow

The graph operates as an advanced data-processing pipeline split into three core phases:

### 1. Extraction (YouTube API Ingestion)
*   **Video Discovery:** Tracks and pulls recent video metadata from a target YouTube architecture channel using the **YouTube node** (`Get many videos`)[cite: 4].
*   **Caption Retrieval:** Executes two consecutive **HTTP Request nodes** targeting the Google YouTube v3 API endpoints (`/captions`)[cite: 4]. The first fetches available caption tracks for the specific `videoId`, and the second downloads the raw, timestamped transcript file payload[cite: 4].
*   **File Parsing:** Passes the raw binary download payload to an **Extract from File node** to isolate the unformatted text block[cite: 4].

### 2. Transformation (Deterministic Custom Chunking)
To prevent context loss and ensure precise search results, the raw text passes into a custom **JavaScript Code block** executing an optimized chunking strategy[cite: 4]:
*   **Window Management:** Breaks down raw timestamp blocks into clean text segments based on a strict `CHUNK_SIZE` limit of 500 characters[cite: 4].
*   **Timestamp-to-Seconds Translation:** Executes an algorithmic translation function (`timeToSeconds`) that converts standard subtitle timeline formats (`HH:MM:SS`) into flat integer seconds[cite: 4].
*   **Deep-Link Engineering:** Dynamically pairs each 500-character chunk with its precise starting time query parameter, injecting a unique, functional URL format directly into the chunk's metadata tracking array:  
    `https://www.youtube.com/watch?v=${videoId}&t=${seconds}`[cite: 4]

### 3. Loading (Vector Storage Insertion)
*   **Item Tokenization:** A **Split Out node** normalizes the compiled JavaScript chunk array into standalone n8n data records[cite: 4].
*   **Semantic Vectorization:** Feeds textual data rows into the **Embeddings Google Gemini node** to compute high-density embedding representations[cite: 4].
*   **Vector Upsertion:** Streamlines the resulting vectors along with the custom time-stamped URLs into the `documents` table of a **Supabase Vector Store**[cite: 4].

## 🚀 How to Import and Run

### 1. Prerequisites
*   An operational [n8n](https://n8n.io/) canvas workspace.
*   Google Developer Console account with **YouTube Data API v3** access enabled.
*   An active **Supabase** instance configured with pgvector extension enabled on your target database[cite: 4].
*   A **Google Gemini** API key[cite: 4].

### 2. Setup & Configuration
1. Clone or copy the `youtube_vector_pipeline.json` payload from this folder.
2. In your n8n interface, start a blank workflow, click the top-right options grid, and select **Import from File**.
3. Supply your verified workspace secrets/keys to the **YouTube account**, **Google Gemini Api**, and **Supabase account** credential slots[cite: 4].
4. Run the manual trigger to backfill historical catalogs, or replace the initial node with a cron/poll script to monitor channels continuously[cite: 4].

## 🛠️ Tech Stack
*   **Workflow Engine:** n8n[cite: 4]
*   **Vector Database:** Supabase (PostgreSQL + pgvector)[cite: 4]
*   **Embedding Generator:** Google Gemini Text Embeddings API[cite: 4]
*   **External APIs Integration:** Google Cloud YouTube v3 Engine[cite: 4]
*   **Data Orchestration Library:** LangChain (n8n Integration Ecosystem)[cite: 4]
