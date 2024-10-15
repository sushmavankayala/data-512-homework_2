# Analysis of Wikipedia Articles on Politicians
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

## Data Files and corresponding schemas

### Input
The below csv files are available in the resources folder of this repository

- [politicians_by_country_AUG.2024.csv](resources%2Fpoliticians_by_country_AUG.2024.csv): List of Wikipedia articles about politicians
```Text
name        # name of the article (politicians name)
url         # url to the wikipedia article
country     # country to which the politician is/was associated
```
- [population_by_country_AUG.2024.csv](resources%2Fpopulation_by_country_AUG.2024.csv): Population data by country
```Text
Geography        # name of the country/region
Population       # population in millions
```

### Generated
The below csv files are present in the generated_files folder of this repository

#### Intermediate files
- [wiki_list_with_revision.csv](generated_files%2Fwiki_list_with_revision.csv): Contains a list of articles and their latest revision ids, based on page info extracted from MediaWiki API
```Text
pageid          # page id of the wikipedia article
name            # name of the article (politicians name)
revision_id     # latest revision id of the wikipedia article (fetched from PageInfo call)
```
- [enriched_article_data.csv](generated_files%2Fenriched_article_data.csv): Contains a list of articles with their latest revision ids and corresponding article rating, based on the response from ORES API
```Text
pageid          # page id of the wikipedia article
name            # name of the article (politicians name)
revision_id     # latest revision id of the wikipedia article (fetched from PageInfo call)
article_rating  # article rating fetched from the ORES API call
                    Possible values for article_rating are:
                    FA      # Featured article
                    GA      # Good article (also known as A-Class)
                    B       # B-Class article
                    C       # C-Class article
                    Start   # Start-class article
                    Stub    # Stub-class article
```
#### Output files
- [wp_politicians_by_country.csv](generated_files%2Fwp_politicians_by_country.csv): Combined data on politicians, countries, and article quality
```Text
country        # country name
region         # region associated to the country
population     # population of the country (in millions)
article_title  # name of the article (politician's name) 
revision_id    # latest revision id of the wikipedia article ( fetched from PageInfo call)
article_rating # article rating fetched from the ORES API call
```
- [wp_countries-no_match.txt](generated_files%2Fwp_countries-no_match.txt): list of countries with no matches between datasets.

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
This assignment highlights the importance of recognizing biases in source data to identify model biases and avoid misinterpreting findings. Most online information inherently contains some bias, which can stem from various factors such as demographics, gender, cultural prejudices, and literacy rates. Post-hoc analysis raised questions about the reliability of articles per capita as an indicator of coverage and its correlation with ORES ratings. Biases were observed in Eastern Europe and Western Asia, where rankings for high-quality articles were disproportionately higher compared to total articles. Population size significantly influenced articles per capita values, with the highest coverage often found in less populous countries/regions. A cultural-linguistic bias was evident, as non-native English speakers produced fewer high-quality articles than native speakers. These inherent biases and data inconsistencies cast doubt on the reliability of the per capita calculations performed in the final stages of the exercise.

### What biases did you expect to find in the data (before you started working with it), and why?
Prior to the exercise, an anticipated bias towards developed countries for high-quality articles was not reflected in the Wikipedia data. The exercise exhibited selection bias due to missing data for numerous countries, including major ones like the US, Canada, and Australia. Lower articles per capita were expected for South and East Asia due to high populations. Smaller countries were anticipated to show inflated articles per capita due to their lower overall article count. Politicians from these countries were expected to have lower ratings as many might not be well-known. Asian and African countries were presumed to have fewer articles per capita, attributed to literacy rates or general cultural-linguistic biases, suggesting that articles in native languages might be of higher quality.
### What might your results suggest about (English) Wikipedia as a data source?
The results indicate that Wikipedia data is dynamic and subject to change across multiple runs, potentially leading to inconsistent analysis. ORES scores may not be entirely reliable as they are generated by an AI model susceptible to ideological, racial, gender, or cultural biases. The ORES Wiki reveals that the model's evaluation of article quality appears to be biased towards article structure rather than content.
### Can you think of a realistic data science research situation where using these data (to train a model, perform hypothesis-driven research, or make business decisions) might create biased or misleading results, due to the inherent gaps and limitations of the data?
Biases could emerge in various fields and models, including content moderation and NLP-based technologies that gather data from the internet. As observed in the readings, gender, demographic, and cultural biases might lead to inaccurate model predictions. The Islamophobia article exemplifies how GPT-3 exhibits bias against a specific religion. These limitations stem from the data fed into AI systems, adhering to the "Garbage In, Garbage Out" principle.
### How might a researcher supplement or transform this dataset to potentially correct for the limitations/biases you observed?
To address the dataset's limitations, a researcher can:
- Extract Wikipedia articles for politicians from countries with missing information.
- Obtain ORES scores for these additional articles to provide a more comprehensive view of high-quality-article-per-capita ratios.
- Update population values for countries currently listed as having zero population.
These steps would help create a more accurate representation of articles-per-capita and high-quality-article-per-capita across all regions and countries.

## Limitations and considerations

 - Some entries, like Mohammad Khan (athlete), Robert Rey (plastic surgeon), etc are not primarily politicians. However, upon reading about these individuals, I learnt that they've had political involvement at some point, justifying their inclusion in the analysis.
 - The PageInfo API call failed for a few articles; 8 to be precise. I've decided to retain the articles for processing, with empty value stored in the latest revision id.
 - The ORES scores API occasionally failed due to timeouts.The call failed 7 times for me, and on discussing with a few individuals in the cohort - I learnt that each of them have different number of API call failure. This suggests that the ORES API call is somewhat unreliable.
 - Countries like Monaco and Tuvalu show populations as '0 million,' when they are likely in the thousands. This still implies that the per capita value is higher, and hence final analysis has 2 tables - with and without these countries with 0.0 as population in millions