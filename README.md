# Project overview 

The adoption of Environmental, Social, and Governance (ESG) metrics in investing represents a significant shift from the tradtional metric to evaluate a companies performance. ESG funds represent about $35 trillion globally invested and is projected to influence a third of global asset managed by 2025. This research aims to dissect the ESG landscape by analyzing corporate 8-K filings from 2013 to 2023, distinguishing between between 'Brown' firms—those in the top 20 percent in terms of greenhouse gas emissions per unit of sales—and 'Green' firms, positioned in the bottom 20 percent 

8-K filings can act as a proxy for measuring firm-level voluntary disclosures (He & Plumlee, 2020). Variations in corporate communications patterns may serve as indicators that explain variation in ESG ratings (Schimanski et al., 2024)

## Examples of Brown Frims:

Duke Energy, Southwest Airlines, Tyson Foods, DuPont, FedEx, NewMarket, Marathon Petroleum

## Example of Green Firms:

Spotify, Prudential, Goldman Sachs, Allstate, MetLife, American Express


## Background

ESG metrics encompass three fundamental pillars: 

1. The "E" dimension evaluates a company's management of natural resources and its environmental impact, encompassing both direct operations and supply chain activities.

2. The "S" aspect pertains to social factors, assessing a company's effectiveness in navigating social trends, labor practices, and political dynamics.

3. The "G" component of ESG focuses on governance factors, examining decision-making processes from governmental policy-making to the allocation of rights and responsibilities within corporations. This includes scrutiny of governance structures involving the board of directors, managers, shareholders, and stakeholders


## The Landscape

ESG ratings play a role in guiding trillions of dollars in investments worldwide. For example, the investment strategy of the Maine Public Employees Retirement System Pension Fund is guided by ESG criteria, and they highlight the significance of integrating sustainability factors for long-term investment success.

Maine is among six states with established ESG investment guidelines and policies
.
However, the ESG movement finds itself embroiled in political discourse. While political progressives view it as a potential tool to address environmental challenges, Republican lawmakers critize it as "woke investing" and seek to restrict its adoption through legislative measures.


## The Challenges 


1. Lack of transparency among the ESG raters on how the scores are assigned.
2. Lack of standards on how a particular concept is measured.
3. Questionable tradeoffs: high scores in one domain may offset very low scores in another area.
4. Absence of an overall score combining performance scores along Environmental, Social, and Governance (ESG) axes; weights given to the "E," "S," and "G."
5. Lack of acknowledgement of stakeholder expectations.


# Exploring the ESG Movement through Corporate 8-K Filings

Form 8-K, commonly referred to as a "current report," is a mandatory filing prescribed by the U.S. Securities and Exchange Commission (SEC) for publicly traded companies in the United States. Its purpose is to promptly notify shareholders of significant events that could impact the company's financial well-being or stock value. By offering a real-time overview of material changes in a company's operations or management, Form 8-K serves as a  tool for investors to stay informed about developments that may affect their investment decisions.


### Rationale for Using 8-K Filings:

The choice of 8-K filings as the primary source of analysis is informed by its unique capacity to  view of how companies disclose  events and strategies. These filings are categorized into two main types: Voluntary and Event-Driven disclosures. By analyzing both categories, this research aims to gain a nuanced understanding of the motivations and practices surrounding ESG disclosures within the corporate landscape.


## Case Study

"ExxonMobil as a potential candidate to engage with, to try to drive value creation by making the business more sustainable? When you start looking at any potential activist target, you look for an outlier. And ExxonMobil was an outlier in that they had phenomenal assets, they had phenomenal engineering talent. Yet, they had underperformed, for nearly a decade, their peer group. This was a company who invented the lithium-ion battery, the company who was the first to actually scale solar production."


## Probelm Statement
 
This study evaluates ESG messaging evolution in 8-k filings among 'brown' and 'green' firms, categorized by their greenhouse gas emissions—top and bottom 20 percent per unit of sales, respectively. It investigates voluntary disclosure practices within these filings to explore their role in addressing or facilitating greenwashing. The aim is to identify trends and strategies in ESG communication among high and low environmental impact companies.


# Data Collection 

The data being evaluated are SEC filings of 1807 Equities held by the Norwegain Wealth Fund, downloaded from the SEC's Electronic Data Gathering, Analysis and Retrieval (EDGAR) website. 

Accessing the data via Amazon SageMaker using the  _smjsindustry_ see link below:

https://github.com/aws/sagemaker-jumpstart-industry-pack

_smjsindustry_ wraps the retrieval functionality into a SageMaker processing container to download a dataset of filings with metadata such as dates and parsed plaintext. The extracted dataframe is written to Amazon S3 storage as as CSV file. The API has three parts. 

1. Top part specifies:


* Tickers of SEC CIK codes for the companies forms are being retrieved
* SEC Form Type (8-K. 10-K, 10-Q)
* Date range of forms by filing date
* The output CSV file and S3 bucket to store the dataset

2. Middle part shows how to assign system resources and has default values in place

3. Last part runs the API


See example of the API call: 

```
%%time

### Top Part
dataset_config = EDGARDataSetConfig(
    tickers_or_ciks=['amzn', 'ntflx'],  # list of stock tickers or CIKs
    form_types=['10-K', '10-Q'],                              # list of SEC form types
    filing_date_start='2019-01-01',                  # starting filing date
    filing_date_end='2020-12-31',                    # ending filing date
    email_as_user_agent='test-user@test.com')        # user agent email

### Middle Part
data_loader = DataLoader(
    role=sagemaker.get_execution_role(),    # loading job execution role
    instance_count=1,                       # instances number, limit varies with instance type
    instance_type='ml.c5.2xlarge',          # instance type
    volume_size_in_gb=30,                   # size in GB of the EBS volume to use
    volume_kms_key=None,                    # KMS key for the processing volume
    output_kms_key=None,                    # KMS key ID for processing job outputs
    max_runtime_in_seconds=None,            # timeout in seconds. Default is 24 hours.
    sagemaker_session=sagemaker.Session(),  # session object
    tags=None)                              # a list of key-value pairs

### Bottom Part
data_loader.load(
    dataset_config,
    's3://{}/{}/{}'.format(bucket, sec_processed_folder, 'output'),    # output s3 prefix (both bucket and folder names are required)
    'dataset_8k.csv',                                                  # output file name
    wait=True,
    logs=True)
```


The output of the DataLoader processing job is a dataframe. The CSV file is downloaded from S3 and then read into a dataframe

![ML-5455-image001-1](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/99c99bf2-fa08-408b-a5ee-9529786ba186)


![download](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/dae97948-717a-43ed-b5a4-7f2e20cc05da)

```
client = boto3.client('s3')
client.download_file(bucket, '{}/{}/{}'.format(sec_processed_folder, 'output', 'dataset_10k.csv'), 'dataset_8k.csv')
data_frame_8k = pd.read_csv('dataset_10k.csv')
data_frame_8k
```

# Week 1 Report

### Project Updates

1. Created a AWS domain, user-acess, virutal space to pull SEC filings. My prefrence would be to bring the project out SageMaker. 

2. I am having some issues managing user execution roles, I am still fimilarizing myself with SageMaker but sometime I get a **403 error** when I pull using the API. Alternative: "OpenEDGAR: Open Source Software for SEC EDGAR Analysis" (https://law.mit.edu/pub/openedgar/release/1)

### Week 1 Goals

1, Split "High Impact/Low Impact" dataframe by industry run TF-IDF

2. TF-IDF Embeding, create timeline of corprate communcations, compare by Industry

**Deliverable** 11 csv files stored in S3 bucket

1. Consumer Discretionary
2. Financials
3. Consumer Staples
4. Industrials
5. Basic Materials
6. Health Care
7. Real Estate
8. Telecommunications
9. Technology
10. Utilities
11. Energy 


AlignedUMAP for Time Varying Data

<img width="910" alt="aligned_umap_politics_demo_spaghetti" src="https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/036510c0-e3f1-440d-ad61-521b2e15caf7">




# Refrence


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

  
4.  Data Extraction and Preprocessing

Extract Relevant Sections: Identify and extract ESG-relevant sections from 8-K filings

Text Cleaning: Standardize formatting and remove non-essential elements.

Refrence: https://scholar.harvard.edu/jbenchimol/files/text-mining-methodologies.pdf

5.  Labeling Data for Sentiment Analysis

Develop a Labeling Guide: Define positive, negative, and neutral ESG sentiments.

Manual Labeling: Label a subset of filings for training the sentiment analysis model.

Refrence: https://aws.amazon.com/blogs/machine-learning/use-sec-text-for-ratings-classification-using-multimodal-ml-in-amazon-sagemaker-jumpstart/



## Source: 
1.  Hartzmark, Samuel M., and Kelly Shue. "Counterproductive Sustainable Investing: The Impact Elasticity of Brown and Green Firms." 1 Nov. 2022, SSRN, https://ssrn.com/abstract=4359282 or http://dx.doi.org/10.2139/ssrn.4359282.

2.  Dubner, Stephen J. “Are E.S.G. Investors Actually Helping the Environment?” Freakonomics Radio, no. 546, Freakonomics, LLC, 14 June 2023, https://freakonomics.com/podcast/are-e-s-g-investors-actually-helping-the-environment/.

3.  



## Related Projects
1. https://www.mdpi.com/2071-1050/12/13/5317
2. https://www.mdpi.com/2071-1050/14/14/8745
3. https://www.pm-research.com/content/pmrjesg/1/3/39
