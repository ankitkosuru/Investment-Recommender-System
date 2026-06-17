# Investment Recommender System

## Overview

This project is an AI-powered investment recommender system that answers natural language questions about company financials. Instead of manually looking up balance sheets, a user can simply ask "Should I invest in AAPL?" and the system retrieves real financial data and generates a meaningful, context-aware response.

The system combines real-time financial data fetching, a fine-tuned large language model, vector-based semantic search, and a graph database into a single end-to-end pipeline.


## How It Works

The system follows a retrieval-augmented generation approach. When a user asks a question, the system first checks the Neo4j graph database for structured financial data. If the data exists there, it returns the answer directly. If not, it falls back to FAISS semantic search to find relevant financial documents and passes them to LLaMA to generate a natural language respo

## Technologies Used

**yfinance** was used to fetch real balance sheet data directly from Yahoo Finance for over 100 publicly listed companies. It requires no API key and provides free access to financial statements.

**LLaMA-2 (NousResearch/LLaMA-2-7b-hf)** is the large language model used for generating natural language responses. It was fine-tuned on a custom financial question-answer dataset to improve its accuracy and response style for financial queries.

**HuggingFace Transformers** was used to load, tokenize, and fine-tune the LLaMA-2 model. It simplifies working with large pretrained models significantly.

**HuggingFace Accelerate** was used to handle memory-efficient training. Since LLaMA-2 with 7 billion parameters requires far more GPU memory than typically available, Accelerate offloads optimizer states to CPU RAM using ZeRO offloading, making training feasible on standard hardware.

**FAISS (Facebook AI Similarity Search)** was used as the vector database. All 300 financial question-answer pairs were converted into vector embeddings and stored in FAISS. When a user asks a question, the system finds the most semantically similar documents using vector similarity search.

**Neo4j Aura** was used as the graph database to store company financial data as a network of connected nodes. Each company node is connected to its financial data node through a relationship called HAS_FINANCIAL_DATA. This allows fast, direct lookup of structured financial information without complex joins.

**py2neo** was used as the Python client to connect to and interact with Neo4j from the code.



## Dataset

Financial data was collected for 100 randomly selected companies from a predefined list spanning technology, banking, healthcare, retail, automotive, and entertainment sectors. For each company, three financial metrics were extracted from the most recent balance sheet: Total Assets, Total Liabilities, and Total Stockholder Equity.

This raw data was converted into 300 natural language question-answer pairs, which were used both for fine-tuning LLaMA-2 and for populating the FAISS vector index.



## Project Structure

The pipeline runs in the following order:

Step 1 - Fetch financial data from Yahoo Finance for 100 companies and save it as a JSON file.

Step 2 - Store all company financial data into Neo4j as a graph with Company nodes connected to Financials nodes.

Step 3 - Convert raw financial data into 300 question-answer pairs and save as a JSON dataset.

Step 4 - Fine-tune LLaMA-2 on the question-answer dataset for 5 epochs using HuggingFace Transformers and Accelerate.

Step 5 - Build a FAISS vector index by embedding all 300 question-answer documents.

Step 6 - Run the full pipeline where user queries are answered using Neo4j or FAISS plus LLaMA.

## Neo4j Graph Structure

The graph database stores two types of nodes connected by one relationship type.

Company node stores the ticker symbol as its property, for example name: "AAPL".

Financials node stores three properties: total_assets, total_liabilities, and shareholder_equity.

The relationship HAS_FINANCIAL_DATA connects a Company node to its corresponding Financials node, showing direction from the company to its financial data.



## Fine-Tuning Details

Model: NousResearch/LLaMA-2-7b-hf
Training examples: 300 question-answer pairs
Epochs: 5
Optimizer: AdamW with learning rate 5e-5
Precision: float16 to reduce GPU memory usage
Memory optimization: HuggingFace Accelerate with ZeRO offloading



## Evaluation

The system was evaluated using BLEU score by comparing LLaMA's generated responses against ground truth financial data fetched live from Yahoo Finance. Ground truth was fetched fresh at evaluation time to ensure the reference values were accurate and current.



## Limitations

The ticker extraction from user queries is currently basic and works best when the ticker symbol appears at the end of the question. The BLEU score metric measures word overlap and is not ideal for evaluating numerical financial accuracy. The Neo4j credentials are currently hardcoded and should be moved to environment variables before any production deployment. The setup_faiss function needs to be completed with an explicit embedding model definition.



## Future Improvements

Replace basic ticker extraction with a named entity recognition model to handle natural query formats. Use numerical accuracy metrics instead of BLEU score for financial evaluation. Move all credentials to a .env file and add it to .gitignore. Build a simple web interface using Streamlit so non-technical users can interact with the system easily. Add support for income statement and cash flow data in addition to balance sheet metrics.



B.Tech, Mahatma Gandhi Institute of Technology, 2025
ankitkosuru03@gmail.com
www.linkedin.com/in/ankitkosuru
