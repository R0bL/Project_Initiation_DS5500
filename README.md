# Project_Initiation_DS5500

"E.S.G. stands for environmental, social, and governance, and E.S.G. investing is about allocating resources in a way that makes money while also improving the world." 

https://freakonomics.com/podcast/are-e-s-g-investors-actually-helping-the-environment/

(ESG)-driven portfolios are already a large and growing portion of global assets under management (35 Trillion).  And that’s expected to grow to about one-third of all assets under management by 2025 (Freakonomics). 

Concerns revolve around ongoing challenges regarding the lack of standardization in disclosure standards, the metrics used to measure progress, and the units of measure for specific ESG issues. 


## Project Objective


Form 8-K is known as a “current report” and it is the report that companies must file with the SEC to announce major events that shareholders should know about

The objective of this project is to evaluate setiment around ESG topics using company 8-K SEC disclosures (2013-2023). 


## Examples of Brown Frims:

Duke Energy, Southwest Airlines, Tyson Foods, DuPont, FedEx, NewMarket, Marathon Petroleum

## Example of Green Firms:

Spotify, Prudential, Goldman Sachs, Allstate, MetLife, American Express



## Project Road Map

ESG Sentiment Analysis Project Using 8-K Filings

#### 1. Compile List of 8-K Financial Disclosures

Time Frame: 2013 - 2023 

Data Source: Utilize the SEC's EDGAR system for 8-K filings access

Refrence:
1. "Measuring Disclosure Using 8-K Filings"

https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3354252

2. "ESG In Corporate Filings: An AI Perspective"

https://arxiv.org/pdf/2212.00018.pdf

3. Capturing Firm Economic Events

https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4510212#:~:text=To%20better%20capture%20events%20that,participants'%20uncertainty%20about%20that%20performance.

#### 2. Data Extraction and Preprocessing

Extract Relevant Sections: Identify and extract ESG-relevant sections from 8-K filings

Text Cleaning: Standardize formatting and remove non-essential elements.

Refrence: https://scholar.harvard.edu/jbenchimol/files/text-mining-methodologies.pdf

#### 3. Labeling Data for Sentiment Analysis

Develop a Labeling Guide: Define positive, negative, and neutral ESG sentiments.

Manual Labeling: Label a subset of filings for training the sentiment analysis model.

Refrence: https://aws.amazon.com/blogs/machine-learning/use-sec-text-for-ratings-classification-using-multimodal-ml-in-amazon-sagemaker-jumpstart/

#### 4. Feature Extraction

NLP Techniques: Apply TF-IDF, sentiment scores, and word embeddings for data conversion.

#### 5. Model Selection and Training

Sentiment Analysis Models: Choose from logistic regression, SVM, LSTM, or BERT models.


Model Training: Train the model using the manually labeled dataset.

#### 6. Model Evaluation and Optimization

Cross-Validation: Implement k-fold cross-validation for model performance.


Metrics: Use accuracy, precision, recall, F1 score, and confusion matrices.

#### 7. Analysis of ESG Sentiment Trends

Time-Series Analysis: Examine sentiment trends over time.


Comparative Analysis: Compare sentiments across industries or companies.

#### 8. Visualization and Reporting

Reporting: Document findings, methodology, and project implications 


## Source: 
1.  Hartzmark, Samuel M., and Kelly Shue. "Counterproductive Sustainable Investing: The Impact Elasticity of Brown and Green Firms." 1 Nov. 2022, SSRN, https://ssrn.com/abstract=4359282 or http://dx.doi.org/10.2139/ssrn.4359282.

2.  Dubner, Stephen J. “Are E.S.G. Investors Actually Helping the Environment?” Freakonomics Radio, no. 546, Freakonomics, LLC, 14 June 2023, https://freakonomics.com/podcast/are-e-s-g-investors-actually-helping-the-environment/.

3.  

NLP use for Stock Prediction. 


## Related Projects
1. https://www.mdpi.com/2071-1050/12/13/5317
2. https://www.mdpi.com/2071-1050/14/14/8745
3. https://www.pm-research.com/content/pmrjesg/1/3/39
