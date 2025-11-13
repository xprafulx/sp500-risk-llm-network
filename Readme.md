1. Introduction

This is a quick assignment to try out some of the methods we worked on in this module. Our goal is to experiment with the pipeline and build a foundation for our discussion during the exam. We analyze S&P 500 earnings call transcripts to explore key topics discussed by companies using an LLM-based pipeline.

***Objectives***

We use an LLM combined with network analysis on a real text corpus. Our objectives are as follows:
1. Extract structured information from text using an LLM.
2. Explore it with basic descriptive statistics.
3. Build a knowledge graph or network from the extracted information.
4. Analyze the resulting network to uncover interesting patterns.

2. Data Loading and Sampling
We used the S&P 500 Earnings Call Transcripts dataset from Hugging Face (Kurry, 2024). After filtering the dataset to the years 2023–2025, we selected only transcripts with valid company_name and company_id values to avoid missing-value issues. From this complete subset, we randomly sampled 500 transcripts to keep the LLM extraction process manageable under API rate limits.

3. Data Processing
To prepare the transcripts for LLM extraction, we performed a simple text-cleaning step. The raw content field includes speaker labels such as “Operator:”, “Analyst:”, and “Executive:”, which add noise and are not part of the actual spoken text. We removed these labels and collapsed extra whitespace to create a cleaner version of each transcript.
This resulted in a new column, clean_content, containing readable text without speaker tags or formatting artifacts. This cleaned text was used as input for the LLM extraction step to ensure more accurate and consistent outputs.

4. LLM-Based Risk Extraction
To extract structured risk information from each earnings call, we used a Large Language Model (Gemini 2.5 Flash Lite) together with a strict JSON schema. The goal was to identify the dominant risk mentioned in each transcript, using a standardized set of risk categories.
Schema
We defined a Pydantic model, EarningsRiskExtraction, which specifies the structure of the extracted output:
company – name of the company
year – transcript year
risk_type – one short label selected from a fixed list
risk_summary – 1–2 sentence description of the risk
confidence – the model’s self-reported certainty (0–1)
This schema ensures that all extractions follow a consistent format and are easy to validate.
Prompt Design
We instructed the model to:
extract only risk-related information
select one dominant risk even if several are mentioned
use one short standardized label from a fixed list of eight risk types
avoid inventing new labels
return JSON only, matching the schema exactly
This controlled prompting helps keep risk categories consistent and avoids unnecessary variation.
Batch Processing (Handling Rate Limits)
The Gemini API allows only 15 requests per minute, so we implemented a batching system:
process 15 transcripts at a time
wait 65 seconds between batches
continue until all 60 transcripts are processed
This approach avoids rate-limit errors while still allowing us to extract all required data.
Output
For each transcript, the model returned:
one risk category
a short summary of what the company said about that risk
a confidence score
These results were combined into a final DataFrame, df_risks, which serves as the basis for descriptive analysis and network construction.

5. Descriptive Exploration
After extracting risk-related information using the LLM, we performed a simple descriptive exploration to understand the patterns in our structured dataset.
📌 Overview of Extracted Data
Number of transcripts processed: 60
Number of unique companies: 60
Number of standardized risk types: 7
Fields extracted:
company, year, risk_type, risk_summary, confidence

📊 Frequency of Risk Types
We computed the frequency of each extracted risk type and visualized it as a bar chart.
The most common risks in our sample were:
Macroeconomic (most frequent)
Market Conditions
Supply Chain
Policy Risk
Macroeconomic risks dominate because many earnings calls refer to inflation, consumer uncertainty, interest rate pressures, and broader market trends that affect almost all companies.

🧪 Informal Precision and Recall
To assess extraction quality, we manually reviewed a subset of results.
Precision (High):
Every extracted risk_type matched a real risk present in the transcript.
The model did not invent new risks, and summaries were accurate at a high level.
Recall (Low):
Transcripts often contained multiple risks, but the model was instructed to return only one dominant risk.
As a result, secondary risks were intentionally not captured.
This behavior is expected given our prompt design.

⚠️ Limitations and Systematic Errors
1. The model outputs only one main risk even when multiple risks are discussed.
2. Risk labels are mostly consistent, but the chosen category can shift depending on sentence phrasing.
3. Some summaries are brief and may overlook important nuance.
4. The model often selects broad categories (e.g., Macroeconomic) instead of more specific risks.
5. Only 60 transcripts were analyzed, so results may not represent the full dataset of ~33,000 transcripts.


6. Knowledge Graph / Network Construction
We built a bipartite knowledge graph linking companies to the dominant risk type extracted from their earnings calls. Each company and each risk type is represented as a node, and edges connect them when a risk is mentioned. The graph also stores useful metadata such as year, confidence, and a short summary. A bipartite setup was chosen because companies only connect to risks, not to each other, making the structure simple and interpretable. This graph helps us see which risks are most widespread across firms and provides the basis for our centrality analysis and insights.

7. Network Analysis
We analyzed the bipartite company–risk graph using degree centrality to understand which risks affect the most companies. The results show that Macroeconomic risk dominates with 27 connected companies, followed by Market Conditions (10) and Supply Chain (9). This suggests that broad economic uncertainty is the most common concern across firms. On the company side, most firms report only one dominant risk, but a few—like C.H. Robinson Worldwide and Zoetis Inc.—connect to multiple risks. We also generated a small risk–risk co-occurrence network, where only three risks overlapped across companies, indicating that most risks occur independently rather than together. Overall, the network structure highlights the concentration of risk exposure and the central role of macroeconomic conditions in shaping corporate narratives.