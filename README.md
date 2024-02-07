# Project_Initiation_DS5500


## Project overview 

The integration of Environmental, Social, and Governance (ESG) metrics into investment strategies signifies a notable shift towards sustainable financial practices, expected to influence a substantial portion of global asset management by 2025. This study aims to analyze corporate 8-K filings from 2013 to 2023, distinguishing between 'Brown' firms (top 20 percent in greenhouse gas emissions per unit of sales) and 'Green' firms (bottom 20 percent).

This study assesses the evolution of ESG messaging within 8-K filings among 'brown' and 'green' firms, categorized based on their greenhouse gas emissions. It investigates the voluntary disclosure practices within these filings to understand their role in addressing or potentially facilitating greenwashing. The objective is to identify trends and strategies in ESG communication among companies with differing environmental impacts.

## Background

ESG metrics encompass three fundamental pillars: 

1. The "E" dimension evaluates a company's management of natural resources and its environmental impact, encompassing both direct operations and supply chain activities.

2. The "S" aspect pertains to social factors, assessing a company's effectiveness in navigating social trends, labor practices, and political dynamics.

3. The "G" component of ESG focuses on governance factors, examining decision-making processes from governmental policy-making to the allocation of rights and responsibilities within corporations. This includes scrutiny of governance structures involving the board of directors, managers, shareholders, and stakeholders


## The Landscape

ESG ratings play a crucial role in guiding trillions of dollars in investments worldwide. For example, the investment strategy of the Maine Public Employees Retirement System Pension Fund is shaped by ESG criteria, highlighting the significance of integrating sustainability factors for long-term investment success.

Maine is among six states with established ESG investment guidelines and policies, reflecting a growing recognition of the importance of sustainability considerations in investment decisions.

However, the ESG movement finds itself embroiled in political debates. While political progressives view it as a means to address environmental challenges and safeguard the planet, Republican lawmakers often dismiss it as "woke investing" and seek to restrict its adoption through legislative measures.






## The Challenges 


1. Lack of transparency among the ESG raters on how the scores are assigned.
2. Lack of standards on how a particular concept is measured.
3. Questionable tradeoffs: high scores in one domain may offset very low scores in another area.
4. Absence of an overall score combining performance scores along Environmental, Social, and Governance (ESG) axes; weights given to the "E," "S," and "G."
5. Lack of acknowledgement of stakeholder expectations.



## Exploring the ESG Movement through Corporate 8-K Filings

Form 8-K, commonly referred to as a "current report," is a mandatory filing prescribed by the U.S. Securities and Exchange Commission (SEC) for publicly traded companies in the United States. Its purpose is to promptly notify shareholders of significant events that could impact the company's financial well-being or stock value. By offering a real-time overview of material changes in a company's operations or management, Form 8-K serves as a  tool for investors to stay informed about developments that may affect their investment decisions.


Rationale for Using 8-K Filings:

The choice of 8-K filings as the primary source of analysis is informed by its unique capacity to  view of how companies disclose  events and strategies. These filings are categorized into two main types: Voluntary and Event-Driven disclosures. By analyzing both categories, this research aims to gain a nuanced understanding of the motivations and practices surrounding ESG disclosures within the corporate landscape.



## Examples of Brown Frims:

Duke Energy, Southwest Airlines, Tyson Foods, DuPont, FedEx, NewMarket, Marathon Petroleum

## Example of Green Firms:

Spotify, Prudential, Goldman Sachs, Allstate, MetLife, American Express

## Probelm Statement
 
This study evaluates ESG messaging evolution in 8-k filings among 'brown' and 'green' firms, categorized by their greenhouse gas emissions—top and bottom 20 percent per unit of sales, respectively. It investigates voluntary disclosure practices within these filings to explore their role in addressing or facilitating greenwashing. The aim is to identify trends and strategies in ESG communication among high and low environmental impact companies.

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



## Related Projects
1. https://www.mdpi.com/2071-1050/12/13/5317
2. https://www.mdpi.com/2071-1050/14/14/8745
3. https://www.pm-research.com/content/pmrjesg/1/3/39
