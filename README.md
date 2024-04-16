# Decoding Corporate Narratives: A RAG-Based Approach to Navigating 10-K Filings

This project is an attempt to understand the shifting messaging ESG messaging in 10-K filings using a RAG (Retrival Augmented Generation Pipeline). 

## The Landscape

ESG metrics encompass three fundamental pillars: 

1. The "E" dimension evaluates a company's management of natural resources and its environmental impact, encompassing both direct operations and supply chain activities.

2. The "S" aspect pertains to social factors, assessing a company's effectiveness in navigating social trends, labor practices, and political dynamics.

3. The "G" component of ESG focuses on governance factors, examining decision-making processes from governmental policy-making to the allocation of rights and responsibilities within corporations. This includes scrutiny of governance structures involving the board of directors, managers, shareholders, and stakeholders

ESG ratings play a role in guiding trillions of dollars in investments worldwide. For example, the investment strategy of the Maine Public Employees Retirement System Pension Fund is guided by ESG criteria, and they highlight the significance of integrating sustainability factors for long-term investment success.

In recent years, ESG-related shareholder proposals—an official vehicle through which shareholders can interface with the board of directors—have become more prominent.

#### Number of Shareholder Proposals by Sub-Categories, 2022-2023
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

#### **Investors’ Support for “Pro-ESG” Shareholder Proposals at S&P 500 Companies**
![image-32](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/7cc9803a-65cb-459c-8174-2a8bd0f39bf4)


## The Goal
The goal of this project is to equip large language models (LLMs) with domain-specific data derived from the 10-K disclosure filings of 968 publicly traded firms, as well as the Norwegian Wealth Fund's voting patterns on shareholder proposals. Enabling the LLMs to tailor their outputs, drawing context from authoritative sources concerning environmental, social, and governance (ESG) messaging and Corprate Goverannce. 

## An Overview: 

This project is broken down into a few steps. 

#### 1. Data Collection from Norwegian Sovereing Wealth Fund to get the list of US equities:
   
Link to API : https://www.nbim.no/en/responsible-investment/voting/our-voting-records/api-access-to-our-voting/

#### 2. Data Collection from SEC EDGAR System to get Corprate 10-K filings:

Link to sec-api.io : https://sec-api.io/docs/sec-filings-item-extraction-api
 
#### 3. Data preprocessing: Ingesting text into a pdf then turn into dictionary, split into chunks and report on token count. 

Link to open source nlp preprocesser spaCy: https://spacy.io/api/sentencizer

#### 4. Embedding the chunks: use a pretrained model mpnet-base model 

Link to hugging face: https://huggingface.co/sentence-transformers/all-mpnet-base-v2

#### 5. Creating a sematic search pipeline between a user query and the text

   
#### 6. Loading an LLM locally

Link to LLM: https://huggingface.co/google/gemma-7b-it
   
#### 7. Generating text with an LLM
 


# Building a local RAG (Retrival Augmented Generation Pipeline) pipeline for 10-K documents

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


# Step 2: Use sec-api.io to pull 10-K by ticker: 

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
# Load the serialized data from the pickle file
with open('Cleaned_US_Item1_1A.pkl', 'rb') as f:
    documents_info = pickle.load(f)
```

Example pull for a 10-K document, get section 1A Risks and clean the text) 
![Screen Shot 2024-04-16 at 3 57 55 PM](https://github.com/R0bL/Project_Initiation_DS5500/assets/133535059/ed1a27c8-af72-43ce-92b6-4d343ccd2f6d)


### For the sake of this example, filter out dataframe and only consider 10-K disclosures for 2023 in the Utility Sector

```
# Load the serialized data from the pickle file
with open('Cleaned_US_Item1_1A.pkl', 'rb') as f:
    documents_info = pickle.load(f)

# Create a DataFrame from the documents_info list
Cleaned_US_Item1_1A = pd.DataFrame(documents_info)

df_2023 = Cleaned_US_Item1_1A[Cleaned_US_Item1_1A['filedAt'].str.startswith('2023')]

unique_utility_df = df_2023[df_2023['sector'] == 'Utilities']

```


### Creating a PDF with Metadata (there way to do this) 
```
def draw_metadata(c, metadata, width, height):
    """Draws metadata at the top of each page."""
    c.setFont("Helvetica", 12)
    c.drawString(72, height - 50, f"Ticker: {metadata['ticker']}, Sector: {metadata['sector']}, Filed At: {metadata['filedAt']}")

def add_text_to_page(c, text, metadata, width, height):
    """Adds text to a page ensuring metadata is drawn first and proper spacing is maintained."""
    draw_metadata(c, metadata, width, height)
    text_object = c.beginText(72, height - 100)  # Adjusted to leave space below the metadata
    text_object.setFont("Helvetica", 10)
    for line in text.split():
        line = preprocess_text(line)  # Preprocess each line if necessary
        if text_object.getX() + c.stringWidth(line) > width - 72:
            text_object.textLine()  # Move to next line if text exceeds the page width
        if text_object.getY() < 100:  # Check if we're near the bottom of the page
            c.drawText(text_object)  # Draw the text collected so far
            c.showPage()  # Start a new page
            draw_metadata(c, metadata, width, height)  # Redraw metadata at top of the new page
            text_object = c.beginText(72, height - 100)
        text_object.textOut(line + " ")  # Add space between words
    c.drawText(text_object)  # Make sure to draw any remaining text
    c.showPage()  # Ensure a new page is started after finishing each company's text

def create_pdf(df, filename):
    """Creates a PDF file with each entry separated onto a new page with metadata at the top."""
    c = canvas.Canvas(filename, pagesize=letter)
    width, height = letter
    for index, row in df.iterrows():
        metadata = {'ticker': row['ticker'], 'sector': row['sector'], 'filedAt': row['filedAt']}
        add_text_to_page(c, row['text'], metadata, width, height)
    c.save()
    print(f"PDF saved as {filename}")

create_pdf(df=unique_utility_df, filename='utility.pdf')
```

# Step 3: 

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
