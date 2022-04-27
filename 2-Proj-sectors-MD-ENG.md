# **Lima Stock Exchange Financial Performance (2015-2020)**

[**View the dashboard online in Tableau**](https://tabsoft.co/3sUUAek)

<center><a href="https://tabsoft.co/3sUUAek "><img src="https://imgur.com/8duoiFH.png" width="600"/></a></center>

***

## _Content_

* **[0. Business Problem](#Business-Problem)**
* **[1. Pre Processing Data](#-Pre-Processing-Data)**
  * [1.1. Data Source](##Data-Source)
  * [1.2. Data Extraction](##Data-Extraction)
* **[2. Data Cleaning](#Data-Cleaning)**
* **[3. Data Modeling](#Data-Modeling)**
* **[4. Data Visualization and Analysis](#Data-Visualization-and-Analysis)**
* **[5. Conclusions and Recommendations](#Conclusions-and-Recommendations)**

***

# Business Problem

Peru's stock market better known as the Lima Stock Exchange (BVL) is considered one of the smallest stock markets in Latin America despite its favorable growth in the last five years by doubling its public debt size from 8.1% in 2015 to 15.7% of GDP in 2019.

### ❓ **<ins>Data Analysis questions</ins>**:

* Do companies are able to raise funds and cover their obligations?
* Which sector shows higher profitability?
* Do companies with higher revenues generate better profit margins?

### 🎯 **<ins>Objective</ins>**: 

To provide an overview of the historical financial status of companies listed on the Lima Stock Exchange, this analysis will support you in making investment decisions.

<br/>

| Scope of the study | Type of analysis | Tools used |
| --- | :-: | --: |
| Longitudinal: From 2015 to 2020. |    Descriptive   | SQL - Python - Tableau |


#  Pre Processing Data

## Source of data

To perform the analysis, the financial information of the companies from 2015 to 2020, available on the [_Lima Stock Exchange website_](https://www.bvl.com.pe/emisores/listado-emisores), was compiled. The financial statements used were the Income Statement and Statement of Financial Position. In addition, to avoid problems with the calculation of indicators, companies that did not present information during this time interval were discarded so that they are homogeneous and comparable. Atria Energia, Medrock, Minera IRL Limited were discarded.

The financial ratios calculated to answer the questions were:

1. **Liquidity Ratio | Current ratio(CR):** Indicates companies' ability to raise funds and cover their obligations (Current Assets / Current Liabilities).
2. **Debt ratio (DR):** Measures how well a company's cash flow can cover its long-term debt (Total liabilities / Total assets).
3. **Return on Assets (ROA):** Measures how profitable a company is in relation to its total assets (Net Income / Total Assets).
4. **Net margin (MN):** Measures how much net income or profit is generated from sales revenue (Net profit / Net sales).

## Data Extraction

For the extraction we used scraping in Python with the Selenium environment and the Pandas library**(see Python code** [**here.**](https://github.com/Lu-Emperatriz/Lima-Stock-Performance/blob/main/pythonScript.ipynb)).

The dataset contains the following attributes:

| Ratio      |                                    Description                                    |
| ---------- | :-------------------------------------------------------------------------------: |
| TASSET     |                                    Total assets                                   |
| TLIABILITY |                                 Total liabilities                                 |
| CURR_ASS   |                                   Current Assets                                  |
| CURR_LIAB  |                                Current liabilities                                |
| NET_SALES  |                            Net sales is total revenues                            |
| NET_PROF   | Net earnings or net profit, after subtracting all interest, operations and taxes. |

# Data Cleanup

A **total of six tables** were exported:

* _'SECTOR'_ with records of the sector to which each company belongs and the URL of the information.
* _'TAB15'_ containing data from 2015, _'TAB16'_ with data from the 2016 period, and likewise for the remaining tables _'TAB17', 'TAB18', 'TAB19'_ and _'TAB20'_.

Then an _ID identity_ was added to identify the companies and the corresponding year.

```sql
--1. Adding identify ID to each table (the same for the others)
ALTER TABLE TAB15
ADD ID int identity(1,1)
GO
--2. Adding year number to each table (the same for the others)
ALTER TABLE TAB15 ADD year integer
UPDATE TAB15 SET year = 2015
```

Finally, the tables were joined in a new view, which includes the sector to which each company belongs and the calculation of the corresponding ratios.

Let's see the code!

```sql
--1. UNION all tables (alias 'a' for TAB15, 'b' for TAB16, ... and 'f' for TAB20)
--2. FORMULAS CALCULATION (where 'CR' is the alias for Current Ratio, 'DB' is Debt Ratio and 'NM' is Net Maergin)

SELECT a.id, a.year, z.NAME, z.sec, a.TASSET, a.TLIABILITY, a.CURR_ASS, a.CURR_LIAB, a.NET_SALES, a.NET_PROF
	,a.CURR_ASS/a.CURR_LIAB AS CR, a.TLIABILITY/a.TASSET AS DR, a.NET_PROF/a.TASSET AS ROA,  a.NET_SALES/a.NET_PROF AS NM
FROM TAB15 a, SECTOR z	WHERE a.id = z.ID  
UNION ALL 	(

SELECT b.id , b.year, z.NAME, z.sec, b.TASSET, b.TLIABILITY , b.CURR_ASS, b.CURR_LIAB, b.NET_SALES, b.NET_PROF
	,b.CURR_ASS/b.CURR_LIAB AS CR, b.TLIABILITY/b.TASSET AS DR, b.NET_PROF/b.TASSET AS ROA,  b.NET_SALES/b.NET_PROF AS NM
FROM TAB16 b, SECTOR z 	WHERE b.id = z.ID
UNION ALL 	(

SELECT d.id , d.year, z.NAME, z.sec, d.TASSET, d.TLIABILITY, d.CURR_ASS, d.CURR_LIAB, d.NET_SALES, d.NET_PROF
	,d.CURR_ASS/d.CURR_LIAB AS CR, d.TLIABILITY/d.TASSET AS DR, d.NET_PROF/d.TASSET AS ROA,  d.NET_SALES/d.NET_PROF AS NM
FROM TAB18 d, SECTOR z 	WHERE d.id = z.ID
UNION ALL 	(

SELECT c.id , c.year, z.NAME, z.sec, c.TASSET, c.TLIABILITY, c.CURR_ASS, c.CURR_LIAB, c.NET_SALES, c.NET_PROF
	,c.CURR_ASS/c.CURR_LIAB AS CR, c.TLIABILITY/c.TASSET AS DR, c.NET_PROF/c.TASSET AS ROA,  c.NET_SALES/c.NET_PROF AS NM
FROM TAB17 c, SECTOR z	WHERE c.id = z.ID)
UNION ALL 	(

SELECT e.id , e.year, z.NAME, z.sec, e.TASSET, e.TLIABILITY, e.CURR_ASS, e.CURR_LIAB, e.NET_SALES, e.NET_PROF
	,e.CURR_ASS/e.CURR_LIAB AS CR, e.TLIABILITY/e.TASSET AS DR, e.NET_PROF/e.TASSET AS ROA,  e.NET_SALES/e.NET_PROF AS NM
FROM TAB19 e, SECTOR z	WHERE e.id = z.ID
UNION ALL 	(

SELECT f.id , f.year, z.NAME, z.sec, f.TASSET, f.TLIABILITY, f.CURR_ASS, f.CURR_LIAB, f.NET_SALES, f.NET_PROF
	,f.CURR_ASS/f.CURR_LIAB AS CR, f.TLIABILITY/f.TASSET AS DR, f.NET_PROF/f.TASSET AS ROA,  f.NET_SALES/f.NET_PROF AS NM
FROM TAB20 f, SECTOR z	WHERE f.id = z.ID	))))

ORDER BY z.NAME, year
```

# Data Modeling

💡 Data modeling connects the different databases extracted in order to structure and organize the data for better understanding.

First the _ID_ column in each table was established as the _primary key_, this 'key' is the attribute that concerns the two tables to be related. The final structure is shown below:

<center><img src="https://imgur.com/Q91NPpo.png" width="300" height="400"/></center>

# Data Visualization and Analysis

For a better understanding of the data a dynamic dashboard was built in Tableau.<br/>Try it out [**HERE**](https://tabsoft.co/3sUUAek).

✔️ **Do companies have the capacity to raise funds and cover their obligations?**

The LIQUIDITY RATIO BY SECTOR boxplot reflects the ability of companies in the Industrials, Mining and Agriculture sectors to meet their short-term debts, as approximately 75% (bottom quartile) of companies have a current ratio greater than 1. On the other hand, the Services sector has an average liquidity of 1.3, but 50% of their companies have negative value.

<center><img src="https://imgur.com/ihmlwSV.png" widht="270"/></center>

On the other hand, the positive trend line shown in the DEBT FINANCING graph shows that companies prefer to finance their activities with external financing rather than their own funds, with the exception of the Mining sector. It can be deduced that this decision regarding financing depends on the specific productive activities of each company.

<center><img src="https://imgur.com/QmbGhZ8.png" widht="300"/></center>

✔️ **Which sector shows the highest profitability?**

ROA indicates how efficient a company is in generating profits with its available assets; for this analysis, ROA values exceeding 5% are considered optimal. Since the value of ROA varies according to the industry, it was analyzed taking into account the period of time.

<center><img src="https://imgur.com/gSqA0fo.png" widht="250"/></center>

In the Industrial and Mining Sector an inefficiency in the use of assets is noticed, since their profits have a tendency to decrease with the passage of time as shown in the ROA EVOLUTION visualization.

✔️ **Do companies with higher revenues generate better profit margins?**

No. The NET PROFIT MARGIN graph, indicates that the higher net margin does not correspond to companies that generate more sales, on the contrary, small companies such as "Cementos Pacasmayo" have higher profit margins than large companies such as "Petroperu".

<center><img src="https://imgur.com/XSmTDIn.png" height="400"/></center>

# Conclusions and Recommendations

The results found in this study serve as a reference for investors when making decisions regarding their stock portfolio. From this study it can be concluded that:

* The Mining sector stands out above the others due to its liquidity value and capacity to cover its obligations, however, from 201 to 2020 a drop in its profitability was observed.
* The largest companies are not necessarily the most efficient in their resources, possibly due to the requirement of higher operational costs.
* The value of the indicators will depend on the ratio and the sector, so it is recommended to analyze the companies' books of accounts in detail.

Interested parties are invited to perform a more comprehensive analysis by making proper use of this tool to make their decisions according to their personal objectives.

***

### **📌 Learn more about my projects [HERE](https://github.com/Lu-Emperatriz)**

Lucero Emperatriz.
 <a href="https://www.linkedin.com/in/lucero-sovero/"><img src="https://imgur.com/p58yPZr.png" height="auto" width="50" style="border-radius:50%"></a>