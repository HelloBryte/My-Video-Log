# Xero Research Assistant Demo

[![Watch the video](http://img.youtube.com/vi/LGg1s5CAIxE/0.jpg)](https://www.youtube.com/watch?v=LGg1s5CAIxE)

A demonstration of a TypeScript web application that answers questions about Xero using only public web pages it has collected, and shows the evidence behind every answer.

## Project Overview

The application gathers a small set of public pages about Xero, stores the extracted passages locally, and answers questions with a real language model. It retrieves only the passages relevant to each question, and checks every claim in the model's answer against the passage it cites before showing it. When the stored evidence cannot answer a question, it says so instead of guessing.

## What the Video Shows

- **Gathering research**: five public pages fetched, extracted and stored with their retrieval times
- **Asking a question**: a model-generated answer with its supporting sources and claim-by-claim checks
- **Tracing evidence**: each citation opens to the source URL, retrieval time, region and the original passage
- **Reuse**: a follow-up question and a second gather that fetch nothing again
- **Refresh**: re-fetching every page, with unchanged pages not reprocessed

## Key Features

- **Evidence Retrieval**: In-process BM25 search selects the relevant passages for each question
- **Citation Checks**: Every figure in an answer must appear in the passage it cites; unverified claims are flagged
- **Research Reuse**: Stored sources are never refetched to answer a question
- **Safe Failure Handling**: A failed page fetch keeps older evidence and marks it; a failed model call produces no answer
- **Activity Log**: Shows what was fetched, reused or reprocessed, and when the model was called

## Technical Highlights

- **Stack**: TypeScript, Node.js, Express, Vitest
- **Model**: DeepSeek through an OpenAI-compatible API
- **Respectful Fetching**: Honours robots.txt, per-host rate limits and timeouts
- **Verification**: Automated tests, a real-model evaluation, held-out questions, and a fact check against the live pages
