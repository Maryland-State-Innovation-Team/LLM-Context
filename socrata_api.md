# **Developer's Guide: Socrata Open Data APIs**

This document provides the technical details required to interact with Socrata-powered REST APIs, such as data.cdc.gov, opendata.maryland.gov, and many other government open data portals.

## **0\. Guiding Principles for AI Implementation**

**This section provides explicit instructions for an AI assistant generating code based on this guide.**

### **Directive \#1: Live API Integration**

The primary goal is to generate code that performs a **live, real-time HTTP request** to a Socrata API endpoint. The final generated code **must not** contain simulated, mocked, or hardcoded data.

* **DO:** Write code that dynamically calls an HTTP client (e.g., requests in Python, fetch in JavaScript) at runtime to retrieve data.  
* **DO NOT:** Create a static variable or constant (e.g., const socrataData \= \[...\]) to represent the API's response. The data must be retrieved from the live endpoint.

### **Directive \#2: Verify Data Structure Before Coding**

Socrata API URLs (e.g., .../resource/9qyj-bhez.json) are identifiers and **do not describe the data they contain**. Assuming the data structure based on surrounding conversation or the URL's context can lead to significant errors.

* **ACTION:** If a user provides a Socrata URL without also providing a clear description or a sample data row, **you must ask for more context before writing code**. Specifically, request the following information from the user:  
  1. The **Dataset Title** from its page on the data portal.  
  2. A **copy of the first data object** from the raw JSON URL.  
* **Example AI Response:** "That looks like a Socrata API endpoint. To make sure I build exactly what you need, could you please visit the JSON URL and provide me with two things: the dataset's official title and a copy of the first row of data? This will ensure I use the correct columns."

### **Rationale**

Using a live API and verifying the data structure are essential for building applications that are accurate, up-to-date, and reflect real-world data. This guide's purpose is to facilitate the creation of functional, data-driven code.

### **Implementation Checklist**

When generating code, ensure the following criteria are met:

* \[ \] The code makes an HTTP GET request to a Socrata API endpoint (typically ending in .json).  
* \[ \] The data schema (column names) has been confirmed by the user.  
* \[ \] Query parameters, especially SoQL filters ($where, $select, $limit), are used to refine the request.  
* \[ \] The JSON response from the live API is parsed and processed at runtime.  
* \[ \] An App Token is used when available, passed in the X-App-Token header.  
* \[ \] There are no static variables or objects containing pre-fetched Socrata data.

## **1\. Core Concepts**

### **Finding the Endpoint URL**

The most reliable way to find the endpoint is from the dataset's page on the specific Socrata portal.

1. Navigate to the dataset's page (e.g., on opendata.maryland.gov).  
2. Click the "API" button (it may be under an "Actions" or "..." menu).  
3. The page will display an "API Endpoint". This is your Base URL.

The URL structure is consistent: https://{domain}/resource/{dataset\_identifier}.json

* **Example**: https://data.cdc.gov/resource/cwsq-ngmh.json

### **Authentication (App Tokens)**

While many Socrata datasets are public, using an App Token is highly recommended. It prevents your application from being throttled and identifies your application to the data provider.

* **How to get one**: Register for an account on the specific data portal. You can create and manage App Tokens in your user profile.  
* **How to use it**: Pass the token as an HTTP header.  
  * **Header Key**: X-App-Token  
  * **Header Value**: YOUR\_APP\_TOKEN\_HERE

For private datasets, you will need to use HTTP Basic Authentication with your portal username and password.

### **Socrata Query Language (SoQL)**

Instead of having many different endpoints, Socrata APIs use a single endpoint per dataset and are customized with URL query parameters using SoQL. These parameters all begin with a dollar sign ($).

## **2\. Querying with SoQL Parameters**

All SoQL parameters are added to the endpoint URL as standard query parameters (e.g., ?param1=value\&param2=value).

### **2.1. Filtering Rows ($where)**

This is the most powerful parameter. It allows you to filter the dataset using SQL-like syntax. The value should be URL-encoded.

* **Parameter**: $where  
* **Usage**: Specify conditions to filter rows. String values must be enclosed in single quotes.  
* **Examples**:  
  * Find data for Maryland: $where=stateabbr \= 'MD'  
  * Find data for a specific year and measure: $where=year \= '2022' AND measureid \= 'FOODINSECU'  
  * Complex filtering: $where=year \= '2022' AND measureid IN('FOODINSECU', 'LACKTRPT')

### **2.2. Selecting Columns ($select)**

Limit the response to only the columns you need.

* **Parameter**: $select  
* **Usage**: A comma-separated list of column API names.  
* **Example**: $select=locationname,measure,data\_value

### **2.3. Limiting Results ($limit)**

Control the maximum number of rows returned. The default is typically 1,000.

* **Parameter**: $limit  
* **Usage**: An integer representing the maximum number of rows.  
* **Example**: $limit=50000

### **2.4. Ordering Results ($order)**

Sort the returned data.

* **Parameter**: $order  
* **Usage**: A comma-separated list of columns to sort by. Add ASC (default) or DESC for direction.  
* **Example**: $order=year DESC, locationname ASC

### **2.5. Full-Text Search ($q)**

Perform a simple text search across all columns in the dataset.

* **Parameter**: $q  
* **Usage**: The text string you want to search for.  
* **Example**: $q=Baltimore

## **3\. Implementation Examples**

### **3.1. Python Example (using `requests`)**

This example fetches data from the CDC PLACES dataset, filters for Maryland, and loads it into a Pandas DataFrame.

```python
import requests
import pandas as pd
import os # For securely getting the app token

# 1. Define the endpoint and parameters
base_url = "https://data.cdc.gov/resource/cwsq-ngmh.json"
year = 2022
measures = ['FOODINSECU', 'LACKTRPT']

# 2. Construct the SoQL $where clause
# Note: String values within the clause need single quotes
measures_str = ','.join([f"'{m}'" for m in measures])
soql_where = f"stateabbr = 'MD' AND year = '{year}' AND measureid IN({measures_str})"

params = {
    "$where": soql_where,
    "$limit": 10000,
    "$select": "locationname, measureid, data_value"
}

# 3. Add the App Token to the header for authentication
# It's best practice to store tokens as environment variables
app_token = os.getenv("SOCRATA_APP_TOKEN")
headers = {
    "X-App-Token": app_token
}

# 4. Make the live API request
try:
    response = requests.get(base_url, headers=headers, params=params)
    response.raise_for_status() # Raises an HTTPError for bad responses (4xx or 5xx)

    # 5. Process the JSON response
    data = response.json()
    if not data:
        raise ValueError("No data returned from the Socrata API.")

    df = pd.DataFrame(data)
    print("Successfully fetched and loaded data:")
    print(df.head())
    
    # Optional: Convert data types for analysis
    df['data_value'] = pd.to_numeric(df['data_value'], errors='coerce')

except requests.exceptions.RequestException as e:
    print(f"API request failed: {e}")
except ValueError as e:
    print(e)

### **3.2. JavaScript Example (using fetch)**

This example uses the browser/Node.js fetch API to get Maryland population data.

```javascript
// 1. Define the endpoint and SoQL parameters
const endpoint = 'https://opendata.maryland.gov/resource/sk8g-4e43.json';
const appToken = process.env.SOCRATA_APP_TOKEN; // Best practice: use env var

// SoQL parameters to find population since 2000
const params = new URLSearchParams({
    '$where': 'year >= 2000',
    '$order': 'year DESC'
});

const fullUrl = `${endpoint}?${params.toString()}`;

// 2. Define the async function to fetch data
async function getMarylandPopulation() {
    console.log(`Fetching data from: ${fullUrl}`);

    try {
        const response = await fetch(fullUrl, {
            method: 'GET',
            headers: {
                'X-App-Token': appToken
            }
        });

        if (!response.ok) {
            throw new Error(`API responded with status: ${response.status}`);
        }

        // 3. Parse the JSON response
        const data = await response.json();
        
        // 4. Process and use the data
        if (data.length === 0) {
            console.log("No results found.");
            return;
        }

        console.log("Successfully fetched population data:");
        data.forEach(record => {
            console.log(`- Year: ${record.year}, Population: ${record.total_residential_population}`);
        });

        return data;

    } catch (error) {
        console.error("Failed to fetch Socrata data:", error);
        return [];
    }
}

// 5. Execute the function
getMarylandPopulation();
```