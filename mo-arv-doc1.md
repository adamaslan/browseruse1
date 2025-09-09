# arXiv Research Agent Codebase Overview

This document provides a high-level overview of the key aspects of the arXiv research agent codebase, which fetches and analyzes academic papers related to momentum trading and investing.

## 1. Setup and Imports

The script starts by importing necessary libraries such as `asyncio`, `json`, `feedparser`, and `datetime`. It loads environment variables and initializes a language model from Google's Gemini-2.5-flash.

## 2. Search Queries

The script defines a list of search queries for fetching papers related to momentum trading and investing. These queries include terms like "momentum trading," "momentum investing," and "momentum transformer."

## 3. File Paths

The script specifies file paths for storing visited papers, markdown summaries, and structured data. This includes files like `visited_arxiv.json`, `arxiv_momentum_summary.md`, and `arxiv_momentum_data.json`.

## 4. Helper Functions

The script includes several helper functions to manage data and files:

- **Load Visited Papers**: Loads a set of already visited paper IDs from a JSON file.
- **Save Visited Papers**: Saves the set of visited paper IDs to a JSON file.
- **Append Markdown**: Appends new text to a markdown file.
- **Append Structured**: Appends new structured data to a JSON file.

## 5. Fetching Papers

The `fetch_arxiv` function fetches papers from arXiv based on a query. It handles URL encoding and error handling, ensuring robust data retrieval. It uses retries and exponential backoff for better reliability.

## 6. Agent Setup

The `make_agent` function creates an agent with a specific task prompt to analyze the papers. The task prompt outlines the format for summarizing each paper, including key findings, implications for investors, and practical recommendations.

## 7. Progress Tracking

The `log_progress` function logs progress messages with timestamps. This helps in tracking the execution and debugging issues.

## 8. Main Run Function

The `run_research` function is the main function that orchestrates the fetching and processing of papers. It handles batching, error handling, and saving results. Key steps include:

1. Fetching papers from all queries.
2. Processing papers in batches.
3. Saving markdown summaries and structured data.
4. Updating the list of visited papers.
5. Pausing between batches to avoid rate limiting.

## 9. Test Function

The `test_arxiv_fetch` function tests the arXiv fetch functionality to ensure it works correctly. It checks if papers can be fetched and displays information about the first paper.

## 10. Manual Execution

The script includes a section for manual execution, allowing users to run the script directly. It tests the arXiv connection and provides instructions for running the full research process.

## 11. Async/Await

The script uses `asyncio` for asynchronous operations, particularly in the `run_research` function. This allows for efficient handling of multiple tasks concurrently.

## 12. Error Handling

The script includes robust error handling and retries for fetching papers and processing them with the agent. This ensures that the script can recover from transient failures and continue processing.
