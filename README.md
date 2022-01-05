# AWS-Redshift-Data-Migration
With this small scale, simple example, we explore the ability of transferring data from one database to another using python!

# Connect to an existing Google Cloud DB


```python
# Dependencies
import pandas as pd
import os
import glob
import numpy as np

pd.set_option('display.max_rows', 50000)
pd.set_option('display.max_columns', 50000)

# Connect to Google Cloud DB
try:
    engine = create_engine('')
    conn = engine.connect()
except Exception as e:
    print(e)
    
# Use the coviddb DB
conn.execute('USE coviddb')
```




    <sqlalchemy.engine.cursor.LegacyCursorResult at 0x1186e02b0>




```python
# Preview Tables in the DB
result = conn.execute('show tables;')
tables = pd.DataFrame(result.fetchall())
tables
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>country_wise_latest</td>
    </tr>
    <tr>
      <th>1</th>
      <td>covid_19_clean_complete</td>
    </tr>
    <tr>
      <th>2</th>
      <td>day_wise</td>
    </tr>
    <tr>
      <th>3</th>
      <td>full_grouped</td>
    </tr>
    <tr>
      <th>4</th>
      <td>usa_county_wise</td>
    </tr>
    <tr>
      <th>5</th>
      <td>worldometer_data</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Write the raw SQL query
query = '''select * from country_wise_latest;'''

ResultProxy = conn.execute(query)
ResultSet = ResultProxy.fetchall()

# Create pandas dataframe
df = pd.DataFrame(ResultSet)
df.columns = ResultSet[0].keys()

# Preview Dataframe
df.head(10)
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country/Region</th>
      <th>Confirmed</th>
      <th>Deaths</th>
      <th>Recovered</th>
      <th>Active</th>
      <th>New cases</th>
      <th>New deaths</th>
      <th>New recovered</th>
      <th>Deaths / 100 Cases</th>
      <th>Recovered / 100 Cases</th>
      <th>Deaths / 100 Recovered</th>
      <th>Confirmed last week</th>
      <th>1 week change</th>
      <th>1 week % increase</th>
      <th>WHO Region</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Afghanistan</td>
      <td>36263</td>
      <td>1269</td>
      <td>25198</td>
      <td>9796</td>
      <td>106</td>
      <td>10</td>
      <td>18</td>
      <td>3.50</td>
      <td>69.49</td>
      <td>5.04</td>
      <td>35526</td>
      <td>737</td>
      <td>2.07</td>
      <td>Eastern Mediterranean</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>4880</td>
      <td>144</td>
      <td>2745</td>
      <td>1991</td>
      <td>117</td>
      <td>6</td>
      <td>63</td>
      <td>2.95</td>
      <td>56.25</td>
      <td>5.25</td>
      <td>4171</td>
      <td>709</td>
      <td>17.00</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>27973</td>
      <td>1163</td>
      <td>18837</td>
      <td>7973</td>
      <td>616</td>
      <td>8</td>
      <td>749</td>
      <td>4.16</td>
      <td>67.34</td>
      <td>6.17</td>
      <td>23691</td>
      <td>4282</td>
      <td>18.07</td>
      <td>Africa</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Andorra</td>
      <td>907</td>
      <td>52</td>
      <td>803</td>
      <td>52</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>5.73</td>
      <td>88.53</td>
      <td>6.48</td>
      <td>884</td>
      <td>23</td>
      <td>2.60</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Angola</td>
      <td>950</td>
      <td>41</td>
      <td>242</td>
      <td>667</td>
      <td>18</td>
      <td>1</td>
      <td>0</td>
      <td>4.32</td>
      <td>25.47</td>
      <td>16.94</td>
      <td>749</td>
      <td>201</td>
      <td>26.84</td>
      <td>Africa</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Antigua and Barbuda</td>
      <td>86</td>
      <td>3</td>
      <td>65</td>
      <td>18</td>
      <td>4</td>
      <td>0</td>
      <td>5</td>
      <td>3.49</td>
      <td>75.58</td>
      <td>4.62</td>
      <td>76</td>
      <td>10</td>
      <td>13.16</td>
      <td>Americas</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Argentina</td>
      <td>167416</td>
      <td>3059</td>
      <td>72575</td>
      <td>91782</td>
      <td>4890</td>
      <td>120</td>
      <td>2057</td>
      <td>1.83</td>
      <td>43.35</td>
      <td>4.21</td>
      <td>130774</td>
      <td>36642</td>
      <td>28.02</td>
      <td>Americas</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Armenia</td>
      <td>37390</td>
      <td>711</td>
      <td>26665</td>
      <td>10014</td>
      <td>73</td>
      <td>6</td>
      <td>187</td>
      <td>1.90</td>
      <td>71.32</td>
      <td>2.67</td>
      <td>34981</td>
      <td>2409</td>
      <td>6.89</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Australia</td>
      <td>15303</td>
      <td>167</td>
      <td>9311</td>
      <td>5825</td>
      <td>368</td>
      <td>6</td>
      <td>137</td>
      <td>1.09</td>
      <td>60.84</td>
      <td>1.79</td>
      <td>12428</td>
      <td>2875</td>
      <td>23.13</td>
      <td>Western Pacific</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Austria</td>
      <td>20558</td>
      <td>713</td>
      <td>18246</td>
      <td>1599</td>
      <td>86</td>
      <td>1</td>
      <td>37</td>
      <td>3.47</td>
      <td>88.75</td>
      <td>3.91</td>
      <td>19743</td>
      <td>815</td>
      <td>4.13</td>
      <td>Europe</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Filter and Obtain specific data
df2 = df.loc[df['WHO Region'].isin(['Europe','Africa'])]
df2.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country/Region</th>
      <th>Confirmed</th>
      <th>Deaths</th>
      <th>Recovered</th>
      <th>Active</th>
      <th>New cases</th>
      <th>New deaths</th>
      <th>New recovered</th>
      <th>Deaths / 100 Cases</th>
      <th>Recovered / 100 Cases</th>
      <th>Deaths / 100 Recovered</th>
      <th>Confirmed last week</th>
      <th>1 week change</th>
      <th>1 week % increase</th>
      <th>WHO Region</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Albania</td>
      <td>4880</td>
      <td>144</td>
      <td>2745</td>
      <td>1991</td>
      <td>117</td>
      <td>6</td>
      <td>63</td>
      <td>2.95</td>
      <td>56.25</td>
      <td>5.25</td>
      <td>4171</td>
      <td>709</td>
      <td>17.00</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Algeria</td>
      <td>27973</td>
      <td>1163</td>
      <td>18837</td>
      <td>7973</td>
      <td>616</td>
      <td>8</td>
      <td>749</td>
      <td>4.16</td>
      <td>67.34</td>
      <td>6.17</td>
      <td>23691</td>
      <td>4282</td>
      <td>18.07</td>
      <td>Africa</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Andorra</td>
      <td>907</td>
      <td>52</td>
      <td>803</td>
      <td>52</td>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>5.73</td>
      <td>88.53</td>
      <td>6.48</td>
      <td>884</td>
      <td>23</td>
      <td>2.60</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Angola</td>
      <td>950</td>
      <td>41</td>
      <td>242</td>
      <td>667</td>
      <td>18</td>
      <td>1</td>
      <td>0</td>
      <td>4.32</td>
      <td>25.47</td>
      <td>16.94</td>
      <td>749</td>
      <td>201</td>
      <td>26.84</td>
      <td>Africa</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Armenia</td>
      <td>37390</td>
      <td>711</td>
      <td>26665</td>
      <td>10014</td>
      <td>73</td>
      <td>6</td>
      <td>187</td>
      <td>1.90</td>
      <td>71.32</td>
      <td>2.67</td>
      <td>34981</td>
      <td>2409</td>
      <td>6.89</td>
      <td>Europe</td>
    </tr>
  </tbody>
</table>
</div>



# Connect to Redshift

[Connecting to AWS Redshift](https://github.com/aws/amazon-redshift-python-driver/blob/master/tutorials/001%20-%20Connecting%20to%20Amazon%20Redshift.ipynb)

[AWS Connection Github](https://github.com/MyBusinessMaterialLinks/Youtube/blob/main/Connecting%20to%20AWS%20Redshift%20via%20Jupyter/Connecting%20to%20AWS%20Redshift.ipynb)

```python
import psycopg2
from sqlalchemy import create_engine
from sqlalchemy import text
```


```python
# Hard Coded Approach
endpoint = ''
user = ''
pass_ = ''
port = 
dbname = ''
```


```python
engine_string = "postgresql+psycopg2://%s:%s@%s:%d/%s" \
% (user, pass_, endpoint, port, dbname)
engine = create_engine(engine_string)
```


```python
# Add "Cleaned" Data to AWS using pandas
df2.to_sql('test',con = engine, if_exists='replace', index=False)
```
