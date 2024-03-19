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


# Exploring the ESG Movement through Corporate 8-K Filings

Form 8-K, commonly referred to as a "current report," is a mandatory filing prescribed by the U.S. Securities and Exchange Commission (SEC) for publicly traded companies in the United States. Its purpose is to promptly notify shareholders of significant events that could impact the company's financial well-being or stock value. By offering a real-time overview of material changes in a company's operations or management, Form 8-K serves as a  tool for investors to stay informed about developments that may affect their investment decisions.

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




### Step 1C: Get list of US Equities held by Norewegian Wealth Fund from 2013-2023


```


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
