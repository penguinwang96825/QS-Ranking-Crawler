# QS_ranking_web_scraping
A tutorial to build a web scraper by using beautifulsoup and selenium.

## Import packages
Download chrome web dirver first. [[Click here](https://sites.google.com/a/chromium.org/chromedriver/)]
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import time
import bs4
from bs4 import BeautifulSoup
from selenium.common.exceptions import *
from selenium import webdriver
%matplotlib inline
```

## Main coding for scraping qs ranking 2020 table
```python
def get_uni_information(year, unilist, page):
    url = r"https://www.topuniversities.com/university-rankings/world-university-rankings/{}".format(year)
    # Open url and get the QS Ranking html page
    driver_path = r"C:\Users\YangWang\Desktop\machineLearning\indiaNewsClassification\chromedriver.exe"
    driver = webdriver.Chrome(driver_path)
    time.sleep(2)
    driver.get(url)
    time.sleep(5)
    
    # Scrawl all the pages (max page is 40)
    for _ in range(int(page)):
        # Use BeautifulSoup to parse every page
        soup = BeautifulSoup(driver.page_source, "html.parser")
        # Find the table which contains the ranking information of every universities
        x = soup.find(name="table", attrs={"class": "dataTable no-footer"})
        # Use for loop to catch every rows in the table, and append the rows into the list
        for tr in x.find(name="tbody"):
            try: 
                tds = tr('td')
                if tds[0].find(name="span") is not None:
                    rank = tds[0].find(name="span").string
                else: 
                    rank = None
                if tds[1].find(name="a") is not None:
                    uni = tds[1].find(name="a").string
                else: 
                    uni = None
                if tds[2].find(attrs={"class": "td-wrap"}) is not None:
                    location = tds[2].find(attrs={"class": "td-wrap"}).string
                else: 
                    location = None
            except (RuntimeError, TypeError, NameError):
                pass
            unilist.append([rank, uni, location])
        # Click next page button
        element = driver.find_element_by_xpath('//*[@id="qs-rankings_next"]')
        driver.execute_script("arguments[0].click();", element)
        time.sleep(5)
    
    driver.quit()
    return unilist
```

## Get the dataframe
```python
def get_qs_ranking_dataframe(year, page):
    unilist = []
    unilist = get_uni_information(year, unilist, page)
    df = pd.DataFrame(unilist)
    df.columns = ["ranking", "uni", "location"]
    df.reset_index(drop=True)
    
    # Dataframe preprocessing
    df["ranking"] = [int(x)+1 for x in range(len(df))]
    df["uni"] = df["uni"].map(str)
    df["location"] = df["location"].map(str)
    
    return df
```

## Visualise top universities
```python
def visualise_qs_ranking(df, year, top_ranking, num):
    """
    df: dataframe
    year: year of the qs ranking
    top_ranking: top # of universities to be selected
    num: # of countries to be visaulised
    """
    plt.style.use('seaborn-paper')
    top = df.iloc[0:top_ranking]
    
    ax = (top['location'].value_counts().head(num).plot(
        kind='barh', 
        figsize=(20, 10), 
        color="tab:blue", 
        title="Number of Top {} Universities in QS Ranking {}".format(len(top['location']), str(year))))
    ax.set_xticks(np.arange(0, top['location'].value_counts()[0]+2, 1))
```

## Do some visualisation
```python
qs_2020 = get_qs_ranking_dataframe(year=2020, page=40)
qs_2020[(qs_2020["location"] == "Japan") & (qs_2020["ranking"] <= 100)]
```

| ranking | uni | location |
| --- | --- | --- |
| 23 | The University of Tokyo | Japan |
| 34 | Kyoto University | Japan |
| 59 | Tokyo Institute of Technology (Tokyo Tech) | Japan |
| 71 | Osaka University | Japan |
| 82 | Tohoku University | Japan |
