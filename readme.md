# 🦅 Hawky.ai — Video Understanding & Performance Prediction

> **Thesis:** We need to build a pipeline where:
>
> 1. Download the videos and metrics like views, shares, likes etc.
> 2. Divide the video into 1 frame per second — so a 20 second video gives 20 frames.
> 3. Use one vision model (like Gemini Flash Vision), one audio transcription model (like Whisper from OpenAI), and a vector embedding model (CLIP) to identify what each frame contains in vector space.
> 4. Build a ranking model using LightGBM — we use this because we have mixed data like scores (views, shares) along with vector embeddings.
> 5. Use SHAP to explain which creative features (hook type, face, music tempo) drove the ranking score for each video .

---

## System Architecture

```mermaid
flowchart TD
    A["Instagram Feed - Videos + Metrics"] --> B["Ingestion - Meta Graph API / Apify"]
    B --> C["Object Storage - S3 / Local"]
    C --> D1["Frame Sampler - ffmpeg at 1fps"]

    D1 --> D2["Gemini 2.0 Flash - Scene, Hook, Emotion, Text"]
    D1 --> D3["Whisper Small - Transcript + Caption"]
    D1 --> D4["CLIP Embeddings - Visual style vectors, PCA 512 to 10 dims"]

    D2 & D3 & D4 --> E["Feature Store - SQLite - structured tags + style dims"]

    F["Performance Metrics DB - views, watch_time, CTR, saves"] --> G

    subgraph G["Model Layer"]
        G1["Composite Score - 0.35 x views + 0.25 x watch_time + ..."]
        G2["LightGBM Regressor - Gemini + CLIP + Whisper to score"]
        G3["SHAP Explainer - which features drive performance"]
        G1 --> G2 --> G3
    end

    E --> G
    G --> H["FastAPI - /predict, /rank, /explain"]
    H --> I["Ranked Videos + Why explanations"]
```

---

## Models Used

| Role | Model | Provider | Est. Cost |
|------|-------|----------|-----------|
| Video understanding (scene, hook, emotion, text) | **Gemini 2.0 Flash** | OpenRouter | ~$0.003–0.008 / video |
| Audio transcription | **Whisper Small** | Replicate | ~$0.005 / video |
| Visual style embeddings | **CLIP ViT-L/14** | Replicate | ~$0.002 / video |
| Performance ranking | **LightGBM** | Local (no API) | Free |
| Explainability | **SHAP** | Local (no API) | Free |

> **Total API cost ≈ $1.00–2.00 per 100 videos**

---
