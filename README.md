# Project_Initiation_DS5500

"E.S.G. stands for environmental, social, and governance, and E.S.G. investing is about allocating resources in a way that makes money while also improving the world." 

https://freakonomics.com/podcast/are-e-s-g-investors-actually-helping-the-environment/

(ESG)-driven portfolios are already a large and growing portion of global assets under management (35 Trillion).  And that’s expected to grow to about one-third of all assets under management by 2025 (Freakonomics). 

Concerns revolve around ongoing challenges regarding the lack of standardization in disclosure standards, the metrics used to measure progress, and the units of measure for specific ESG issues. 


## Project Objective


Form 8-K, indeed known as a "current report," is a  document required by the U.S. Securities and Exchange Commission (SEC). Companies publicly traded in the United States are mandated to file this form to announce significant events that shareholders should be aware of. The purpose of Form 8-K is to provide the public with an up-to-date overview of material changes in a company's operations or management that might affect its financial health or stock value.

The objective of this project is to evaluate setiment around ESG topics using company 8-K SEC disclosures (2013-2023). 


## Examples of Brown Frims:

Duke Energy, Southwest Airlines, Tyson Foods, DuPont, FedEx, NewMarket, Marathon Petroleum

## Example of Green Firms:

Spotify, Prudential, Goldman Sachs, Allstate, MetLife, American Express

## Probelm Statement

Identify the Problem:

Ongoing challenges regarding the lack of standardization in disclosure standards, the metrics used to measure progress, and the units of measure for specific ESG issues

 
Put the Problem into Context:

Companies and investors measure different things and report in different ways. It gets confusing, especially when investors try and de-code whether their money has made a difference or not.


## Project Road Map

ESG Sentiment Analysis Project Using 8-K Filings

#### 1. Compile List of 8-K Financial Disclosures

Time Frame: 2013 - 2023 

Data Source: Utilize the SEC's EDGAR system for 8-K filings access

Refrence:
1. "Measuring Disclosure Using 8-K Filings"


https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3354252


* VDisc_Ct is the number of voluntary 8K items and associated exhibits
* VDisc_WC is the number ofwords within the voluntary 8K items and associated exhibits

2. "ESG In Corporate Filings: An AI Perspective"

Tie the corporate actions on the environment to the investor expectations.

https://arxiv.org/pdf/2212.00018.pdf
 
Used:
 * keywords corresponding to the SASB categories of ESG terms
   
Among the key findings are: 
 * lack of transparency among the ESG raters on how the scores are assigned;
 * lack of standards on how a particular concept is measured 
 * Questionable tradeoffs: high scores in one domain may offset very low scores in another area 
 * Absence of an overall score combining performance scores alongenvironmental, social and governance axes
 * lack of acknowledgement of stakeholder expectations leading to lower acceptance rates.
   

3. Capturing Firm Economic Events

https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4510212#:~:text=To%20better%20capture%20events%20that,participants'%20uncertainty%20about%20that%20performance.


* event-driven 8-K, Items (all 8-K items except Items 2.02, 7.01, 8.01, and 9.01)
* disclosure-driven, Items 2.02, 7.01, and 8.01 constitute 8-K items with a voluntary disclosure component 
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
