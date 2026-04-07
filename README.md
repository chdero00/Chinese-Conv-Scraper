# DimDiaASQ-Dataset-Pipeline
A two-stage data processing pipeline for constructing the Traditional Chinese DimDiaASQ (Dimensional Dialogue Aspect-based Sentiment Quadruple) dataset from YouTube comments.

# DimDiaASQ Dataset Construction Pipeline

## Project Title
Constructing the Traditional Chinese DimDiaASQ (Dimensional Dialogue Aspect-based Sentiment Quadruple) Dataset and Prediction Model

## Research Objective
This project aims to build a novel Chinese dialogue dataset by integrating the concepts of DiaASQ (Dialogue Aspect-based Sentiment Quadruple) and DimABSA (Dimensional Aspect-based Sentiment Analysis). The primary task is to predict the sentiment quadruple `(target, aspect, opinion, intensity)` from multi-turn dialogues. Unlike traditional polarity classification (positive/negative/neutral), `intensity` here consists of continuous numerical values representing `Valence` and `Arousal`. The final prediction target is formatted as: `(target, aspect, opinion, valence#arousal)`.

## Current Research Setup
1.  **Data Source:** YouTube comment sections (fetched via YouTube API v3).
2.  **Domains:** Laptops, Mobile Phones, Hotels, and Restaurants (currently prioritizing Traditional Chinese review/unboxing videos).
3.  **Valence & Arousal Calculation:** The supervising professor has provided an automated tool for calculating Valence and Arousal values. Therefore, manual annotation of VA values is not required. The research focuses on accurately extracting Targets (entities), Aspects, and Opinions from dialogues, which are then processed by the professor's tool to generate VA labels.

## Current Engineering Progress: Dataset Construction Pipeline
A complete "two-stage" data preprocessing pipeline has been implemented in Python, successfully converting flat YouTube comments into multi-turn Dialogue Trees that meet strict DiaASQ standards.

### Stage 1: Loose Scraping
* **Method:** Automatically fetching comments from videos in specific domain playlists via the YouTube API.
* **Filtering:** Retaining only comment threads with `totalReplyCount >= 4` (meaning the entire thread has at least 5 utterances).
* **Output:** Flattened JSON data preserving the original `text` and `@username`, with `reply_to` defaulting to the top-level comment (`utt_id: 0`). A `domain` label is automatically added for cross-domain evaluation.

### Stage 2: Dialogue Topology Reconstruction & Denoising
* **Entity Alignment & Reply Relationship Reconstruction:** Utilizing dynamic string matching algorithms to scan for `@username` within the `text`, recursively searching backward for previous speakers to correctly map the `reply_to` of child comments to their corresponding `utt_id`. This successfully reconstructs the $n \rightarrow m$ dialogue tree structure (Utterance-to-Utterance).
* **Text Denoising & Span Alignment Safeguards:**
    * Precisely removing `@username` to prevent interference with the attention mechanisms of future Pre-trained Language Models (PLMs).
    * Retaining Emojis, as they contain strong Valence and Arousal features.
    * Handling newline characters: Using regular expressions to replace all continuous `\r` and `\n` with a single full-width comma `，`, and compressing redundant spaces. This completely resolves potential character misalignment issues when annotating T-A-O span indices (Start/End Index) in the future.
* **Topology Filtering (Aligning with DiaASQ standards):**
    * Using DFS (Depth-First Search) algorithms to calculate dialogue tree depth and leaf node count.
    * Strict filtering criteria: Total nodes limited to `5 ~ 12`, maximum tree depth of `3 ~ 5` levels, and branches (leaf nodes) $\ge$ `2`. Low-quality or one-way dialogues that do not meet these conditions are discarded.
