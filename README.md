# Project overview 


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


# Exploring the ESG Movement through Corporate 10-K Filings

## Probelm Statement
 
This study evaluates ESG messaging evolution in 10-k filings. The aim is to identify trends and strategies in ESG communication among high and low environmental impact companies.


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

![Screen Shot 2024-03-19 at 7 22 01 PM](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/5dc4c64e-0041-48ee-acfd-c55ad0a24cde)




### Step 1C: Use sec-api.io to pull 10-K by ticker: 

See link: [SEC-API](https://sec-api.io/)

```
!pip install sec-api
```

See query: 


```
query = {
  "query": { "query_string": { 
      "query": "ticker:DDD AND filedAt:[2013-01-01 TO 2023-12-31] AND formType:\"10-K\"",
      "time_zone": "America/New_York"
  } },
  "from": "0",
  "size": "10",
  "sort": [{ "filedAt": { "order": "desc" } }]
}

response = queryApi.get_filings(query)
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

![Screen Shot 2024-03-20 at 12 32 55 PM](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/0ae39919-c0a4-44d1-bbac-b159b23f9697)



# Pulling a large amount of Data


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

1. Create dataframe for embedding across 1260 companies. 
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


Using Google Colab Pro to access GPU for embeddings + Gemma 7B as a LLM

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
