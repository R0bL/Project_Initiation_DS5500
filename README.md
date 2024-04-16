# Decoding Corporate Narratives: A RAG-Based Approach to Navigating 10-K Filings

This project is an attempt to understand the shifting messaging ESG messaging in 10-K filings using a RAG (Retrival Augmented Generation Pipeline). 

## The Landscape

ESG metrics encompass three fundamental pillars: 

1. The "E" dimension evaluates a company's management of natural resources and its environmental impact, encompassing both direct operations and supply chain activities.

2. The "S" aspect pertains to social factors, assessing a company's effectiveness in navigating social trends, labor practices, and political dynamics.

3. The "G" component of ESG focuses on governance factors, examining decision-making processes from governmental policy-making to the allocation of rights and responsibilities within corporations. This includes scrutiny of governance structures involving the board of directors, managers, shareholders, and stakeholders

ESG ratings play a role in guiding trillions of dollars in investments worldwide. For example, the investment strategy of the Maine Public Employees Retirement System Pension Fund is guided by ESG criteria, and they highlight the significance of integrating sustainability factors for long-term investment success.

![image-31](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/089b5330-35f6-46f3-9d53-eef8d96d84f4)

#### The Challenges 


1. Lack of transparency among the ESG raters on how the scores are assigned.
2. Lack of standards on how a particular concept is measured.
3. Questionable tradeoffs: high scores in one domain may offset very low scores in another area.
4. Absence of an overall score combining performance scores along Environmental, Social, and Governance (ESG) axes; weights given to the "E," "S," and "G."
5. Lack of acknowledgement of stakeholder expectations.



## The Data

This analysis specifically examines the 966 US equities in which the Norwegian Sovereign Wealth Fund has invested. The $1.4 trillion fund, managed by the Norwegian government, originated from oil and gas resources discovered in the late 1960s on the Norwegian continental shelves. Serving as a strategic financial reserve, the fund holds stakes in about 9,000 companies worldwide, owning approximately 1.4 percent of every listed company globally. 

I have chosen to focus on this subset of publicly traded companies to benchmark US firms against European long-term sustainable investing strategies, specifically examining the Norwegian Sovereign Wealth Fund's voting guidelines. This approach aims to compare the governance and sustainability practices of American corporations with those promoted by one of Europe's most significant investors, known for its adherence to ESG principles.


![image-32](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/7cc9803a-65cb-459c-8174-2a8bd0f39bf4)


## the Goal
The goal of this project is to equip large language models (LLMs) with domain-specific data derived from the 10-K disclosure filings of 968 publicly traded firms, as well as the Norwegian Wealth Fund's voting patterns on shareholder proposals. Enabling the LLMs to tailor their outputs, drawing context from authoritative sources concerning environmental, social, and governance (ESG) messaging and Corprate Goverannce. 

## An overview: 

This project is broken down into a few steps. 

1. Data Collection and Cleaning: (1) Collecting data from Norwegain Wealth Fund API (2) Validating/Cleaning the data with Yahoo Finance

https://www.nbim.no/en/responsible-investment/voting/our-voting-records/api-access-to-our-voting/
 
2. Ingesting text: Splitting the text into chunks
   
3. Embedding the chunks: use a pretrained model mpnet-base model 

4. Creating a sematic search pipeline
   
5. Loading an LLM locally
   
6. Generating text with an LLM
 





## The Challenges 


1. Lack of transparency among the ESG raters on how the scores are assigned.
2. Lack of standards on how a particular concept is measured.
3. Questionable tradeoffs: high scores in one domain may offset very low scores in another area.
4. Absence of an overall score combining performance scores along Environmental, Social, and Governance (ESG) axes; weights given to the "E," "S," and "G."
5. Lack of acknowledgement of stakeholder expectations.


# Exploring the ESG Movement through Corporate 10-K Filings



## Step 1: Data Collection

### Step 1A: Get list of US Equities held by Norewegian Wealth Fund from 2013-2023


A link to the API can be found here: [Norges Bank Investment Management API](https://www.nbim.no/en/responsible-investment/voting/our-voting-records/api-access-to-our-voting/) 


I used this code to get all companies listed in their database: 

```
api_key = " Insert Your API Key Here" 

def fetch_company_details(api_key, company_name):
    detail_url = f"https://vd.a.nbim.no/v1/query/company/{requests.utils.quote(company_name)}"
    headers = {"x-api-key": api_key}
    response = requests.get(detail_url, headers=headers)
    if response.status_code == 200:
        return response.json()
    elif response.status_code == 429:
        print("Rate limit exceeded. Sleeping...")
        time.sleep(900)  # Sleep time might need adjustment
        return fetch_company_details(api_key, company_name)
    else:
        print(f"Failed to fetch details for company {company_name}: {response.status_code}")
        return {}


def build_companies_dataframe(api_key, companies_list, save_path):
    detailed_companies = []
    start_from_index = get_last_processed_index(companies_list, save_path)

    for i, company in enumerate(tqdm(companies_list[start_from_index:], desc="Fetching company details"), start=start_from_index):
        company_name = company['n']
        company_details = fetch_company_details(api_key, company_name)
        if 'companies' in company_details:
            for detail in company_details['companies']:
                detailed_companies.append({
                    'Company Name': company_name,
                    'Ticker': detail.get('Ticker', 'N/A'),
                    'Country': detail.get('country', 'N/A')
                })
        if (i + 1 - start_from_index) % 5 == 0 or i == len(companies_list) - 1:
            pd.DataFrame(detailed_companies).to_csv(save_path, mode='a', header=not os.path.exists(save_path), index=False)
            detailed_companies = []  # Reset to avoid re-saving data

        time.sleep(1)  # Consider dynamic adjustment based on API's rate limiting response

    if os.path.exists(save_path):
        return pd.read_csv(save_path)
    else:
        return pd.DataFrame()

```

If API times out, restart at index: 

```
def get_last_processed_index(companies_list, save_path):
    try:
        df = pd.read_csv(save_path)
        # Correct column name to 'Company Name'
        last_processed_company = df.iloc[-1]['11 88 0 Solutions AG']
        for index, company in enumerate(companies_list):
            if company['n'] == last_processed_company:
                return index + 1  # Resume from the next company
    except (pd.errors.EmptyDataError, FileNotFoundError):
        print("CSV file is empty or does not exist.")
    return 0
```

### Step 1B: Cross refrence data with yfinance

The data being evaluated are SEC filings of 1260 Equities held by the Norwegain Wealth Fund, downloaded from the SEC's Electronic Data Gathering, Analysis and Retrieval (EDGAR) website. 

Cross refrenced data pulled with y_finance [yfinance](https://pypi.org/project/yfinance/) - Norwegain Wealth Fund Database did not have the correct industry tags: 

```
import pandas as pd
import yfinance as yf
from tqdm import tqdm

# Assuming 'test' is your dataframe and it already exists
# Make sure it is either the full dataframe or a copy, to avoid SettingWithCopyWarning:
# test = df.head().copy()

# Enable tqdm for pandas apply
tqdm.pandas()

# Function to fetch sector for a ticker
def fetch_sector(ticker):
    try:
        ticker_data = yf.Ticker(ticker)
        return ticker_data.info.get('sector', "N/A")
    except Exception as e:
        return "Error"

# Apply the function to your dataframe to create a new 'Sector' column
# Using progress_apply instead of apply to show the progress bar
df['Sector'] = df['Ticker'].progress_apply(fetch_sector)

# Display the updated dataframe
df.head()
```


### Step 1C: Use sec-api.io to pull 10-K by ticker: 

See link: [SEC-API](https://sec-api.io/)

```
!pip install sec-api
```

See query: 


```
import pickle
import re
from tqdm import tqdm

# Assuming `df_documents_info_cleaned` is your DataFrame containing the URLs and metadata
documents_info = []

for index, row in tqdm(df_documents_info_cleaned.iterrows(), total=df_documents_info_cleaned.shape[0], desc="Fetching and Cleaning Documents"):
    # Extract necessary information from the row
    ticker = row['ticker']
    filedAt = row['filedAt']
    sector = row['sector']
    filing_url = row['linkToFilingDetails']
    
    # Fetch the document text for both sections 1 and 1A
    section_1_text = extractorApi.get_section(filing_url, "1", "text")
    section_1A_text = extractorApi.get_section(filing_url, "1A", "text")
    
    # Combine both sections' texts
    combined_text = section_1_text + " " + section_1A_text  # Ensure there's a space between the two sections
    
    # Clean the combined section text
    cleaned_combined_section = re.sub(r"\n|&#[0-9]+;", "", combined_text)
    
    # Append a dictionary for each document containing its metadata and cleaned combined text
    documents_info.append({
        'ticker': ticker,
        'filedAt': filedAt,
        'sector': sector,
        'text': cleaned_combined_section
    })

# Serialize the list of dictionaries to a file using pickle
with open('Cleaned_US_Item1_1A.pkl', 'wb') as f:
    pickle.dump(documents_info, f)
```

Convert to Pandas Dataframe: 

```
import pandas as pd

metadata = pd.DataFrame.from_records(response['filings'])
metadata
```

![Screen Shot 2024-03-19 at 7 29 31 PM](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/ae71069d-a4d8-417e-9eb7-5db42130d78c)


Example pull for a 10-K document, get section 1A Risks and clean the text) 

```
# Example 10-K filing
filing_url = metadata['linkToFilingDetails'][0]

# get the standardized and cleaned text of section 1A "Risk Factors"
section_text = extractorApi.get_section(filing_url, "1A", "text")


# we use a regular expression to substitute new line characters and HTML entities
# with an empty string ""
import re
cleaned_section = re.sub(r"\n|&#[0-9]+;", "", section_text)
```

Output: 
Item 1A. Risk Factors You should carefully read the following discussion of significant factors, events and uncertainties when evaluating our business and the forward-looking information contained in this Form 10-K. The events and consequences discussed in these risk factors could materially and adversely affect our business, operating results, liquidity and financial condition. While we believe we have identified and discussed below the key risk factors affecting our business, these risk factors do not identify all the risks we face, and there may be additional risks and uncertainties that we do not presently know or that we do not currently believe to be significant that may have a material adverse effect on our business, performance or financial condition in the future. Operational  Financial Risk Factors Our uneven sales cycle makes planning and inventory management difficult and future financial results less predictable. Our quarterly sales often have reflected a pattern in which a disproportionate percentage of each quarters total sales occur towards the end of the quarter, in particular for sales of hardware and software products. This uneven sales pattern makes predicting net revenue, earnings, cash flow from operations and working capital for each financial period difficult, increases the risk of unanticipated variations in our quarterly results and financial condition and places pressure on our inventory management and logistics systems. If predicted demand is substantially greater than orders, there may be excess inventory. Alternatively, if orders substantially exceed predicted demand, we may not be able to fulfill all of the orders received in each quarter and such orders may be canceled. Depending on when they occur in a quarter, developments such as an information systems failure, component pricing movements, component shortages or global logistics disruptions could adversely impact our inventory levels and results of operations in a manner that is disproportionate to the number of days in the quarter affected. The variety of products that we sell could cause significant quarterly fluctuations in our gross profit margins, and those fluctuations in margins could cause fluctuations in operating income or loss and net income or loss.



### Pulling URLS for all firms considered

```
from sec_api import QueryApi
import pandas as pd
from tqdm.auto import tqdm

def fetch_filings_for_tickers_to_df(tickers, start_year, end_year, api_key):
    query_api = QueryApi(api_key=api_key)
    filings_list = []

    for ticker in tqdm(tickers, desc="Processing tickers"):
        for year in tqdm(range(start_year, end_year + 1), desc="Years", leave=False):
            for month in tqdm(range(1, 13), desc="Months", leave=False):
                from_param = 0
                more_filings_exist = True

                while more_filings_exist:
                    query = {
                      "query": { 
                          "query_string": { 
                              "query": f"ticker:{ticker} AND filedAt:[{year}-{month:02d}-01 TO {year}-{month:02d}-31] AND formType:\"10-K\"",
                              "time_zone": "America/New_York"
                          } 
                      },
                      "from": str(from_param),
                      "size": "200",
                      "sort": [{"filedAt": {"order": "desc"}}]
                    }

                    response = query_api.get_filings(query)
                    filings = response['filings']

                    if filings:
                        for filing in filings:
                            # Here you can add any information you want about each filing to the dictionary
                            filings_list.append({
                                "ticker": ticker,
                                "filedAt": filing.get("filedAt", ""),
                                "formType": filing.get("formType", ""),
                                "linkToFilingDetails": filing.get("linkToFilingDetails", "")
                            })
                    else:
                        more_filings_exist = False
                    
                    from_param += 200

    filings_df = pd.DataFrame(filings_list)
    return filings_df

# Example usage
tickers = tickers
api_key = 'API KEY'  # Replace with your actual SEC API key
filings_df = fetch_filings_for_tickers_to_df(tickers, 2013, 2023, api_key)

filings_df.head()
```


# Week 1 Report

### Project Updates

1. Access company list of Norwegian Wealth Fund API: 


### Week 1 Goals

1. Build company database

2. File list of 10-K companie by industry: 

# Sector Overview

The following table provides an overview of the number of companies across various sectors:

| Sector                  | Number of Companies |
|-------------------------|---------------------|
| Basic Materials         | 63                  |
| Communication Services | 35                  |
| Consumer Cyclical       | 96                  |
| Consumer Defensive      | 38                  |
| Energy                  | 58                  |
| Financial Services      | 153                 |
| Healthcare              | 108                 |
| Industrials             | 157                 |
| Real Estate             | 87                  |
| Technology              | 134                 |
| Utilities               | 37                  |







# Week 2 Report

### Project Updates

1. Pull 10-K filings using sec-io
2. Validate data using yfinance
   

### Week 2 Goals

1. Clean text
2. Prepare of embedding


# Week 3 Report

### Project Updates

1. Create standarized notebooks to pull data
2. Started running TF-IDF

### Week 3 goals

1. Create dataframe for embedding across 966 companies. 
2. Generate U-MAP for a single time slice


![Screen Shot 2024-03-19 at 7 43 41 PM](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/37b3439d-2499-4e93-aafa-6526b6b12cf3)


# Week 4 Report

1. Apply ESG BERT See: https://www.sciencedirect.com/science/article/pii/S1544612324000096?via%3Dihub#da1

Download the model from hugging face: https://huggingface.co/ESGBERT

https://huggingface.co/climatebert/distilroberta-base-climate-sentiment

2. Summarize text using GPT.
   https://medium.com/@jan_5421/summarize-sec-filings-with-openai-gtp3-a363282d8d8


# Week 5 Report

Going to impliment a Retrieval-Augmented Generation (RAG) pipeline for 10-K disclousre text. 

LLM to reference an authoritative knowledge base (10-K Text) outside the training data before generating a response. 

See flowchart: 

![PNG image](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/60fc8fc5-60ee-4025-aa87-fa5328625b9f)


Using Google Colab Pro to access A100 GPU for embeddings + Gemma 7B as a LLM

GPU memory: 40 | Recommend model: Gemma 7B in 4-bit or float16 precision.

```
Note: the following is Gemma focused, however, there are more and more LLMs of the 2B and 7B size appearing for local use.
if gpu_memory_gb < 5.1:
    print(f"Your available GPU memory is {gpu_memory_gb}GB, you may not have enough memory to run a Gemma LLM locally without quantization.")
elif gpu_memory_gb < 8.1:
    print(f"GPU memory: {gpu_memory_gb} | Recommended model: Gemma 2B in 4-bit precision.")
    use_quantization_config = True 
    model_id = "google/gemma-2b-it"
elif gpu_memory_gb < 19.0:
    print(f"GPU memory: {gpu_memory_gb} | Recommended model: Gemma 2B in float16 or Gemma 7B in 4-bit precision.")
    use_quantization_config = False 
    model_id = "google/gemma-2b-it"
elif gpu_memory_gb > 19.0:
    print(f"GPU memory: {gpu_memory_gb} | Recommend model: Gemma 7B in 4-bit or float16 precision.")
    use_quantization_config = False 
    model_id = "google/gemma-7b-it"


print(f"use_quantization_config set to: {use_quantization_config}")
print(f"model_id set to: {model_id}")
```

Refrence: https://github.com/mrdbourke/simple-local-rag/tree/main

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
