# Wikipedia Political Figures Analysis
#### DATA-512 | Homework 2 | Considering Bias in Data

## Project Goal
This project explores bias in Wikipedia articles about political figures across different countries. We combine Wikipedia article data with country population statistics and use the ORES machine learning service to estimate article quality. The analysis focuses on:

1. Countries with the highest and lowest coverage of politicians relative to population.
2. Countries with the highest and lowest proportion of high-quality articles about politicians.
3. Ranking of geographic regions by articles-per-person and proportion of high-quality articles.

## Licenses

### Source Data
The Wikipedia [Category:Politicians by nationality](https://en.wikipedia.org/wiki/Category:Politicians_by_nationality) was crawled to generate a list of Wikipedia article pages about politicians from a wide range of countries. This data is in the homework folder as politicians_by_country.AUG.2024.csv.
The list of articles mentioned in the csv file [politicians_by_country_AUG.2024.csv](resources%2Fpoliticians_by_country_AUG.2024.csv) has information about the article name, url, and country the politician is/was associated to.

[Wikimedia Foundation Terms of Use](https://foundation.wikimedia.org/wiki/Policy:Terms_of_Use) allows free use of content while requiring attribution, preserving open licenses, and respecting copyright and user rights.
To ensure that this project complies with Wikimedia's Terms of Use, I've provided the required attribution referencing the original source of data.

The population data is available in the csv file [population_by_country_AUG.2024.csv](resources%2Fpopulation_by_country_AUG.2024.csv) was downloaded from the [world population data sheet](https://www.prb.org/international/indicator/population/table/). This data is published by the Population Reference Bureau.

### Code for Data Acquisition

Snippets of the code under [Wiki_articles_bias_analysis.ipynb](Wiki_articles_bias_analysis.ipynb) were taken from a code example developed by Dr. David W. McDonald for use in DATA 512, a course in the UW MS Data Science degree program. This code is provided under the [Creative Commons CC-BY license](https://creativecommons.org/licenses/by/4.0/)
A copy of the reference code can be found in the `reference_code` folder of this project

## Documentation

#### API
- [MediaWiki PageInfo API](https://www.mediawiki.org/wiki/API:Info) : API to fetch page info for a given article title.
- [ORES API](https://www.mediawiki.org/wiki/ORES) : API to fetch article rating for a given article's revision id.

#### Packages
* [urllib](https://docs.python.org/3/library/urllib.request.html) : For fetching data from MediaWiki's PageInfo API
* [pandas](https://pandas.pydata.org/docs/reference/index.html) : For processing the timeseries data fetched from the API calls
* [request](https://requests.readthedocs.io/en/latest/api/#) : For calling the MediaWiki and ORES APIs

## How to Run

The code for the entire analysis resides in [Wiki_articles_bias_analysis.ipynb](Wiki_articles_bias_analysis.ipynb).\

### Get access token

To access Lift Wing (the ML API service), you'll need a Wikimedia account. If you already have a Wikipedia account, it might work. Otherwise, create a new account to get an access token.
Use this [official guide](https://api.wikimedia.org/wiki/Authentication) to create an API key for your account. * Make sure to create a Personal API token instead of Server-side app key* 
On successfully creating the access token, you'll receive a Client ID, Client Secret, and Access Token—save all three. If lost, you'll need to deactivate the token and create a new one.

In the code for data acquisition, make sure to use your username and the Access Token generated.

The notebook does not need any additional configurations, so you can use the `Run All Cells` or `Restart Kernel and Run All Cells`  option of the jupyter notebook.

Note:
- The data acquisition step takes ~2 hours to loop through the revision ids of all articles mentioned in the CSV file and do the ORES call to fetch the article rating. This is attributed to the number of articles and sleep time added to ensure that we do not hit throttling limits of the ORES API
- Make sure that the modules `requests` and `pandas` are installed before running the notebook. You can use `pip/pip3` for installation.

## Data Files

### Input
The below csv files are available in the resources folder of this repository

- [politicians_by_country_AUG.2024.csv](resources%2Fpoliticians_by_country_AUG.2024.csv): List of Wikipedia articles about politicians
- [population_by_country_AUG.2024.csv](resources%2Fpopulation_by_country_AUG.2024.csv): Population data by country

### Generated
The below csv files are present in the generated_files folder of this repository
- [wiki_list_with_revision.csv](generated_files%2Fwiki_list_with_revision.csv): Contains a list of articles and their latest revision ids, based on page info extracted from MediaWiki API
- [enriched_article_data.csv](generated_files%2Fenriched_article_data.csv): Contains a list of articles with their latest revision ids and corresponding article rating, based on the response from ORES API
- [wp_politicians_by_country.csv](generated_files%2Fwp_politicians_by_country.csv): Combined data on politicians, countries, and article quality
- [wp_countries-no_match.txt](generated_files%2Fwp_countries-no_match.txt): Countries with no matches between datasets

## Processing workflow

#### Step 1: Fetching Page Information

The script loops through a dataset of article titles, retrieving page information for each one from the Wikimedia API. This data is saved to an intermediary file for later use.

#### Step 2: Retrieving Quality Scores

For each article, the script uses the last revision ID to request a quality score from the ORES API. If successful, the score is saved to the intermediary CSV file. If the request fails or a revision ID is missing, those articles are logged.

#### Step 3: Generating the Output File

Once the ORES data is gathered, it is combined with the Wikipedia page info. Articles without revision IDs are included but left with blank score fields. The dataset is then merged with population data, and the final output is stored as a CSV file. Any countries that were not matched are recorded in a text file.

#### Step 4: Data Analysis

The analysis produces the following insights from the generated output file [wp_politicians_by_country.csv](generated_files%2Fwp_politicians_by_country.csv) :

- **Top 10 countries by article coverage**: Countries with the highest number of articles per capita (descending order).
- **Bottom 10 countries by article coverage**: Countries with the lowest number of articles per capita (ascending order).
- **Top 10 countries by article quality**: Countries with the most high-quality articles per capita (descending order).
- **Bottom 10 countries by article quality**: Countries with the fewest high-quality articles per capita (ascending order).
- **Regions by total article coverage**: A ranked list of geographic regions by total articles per capita (descending order).
- **Regions by article quality coverage**: A ranked list of regions by high-quality articles per capita (descending order).

## Research Implications

TODO: Add details
