# QS Ranking Web Scraping
A note to build a web scraper by using beautifulsoup and selenium. This program crawls [QS Rankings](https://www.topuniversities.com/qs-world-university-rankings) to discover the top universities from all over the world. Uni name, ranking and location are fetched from the table and stored as a csv file. Jupyter notebook is available as well through this [link](https://github.com/penguinwang96825/QS_ranking_web_scraping/blob/master/QS.ipynb).

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

## Main code
Only 25 universities are listed on the table per page, so I have to set the number of page I want to crawl. (max page: 40)

1. Open the html via chrome driver. (make sure `webdriver.exe` is in the same directory)
2. Parse the html using `BeautifulSoup`.
3. Create a loop to crawl all the elements (ranking, uni name, location) in each row.
4. Click to the next page and start over the loop in step three.
5. Stop fetching the data until all pages are done.

```python
def get_uni_information(year=2020, unilist, page=40):
    url = r"https://www.topuniversities.com/university-rankings/world-university-rankings/{}".format(year)
    # Open url and get the QS Ranking html page
    driver_path = r"C:\Users\YangWang\Desktop\machineLearning\indiaNewsClassification\chromedriver.exe"
    driver = webdriver.Chrome(driver_path)
    time.sleep(2)
    driver.get(url)
    time.sleep(5)
    
    # Crawl all the pages (max page is 40)
    if page <=40: 
        for _ in range(int(page)):
            # Use BeautifulSoup to parse every page
            soup = BeautifulSoup(driver.page_source, "html.parser")
            # Find the table which contains the ranking information of every universities
            x = soup.find(name="table", attrs={"class": "dataTable no-footer"})
            # Use 'for' loop to catch every rows in the table, and append the rows into the list
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
        else:
            print("Max page is 40.")
    
    driver.quit()
    return unilist
```

## Get the dataframe
Using `get_uni_information` function to crawl the information and then store them into a dataframe.
Also do some dataframe preprocessing in order to make sure every columns are in right data types.

```python
def get_qs_ranking_dataframe(year=2020, page=40):
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
### Take a look at the dataframe
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

## Visualise top universities
Visualise top `top_ranking` universities and show top `num` countries in the image of the certain year.

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

