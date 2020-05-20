# Datastream Sustainable Development Goals Country Scores

Environmental, Social and Governance (ESG) data is difficult to come by. It is also becoming critical for effective investment analysis. It helps you assess the risks – and opportunities – posed by companies’ performance in critical areas such as climate change, executive remuneration, and diversity and inclusion. But a definite lack of transparency and standardization in such reporting presents major challenges for investors.
$$ \ $$
This article attempts to lay a framework to allow any investor/agent to collect, analyse and gather insight into countries' ESG metrics at granular and macro-levels.
$$ \\ $$
The file 'ESG-DS.csv' will be needed to collect individual country series codes from our ESG database.

$$ \\ $$
## Get Coding

### Import Libraries
math, statistics, numpy, pandas and openpyxl are needed for dataset manipulation and their statistical and mathematical manipulations.


```python
import math
import statistics
import numpy as np
```


```python
import pandas as pd
# This line will ensure that all columns of our data-frames are always shown:
pd.set_option('display.max_columns', None)
```


```python
# xlrd and openpyxl is needed to export data-frames to Excel files.
# The former doesn't need to be imported and can be installed via 'pip install xlrd'
import openpyxl
```

$$ \\ $$
pickle is only used here to record data so that is doesn't have to be collected every time this code is ran (if it is ran several times)


```python
# need to ' pip install pickle-mixin '
import pickle
```

$$ \\ $$
We need to gather our data. Since **Refinitiv's [DataStream](https://www.refinitiv.com/en/products/datastream-macroeconomic-analysis) Web Services (DSWS)** allows for access to ESG data covering nearly 70% of global market cap and over 400 metrics, naturally it is more than appropriate. We can access DSWS via the Python library "DatastreamDSWS" that can be installed simply by using $\textit{pip install}$.


```python
import DatastreamDSWS as DSWS

# We can use our Refinitiv's Datastream Web Socket (DSWS) API keys that allows us to be identified by
# Refinitiv's back-end services and enables us to request (and fetch) data:
# Credentials are placed in a text file so that it may be used in this code without showing it itself.
(DSWS_username, DSWS_password) = (open("Datastream_username.txt","r"),
                                  open("Datastream_password.txt","r"))

ds = DSWS.Datastream(username = str(DSWS_username.read()),
                     password = str(DSWS_password.read()))

# It is best to close the files we opened in order to make sure that we don't stop any other
# services/programs from accessing them if they need to.
DSWS_username.close()
DSWS_password.close()
```

$$ \\ $$
For full replication, note that the version of libraries used


```python
import sys # ' sys ' is only needed to display our Pyhon version
print("This code is running on Python version " + sys.version[0:5])
```

    This code is running on Python version 3.7.7
    


```python
for i,j in zip(["np", "pd", "openpyxl"], [np, pd, openpyxl]):
    print("The " + str(i) + " library imported in this code is version: " + j.__version__)
```

    The np library imported in this code is version: 1.18.2
    The pd library imported in this code is version: 1.0.3
    The openpyxl library imported in this code is version: 3.0.3
    

$$ \\ $$
The 'datetime' library is a Python-built-in library, therefore it does not have a specific version number.


```python
from datetime import date
```

$ \\ $
### Setup Functions

We rate each ESG category on a scale from 1 to 10 as per the function defined bellow


```python
def Point(value, # ' value ' as an integer or float between 0 and 1 (inclusive)
          polarity = "Positive"): # ' polarity ' informs us if we are grading the value as 'higher is better' (i.e.: positively polarised) or 'lower is better'
    
    if math.isnan(value):
        # This if function captures the eventuality when we don't have a value passed through this function
        result = np.nan
    elif value >= 0.9:
        result = 10
    elif value >= 0.8:
        result = 9
    elif value >= 0.7:
        result = 8
    elif value >= 0.6:
        result = 7
    elif value >= 0.5:
        result = 6
    elif value >= 0.4:
        result = 5
    elif value >= 0.3:
        result = 4
    elif value >= 0.2:
        result = 3
    elif value >= 0.1:
        result = 2
    elif value >= 0.0:
        result = 1
    
    if polarity == "Positive":
        # This function this far assumes positive polarity
        return result
    elif polarity == "Negative":
        # We now can look into negatively polarised data
        if math.isnan(value):
            return result
        elif result <= 2:
            return 10
        elif value >= 1:
            return 1
        else:
            return 12 - result
```

$$ \\ $$
### Collect Data

$$ \\ $$
We first need to collect the individual country series codes to ping DataStream with.
There are a great many of them, we thus collect them from the comma-separated values (csv) file 'ESG-DS.csv'.


```python
# This file includes polarity information as well as series codes, the ' .iloc[:,6:-1] ' bellow ensures that we only collect the latter.
df = pd.read_csv("ESG-DS.csv", header = 1, index_col = 6).iloc[:,6:-1]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>1.1.1</th>
      <th>1.1.1.1</th>
      <th>1.2.1</th>
      <th>1.2.2</th>
      <th>1.3.1</th>
      <th>1.5.3</th>
      <th>1.A.1</th>
      <th>1.A.2</th>
      <th>1.A.2.1</th>
      <th>1.B.1</th>
      <th>2.1.1</th>
      <th>2.1.2</th>
      <th>2.2.1</th>
      <th>2.2.2</th>
      <th>2.2.2.1</th>
      <th>2.A.1</th>
      <th>2.A.2</th>
      <th>3.1.1</th>
      <th>3.1.2</th>
      <th>3.2.1</th>
      <th>3.2.2</th>
      <th>3.3.1</th>
      <th>3.3.1.1</th>
      <th>3.3.1.2</th>
      <th>3.3.2</th>
      <th>3.7.2</th>
      <th>3.8.2</th>
      <th>3.9.2</th>
      <th>3.A.1</th>
      <th>3.A.1.1</th>
      <th>3.C.1</th>
      <th>4.1.1</th>
      <th>4.1.1.1</th>
      <th>4.1.1.2</th>
      <th>4.1.1.3</th>
      <th>4.1.1.4</th>
      <th>4.1.1.5</th>
      <th>4.4.1</th>
      <th>4.5.1</th>
      <th>4.6.1</th>
      <th>4.6.1.1</th>
      <th>4.6.1.2</th>
      <th>4.6.1.3</th>
      <th>4.6.1.4</th>
      <th>4.6.1.5</th>
      <th>4.A.1</th>
      <th>5.1.1</th>
      <th>5.5.1</th>
      <th>5.6.1</th>
      <th>5.6.1.1</th>
      <th>5.B.1</th>
      <th>5.B.1.1</th>
      <th>6.1.1</th>
      <th>6.2.1</th>
      <th>6.4.2</th>
      <th>7.1.1</th>
      <th>7.1.2</th>
      <th>7.2.1</th>
      <th>7.2.1.1</th>
      <th>7.3.1</th>
      <th>7.3.1.1</th>
      <th>8.1.1</th>
      <th>8.2.1</th>
      <th>8.2.1.1</th>
      <th>8.2.1.2</th>
      <th>8.5.2</th>
      <th>8.5.2.1</th>
      <th>8.5.2.2</th>
      <th>8.5.2.3</th>
      <th>8.5.2.4</th>
      <th>8.5.2.5</th>
      <th>8.6.1</th>
      <th>8.6.1.1</th>
      <th>8.6.1.2</th>
      <th>8.7.1</th>
      <th>8.7.1.1</th>
      <th>8.7.1.2</th>
      <th>8.7.1.3</th>
      <th>8.7.1.4</th>
      <th>8.7.1.5</th>
      <th>8.7.1.6</th>
      <th>8.7.1.7</th>
      <th>8.7.1.8</th>
      <th>8.8.2</th>
      <th>8.10.1</th>
      <th>8.10.1.1</th>
      <th>8.B.1</th>
      <th>9.1.2.</th>
      <th>9.1.2..1</th>
      <th>9.1.2..2</th>
      <th>9.2.1.</th>
      <th>9.2.1..1</th>
      <th>9.2.1..2</th>
      <th>9.2.2.</th>
      <th>9.4.1</th>
      <th>9.5.1</th>
      <th>9.5.2</th>
      <th>9.C.1</th>
      <th>10.1.1</th>
      <th>10.1.1.1</th>
      <th>10.2.1</th>
      <th>10.3.1</th>
      <th>10.3.1.1</th>
      <th>10.3.1.2</th>
      <th>10.3.1.3</th>
      <th>10.4.1</th>
      <th>10.5.1</th>
      <th>10.5.1.1</th>
      <th>10.5.1.2</th>
      <th>11.6.2</th>
      <th>11.6.2.1</th>
      <th>12.7.1</th>
      <th>12.C.1</th>
      <th>13.2.1</th>
      <th>13.2.1.1</th>
      <th>15.1.1</th>
      <th>15.1.2</th>
      <th>15.1.2.1</th>
      <th>15.1.2.2</th>
      <th>15.2.1</th>
      <th>15.3.1</th>
      <th>15.5.1</th>
      <th>15.9.1</th>
      <th>15.9.1.1</th>
      <th>15.9.1.2</th>
      <th>15.9.1.3</th>
      <th>15.A.1</th>
      <th>15.A.1.1</th>
      <th>15.B.1</th>
      <th>15.B.1.1</th>
      <th>16.1.1</th>
      <th>16.1.2</th>
      <th>16.1.2.1</th>
      <th>16.2.2</th>
      <th>16.4.1</th>
      <th>16.4.2</th>
      <th>16.5.1</th>
      <th>16.5.1.1</th>
      <th>16.5.2</th>
      <th>16.5.2.1</th>
      <th>16.6.2</th>
      <th>16.9.1</th>
      <th>16.10.2</th>
      <th>16.A.1</th>
      <th>16.B.1</th>
      <th>16.B.1.1</th>
      <th>17.1.1</th>
      <th>17.1.1.1</th>
      <th>17.1.1.2</th>
      <th>17.1.1.3</th>
      <th>17.3.1</th>
      <th>17.4.1</th>
      <th>17.6.2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghanistan</th>
      <td>AFWD8I78R</td>
      <td>AFWDNYMYR</td>
      <td>AFWDYTADR</td>
      <td>AFACPOVBR</td>
      <td>AFWDUGUER</td>
      <td>AFWDA131R</td>
      <td>AFWDUGUER</td>
      <td>AFWDA903R</td>
      <td>AFWDQOG4R</td>
      <td>AFWDMD93R</td>
      <td>AFWDLBORR</td>
      <td>AFWDA378P</td>
      <td>AFWD1JCLR</td>
      <td>AFWD2KTMR</td>
      <td>AFWDYI32R</td>
      <td>AFGFF42G</td>
      <td>AFGFF42O</td>
      <td>AFWDPVWXR</td>
      <td>AFWD5MSAR</td>
      <td>AFWD7X4NP</td>
      <td>AFWDA141R</td>
      <td>AFWD153FR</td>
      <td>AFWD6MMLR</td>
      <td>AFWDHGSZR</td>
      <td>AFWDXB26P</td>
      <td>AFWDANDRP</td>
      <td>AFWDZMVPA</td>
      <td>AFWDZPVLR</td>
      <td>AFWDCU7JR</td>
      <td>AFWD1LNHR</td>
      <td>AFWD67W8P</td>
      <td>AFWDTMWJR</td>
      <td>AFWDTMWJR</td>
      <td>AFWDTMWJR</td>
      <td>AFWDTMWJR</td>
      <td>AFWDTMWJR</td>
      <td>AFWDTMWJR</td>
      <td>AFGCIBXV</td>
      <td>AFWDUGUER</td>
      <td>AFWDSWAWR</td>
      <td>AFWD3PM6R</td>
      <td>AFWDNNO1R</td>
      <td>AFWD24FSR</td>
      <td>AFWDGC8WR</td>
      <td>AFWDBSXAR</td>
      <td>AFWDA903R</td>
      <td>AFWDMD93R</td>
      <td>AFWDBIZXR</td>
      <td>AFWDOOGER</td>
      <td>AFWDLP15P</td>
      <td>AFWDDLO5P</td>
      <td>AFWD8FD7P</td>
      <td>AFWDZPVLR</td>
      <td>AFWDZPVLR</td>
      <td>AFWDL6Z8R</td>
      <td>AFWDA121R</td>
      <td>AFWD9I7VR</td>
      <td>AFWDA124R</td>
      <td>AFWD8S7HR</td>
      <td>AFXTPEN.</td>
      <td>AFWD2OO1A</td>
      <td>AFWDB1HMR</td>
      <td>AFWDWSO8R</td>
      <td>AFWD8FD7P</td>
      <td>AFWD3QBGR</td>
      <td>AFWDDK5XR</td>
      <td>AFWDAJTXR</td>
      <td>AFWDZP5NR</td>
      <td>AFWDFCM9R</td>
      <td>AFWD7YW0R</td>
      <td>AFWDIT9QR</td>
      <td>AFWDA443R</td>
      <td>AFWDA444R</td>
      <td>AFWDA445R</td>
      <td>AFWDVMS9R</td>
      <td>AFWDVM7CR</td>
      <td>AFWD6GL2R</td>
      <td>AFWDEZ1JR</td>
      <td>AFWDVVWFR</td>
      <td>AFWDL1BLR</td>
      <td>AFWDYPL5R</td>
      <td>AFWD3OFRR</td>
      <td>AFWDSUEDR</td>
      <td>AFGCWQYS</td>
      <td>AFWDA151P</td>
      <td>AFWDA039P</td>
      <td>AFGFF0XL</td>
      <td>AFWD64LUP</td>
      <td>AFWDXXOQP</td>
      <td>AFWD5U24P</td>
      <td>AFWDKN5RR</td>
      <td>AFWD7MZ1A</td>
      <td>AFWD8FD7P</td>
      <td>AFWDL0XWR</td>
      <td>AFWDIVNLP</td>
      <td>AFWDQ4L5R</td>
      <td>AFWDALCKR</td>
      <td>AFWDGJ88P</td>
      <td>AFWDA432R</td>
      <td>AFWDA433R</td>
      <td>AFWDFW5SF</td>
      <td>AFWDMD93R</td>
      <td>AFIFCLPNR</td>
      <td>AFCRCPRSR</td>
      <td>AFCRECOSR</td>
      <td>AFPWLBSH</td>
      <td>AFCRILLBR</td>
      <td>AFWD7OM6R</td>
      <td>AFAMTOTSR</td>
      <td>AFWDA441P</td>
      <td>AFWD7TPDP</td>
      <td>AFWDUYDZR</td>
      <td>AFWDOS8LR</td>
      <td>AFWDIVNLP</td>
      <td>AFWDLIKMP</td>
      <td>AFWDE0D5R</td>
      <td>AFWDA150R</td>
      <td>AFWDY9K8R</td>
      <td>AFWD1OBUR</td>
      <td>AFWDA075R</td>
      <td>AFWDODTKR</td>
      <td>AFWDJZYDR</td>
      <td>AFWDGHL4P</td>
      <td>AFWD3IS4P</td>
      <td>AFWD75RDP</td>
      <td>AFWDU54MP</td>
      <td>AFGFF54G</td>
      <td>AFGFF54O</td>
      <td>AFGFF54G</td>
      <td>AFGFF54O</td>
      <td>AFWDA160P</td>
      <td>AFWDA011P</td>
      <td>AFWD8FD7P</td>
      <td>AFCRHMTSR</td>
      <td>AFCRILLSR</td>
      <td>AFCRGEXSR</td>
      <td>AFTCPSCO</td>
      <td>AFACAMLSR</td>
      <td>AFTCPSCO</td>
      <td>AFACAMLSR</td>
      <td>AFCRGVESR</td>
      <td>AFWDCSMYR</td>
      <td>AFGCI9SS</td>
      <td>AFCRHMROR</td>
      <td>AFWDMD93R</td>
      <td>AFIFCLPNR</td>
      <td>AFGFREVO</td>
      <td>AFGFTRBO</td>
      <td>AFGFGRVO</td>
      <td>AFGFORVO</td>
      <td>AFINVR..</td>
      <td>AFWDVN9PR</td>
      <td>AFGCY4PR</td>
    </tr>
    <tr>
      <th>Albania</th>
      <td>ALWD8I78R</td>
      <td>ALWDNYMYR</td>
      <td>ALWDYTADR</td>
      <td>ALACPOVBR</td>
      <td>ALWDUGUER</td>
      <td>ALWDA131R</td>
      <td>ALWDUGUER</td>
      <td>ALWDA903R</td>
      <td>ALWDQOG4R</td>
      <td>ALWDMD93R</td>
      <td>ALWDLBORR</td>
      <td>ALWDA378P</td>
      <td>ALWD1JCLR</td>
      <td>ALWD2KTMR</td>
      <td>ALWDYI32R</td>
      <td>ALGFF42G</td>
      <td>ALGFF42O</td>
      <td>ALWDPVWXR</td>
      <td>ALWD5MSAR</td>
      <td>ALWD7X4NP</td>
      <td>ALWDA141R</td>
      <td>ALWD153FR</td>
      <td>ALWD6MMLR</td>
      <td>ALWDHGSZR</td>
      <td>ALWDXB26P</td>
      <td>ALWDANDRP</td>
      <td>ALWDZMVPA</td>
      <td>ALWDZPVLR</td>
      <td>ALWDCU7JR</td>
      <td>ALWD1LNHR</td>
      <td>ALWD67W8P</td>
      <td>ALWDTMWJR</td>
      <td>ALWDTMWJR</td>
      <td>ALWDTMWJR</td>
      <td>ALWDTMWJR</td>
      <td>ALWDTMWJR</td>
      <td>ALWDTMWJR</td>
      <td>ALGCIBXV</td>
      <td>ALWDUGUER</td>
      <td>ALWDSWAWR</td>
      <td>ALWD3PM6R</td>
      <td>ALWDNNO1R</td>
      <td>ALWD24FSR</td>
      <td>ALWDGC8WR</td>
      <td>ALWDBSXAR</td>
      <td>ALWDA903R</td>
      <td>ALWDMD93R</td>
      <td>ALWDBIZXR</td>
      <td>ALWDOOGER</td>
      <td>ALWDLP15P</td>
      <td>ALWDDLO5P</td>
      <td>ALWD8FD7P</td>
      <td>ALWDZPVLR</td>
      <td>ALWDZPVLR</td>
      <td>ALWDL6Z8R</td>
      <td>ALWDA121R</td>
      <td>ALWD9I7VR</td>
      <td>ALWDA124R</td>
      <td>ALWD8S7HR</td>
      <td>ALXTPEN.</td>
      <td>ALWD2OO1A</td>
      <td>ALWDB1HMR</td>
      <td>ALWDWSO8R</td>
      <td>ALWD8FD7P</td>
      <td>ALWD3QBGR</td>
      <td>ALWDDK5XR</td>
      <td>ALWDAJTXR</td>
      <td>ALWDZP5NR</td>
      <td>ALWDFCM9R</td>
      <td>ALWD7YW0R</td>
      <td>ALWDIT9QR</td>
      <td>ALWDA443R</td>
      <td>ALWDA444R</td>
      <td>ALWDA445R</td>
      <td>ALWDVMS9R</td>
      <td>ALWDVM7CR</td>
      <td>ALWD6GL2R</td>
      <td>ALWDEZ1JR</td>
      <td>ALWDVVWFR</td>
      <td>ALWDL1BLR</td>
      <td>ALWDYPL5R</td>
      <td>ALWD3OFRR</td>
      <td>ALWDSUEDR</td>
      <td>ALGCWQYS</td>
      <td>ALWDA151P</td>
      <td>ALWDA039P</td>
      <td>ALGFF0XL</td>
      <td>ALWD64LUP</td>
      <td>ALWDXXOQP</td>
      <td>ALWD5U24P</td>
      <td>ALWDKN5RR</td>
      <td>ALWD7MZ1A</td>
      <td>ALWD8FD7P</td>
      <td>ALWDL0XWR</td>
      <td>ALWDIVNLP</td>
      <td>ALWDQ4L5R</td>
      <td>ALWDALCKR</td>
      <td>ALWDGJ88P</td>
      <td>ALWDA432R</td>
      <td>ALWDA433R</td>
      <td>ALWDFW5SF</td>
      <td>ALWDMD93R</td>
      <td>ALIFCLPNR</td>
      <td>ALCRCPRSR</td>
      <td>ALCRECOSR</td>
      <td>ALPWLBSH</td>
      <td>ALCRILLBR</td>
      <td>ALWD7OM6R</td>
      <td>ALAMTOTSR</td>
      <td>ALWDA441P</td>
      <td>ALWD7TPDP</td>
      <td>ALWDUYDZR</td>
      <td>ALWDOS8LR</td>
      <td>ALWDIVNLP</td>
      <td>ALWDLIKMP</td>
      <td>ALWDE0D5R</td>
      <td>ALWDA150R</td>
      <td>ALWDY9K8R</td>
      <td>ALWD1OBUR</td>
      <td>ALWDA075R</td>
      <td>ALWDODTKR</td>
      <td>ALWDJZYDR</td>
      <td>ALWDGHL4P</td>
      <td>ALWD3IS4P</td>
      <td>ALWD75RDP</td>
      <td>ALWDU54MP</td>
      <td>ALGFF54G</td>
      <td>ALGFF54O</td>
      <td>ALGFF54G</td>
      <td>ALGFF54O</td>
      <td>ALWDA160P</td>
      <td>ALWDA011P</td>
      <td>ALWD8FD7P</td>
      <td>ALCRHMTSR</td>
      <td>ALCRILLSR</td>
      <td>ALCRGEXSR</td>
      <td>ALTCPSCO</td>
      <td>ALACAMLSR</td>
      <td>ALTCPSCO</td>
      <td>ALACAMLSR</td>
      <td>ALCRGVESR</td>
      <td>ALWDCSMYR</td>
      <td>ALGCI9SS</td>
      <td>ALCRHMROR</td>
      <td>ALWDMD93R</td>
      <td>ALIFCLPNR</td>
      <td>ALGFREVO</td>
      <td>ALGFTRBO</td>
      <td>ALGFGRVO</td>
      <td>ALGFORVO</td>
      <td>ALINVR..</td>
      <td>ALWDVN9PR</td>
      <td>ALGCY4PR</td>
    </tr>
    <tr>
      <th>Algeria</th>
      <td>AAWD8I78R</td>
      <td>AAWDNYMYR</td>
      <td>AAWDYTADR</td>
      <td>AAACPOVBR</td>
      <td>AAWDUGUER</td>
      <td>AAWDA131R</td>
      <td>AAWDUGUER</td>
      <td>AAWDA903R</td>
      <td>AAWDQOG4R</td>
      <td>AAWDMD93R</td>
      <td>AAWDLBORR</td>
      <td>AAWDA378P</td>
      <td>AAWD1JCLR</td>
      <td>AAWD2KTMR</td>
      <td>AAWDYI32R</td>
      <td>AAGFF42G</td>
      <td>AAGFF42O</td>
      <td>AAWDPVWXR</td>
      <td>AAWD5MSAR</td>
      <td>AAWD7X4NP</td>
      <td>AAWDA141R</td>
      <td>AAWD153FR</td>
      <td>AAWD6MMLR</td>
      <td>AAWDHGSZR</td>
      <td>AAWDXB26P</td>
      <td>AAWDANDRP</td>
      <td>AAWDZMVPA</td>
      <td>AAWDZPVLR</td>
      <td>AAWDCU7JR</td>
      <td>AAWD1LNHR</td>
      <td>AAWD67W8P</td>
      <td>AAWDTMWJR</td>
      <td>AAWDTMWJR</td>
      <td>AAWDTMWJR</td>
      <td>AAWDTMWJR</td>
      <td>AAWDTMWJR</td>
      <td>AAWDTMWJR</td>
      <td>AAGCIBXV</td>
      <td>AAWDUGUER</td>
      <td>AAWDSWAWR</td>
      <td>AAWD3PM6R</td>
      <td>AAWDNNO1R</td>
      <td>AAWD24FSR</td>
      <td>AAWDGC8WR</td>
      <td>AAWDBSXAR</td>
      <td>AAWDA903R</td>
      <td>AAWDMD93R</td>
      <td>AAWDBIZXR</td>
      <td>AAWDOOGER</td>
      <td>AAWDLP15P</td>
      <td>AAWDDLO5P</td>
      <td>AAWD8FD7P</td>
      <td>AAWDZPVLR</td>
      <td>AAWDZPVLR</td>
      <td>AAWDL6Z8R</td>
      <td>AAWDA121R</td>
      <td>AAWD9I7VR</td>
      <td>AAWDA124R</td>
      <td>AAWD8S7HR</td>
      <td>AAXTPEN.</td>
      <td>AAWD2OO1A</td>
      <td>AAWDB1HMR</td>
      <td>AAWDWSO8R</td>
      <td>AAWD8FD7P</td>
      <td>AAWD3QBGR</td>
      <td>AAWDDK5XR</td>
      <td>AAWDAJTXR</td>
      <td>AAWDZP5NR</td>
      <td>AAWDFCM9R</td>
      <td>AAWD7YW0R</td>
      <td>AAWDIT9QR</td>
      <td>AAWDA443R</td>
      <td>AAWDA444R</td>
      <td>AAWDA445R</td>
      <td>AAWDVMS9R</td>
      <td>AAWDVM7CR</td>
      <td>AAWD6GL2R</td>
      <td>AAWDEZ1JR</td>
      <td>AAWDVVWFR</td>
      <td>AAWDL1BLR</td>
      <td>AAWDYPL5R</td>
      <td>AAWD3OFRR</td>
      <td>AAWDSUEDR</td>
      <td>AAGCWQYS</td>
      <td>AAWDA151P</td>
      <td>AAWDA039P</td>
      <td>AAGFF0XL</td>
      <td>AAWD64LUP</td>
      <td>AAWDXXOQP</td>
      <td>AAWD5U24P</td>
      <td>AAWDKN5RR</td>
      <td>AAWD7MZ1A</td>
      <td>AAWD8FD7P</td>
      <td>AAWDL0XWR</td>
      <td>AAWDIVNLP</td>
      <td>AAWDQ4L5R</td>
      <td>AAWDALCKR</td>
      <td>AAWDGJ88P</td>
      <td>AAWDA432R</td>
      <td>AAWDA433R</td>
      <td>AAWDFW5SF</td>
      <td>AAWDMD93R</td>
      <td>AAIFCLPNR</td>
      <td>AACRCPRSR</td>
      <td>AACRECOSR</td>
      <td>AAPWLBSH</td>
      <td>AACRILLBR</td>
      <td>AAWD7OM6R</td>
      <td>AAAMTOTSR</td>
      <td>AAWDA441P</td>
      <td>AAWD7TPDP</td>
      <td>AAWDUYDZR</td>
      <td>AAWDOS8LR</td>
      <td>AAWDIVNLP</td>
      <td>AAWDLIKMP</td>
      <td>AAWDE0D5R</td>
      <td>AAWDA150R</td>
      <td>AAWDY9K8R</td>
      <td>AAWD1OBUR</td>
      <td>AAWDA075R</td>
      <td>AAWDODTKR</td>
      <td>AAWDJZYDR</td>
      <td>AAWDGHL4P</td>
      <td>AAWD3IS4P</td>
      <td>AAWD75RDP</td>
      <td>AAWDU54MP</td>
      <td>AAGFF54G</td>
      <td>AAGFF54O</td>
      <td>AAGFF54G</td>
      <td>AAGFF54O</td>
      <td>AAWDA160P</td>
      <td>AAWDA011P</td>
      <td>AAWD8FD7P</td>
      <td>AACRHMTSR</td>
      <td>AACRILLSR</td>
      <td>AACRGEXSR</td>
      <td>AATCPSCO</td>
      <td>AAACAMLSR</td>
      <td>AATCPSCO</td>
      <td>AAACAMLSR</td>
      <td>AACRGVESR</td>
      <td>AAWDCSMYR</td>
      <td>AAGCI9SS</td>
      <td>AACRHMROR</td>
      <td>AAWDMD93R</td>
      <td>AAIFCLPNR</td>
      <td>AAGFREVO</td>
      <td>AAGFTRBO</td>
      <td>AAGFGRVO</td>
      <td>AAGFORVO</td>
      <td>AAINVR..</td>
      <td>AAWDVN9PR</td>
      <td>AAGCY4PR</td>
    </tr>
    <tr>
      <th>American Samoa</th>
      <td>SMWD8I78R</td>
      <td>SMWDNYMYR</td>
      <td>SMWDYTADR</td>
      <td>SMACPOVBR</td>
      <td>SMWDUGUER</td>
      <td>SMWDA131R</td>
      <td>SMWDUGUER</td>
      <td>SMWDA903R</td>
      <td>SMWDQOG4R</td>
      <td>SMWDMD93R</td>
      <td>SMWDLBORR</td>
      <td>SMWDA378P</td>
      <td>SMWD1JCLR</td>
      <td>SMWD2KTMR</td>
      <td>SMWDYI32R</td>
      <td>SMGFF42G</td>
      <td>SMGFF42O</td>
      <td>SMWDPVWXR</td>
      <td>SMWD5MSAR</td>
      <td>SMWD7X4NP</td>
      <td>SMWDA141R</td>
      <td>SMWD153FR</td>
      <td>SMWD6MMLR</td>
      <td>SMWDHGSZR</td>
      <td>SMWDXB26P</td>
      <td>SMWDANDRP</td>
      <td>SMWDZMVPA</td>
      <td>SMWDZPVLR</td>
      <td>SMWDCU7JR</td>
      <td>SMWD1LNHR</td>
      <td>SMWD67W8P</td>
      <td>SMWDTMWJR</td>
      <td>SMWDTMWJR</td>
      <td>SMWDTMWJR</td>
      <td>SMWDTMWJR</td>
      <td>SMWDTMWJR</td>
      <td>SMWDTMWJR</td>
      <td>SMGCIBXV</td>
      <td>SMWDUGUER</td>
      <td>SMWDSWAWR</td>
      <td>SMWD3PM6R</td>
      <td>SMWDNNO1R</td>
      <td>SMWD24FSR</td>
      <td>SMWDGC8WR</td>
      <td>SMWDBSXAR</td>
      <td>SMWDA903R</td>
      <td>SMWDMD93R</td>
      <td>SMWDBIZXR</td>
      <td>SMWDOOGER</td>
      <td>SMWDLP15P</td>
      <td>SMWDDLO5P</td>
      <td>SMWD8FD7P</td>
      <td>SMWDZPVLR</td>
      <td>SMWDZPVLR</td>
      <td>SMWDL6Z8R</td>
      <td>SMWDA121R</td>
      <td>SMWD9I7VR</td>
      <td>SMWDA124R</td>
      <td>SMWD8S7HR</td>
      <td>SMXTPEN.</td>
      <td>SMWD2OO1A</td>
      <td>SMWDB1HMR</td>
      <td>SMWDWSO8R</td>
      <td>SMWD8FD7P</td>
      <td>SMWD3QBGR</td>
      <td>SMWDDK5XR</td>
      <td>SMWDAJTXR</td>
      <td>SMWDZP5NR</td>
      <td>SMWDFCM9R</td>
      <td>SMWD7YW0R</td>
      <td>SMWDIT9QR</td>
      <td>SMWDA443R</td>
      <td>SMWDA444R</td>
      <td>SMWDA445R</td>
      <td>SMWDVMS9R</td>
      <td>SMWDVM7CR</td>
      <td>SMWD6GL2R</td>
      <td>SMWDEZ1JR</td>
      <td>SMWDVVWFR</td>
      <td>SMWDL1BLR</td>
      <td>SMWDYPL5R</td>
      <td>SMWD3OFRR</td>
      <td>SMWDSUEDR</td>
      <td>SMGCWQYS</td>
      <td>SMWDA151P</td>
      <td>SMWDA039P</td>
      <td>SMGFF0XL</td>
      <td>SMWD64LUP</td>
      <td>SMWDXXOQP</td>
      <td>SMWD5U24P</td>
      <td>SMWDKN5RR</td>
      <td>SMWD7MZ1A</td>
      <td>SMWD8FD7P</td>
      <td>SMWDL0XWR</td>
      <td>SMWDIVNLP</td>
      <td>SMWDQ4L5R</td>
      <td>SMWDALCKR</td>
      <td>SMWDGJ88P</td>
      <td>SMWDA432R</td>
      <td>SMWDA433R</td>
      <td>SMWDFW5SF</td>
      <td>SMWDMD93R</td>
      <td>SMIFCLPNR</td>
      <td>SMCRCPRSR</td>
      <td>SMCRECOSR</td>
      <td>SMPWLBSH</td>
      <td>SMCRILLBR</td>
      <td>SMWD7OM6R</td>
      <td>SMAMTOTSR</td>
      <td>SMWDA441P</td>
      <td>SMWD7TPDP</td>
      <td>SMWDUYDZR</td>
      <td>SMWDOS8LR</td>
      <td>SMWDIVNLP</td>
      <td>SMWDLIKMP</td>
      <td>SMWDE0D5R</td>
      <td>SMWDA150R</td>
      <td>SMWDY9K8R</td>
      <td>SMWD1OBUR</td>
      <td>SMWDA075R</td>
      <td>SMWDODTKR</td>
      <td>SMWDJZYDR</td>
      <td>SMWDGHL4P</td>
      <td>SMWD3IS4P</td>
      <td>SMWD75RDP</td>
      <td>SMWDU54MP</td>
      <td>SMGFF54G</td>
      <td>SMGFF54O</td>
      <td>SMGFF54G</td>
      <td>SMGFF54O</td>
      <td>SMWDA160P</td>
      <td>SMWDA011P</td>
      <td>SMWD8FD7P</td>
      <td>SMCRHMTSR</td>
      <td>SMCRILLSR</td>
      <td>SMCRGEXSR</td>
      <td>SMTCPSCO</td>
      <td>SMACAMLSR</td>
      <td>SMTCPSCO</td>
      <td>SMACAMLSR</td>
      <td>SMCRGVESR</td>
      <td>SMWDCSMYR</td>
      <td>SMGCI9SS</td>
      <td>SMCRHMROR</td>
      <td>SMWDMD93R</td>
      <td>SMIFCLPNR</td>
      <td>SMGFREVO</td>
      <td>SMGFTRBO</td>
      <td>SMGFGRVO</td>
      <td>SMGFORVO</td>
      <td>SMINVR..</td>
      <td>SMWDVN9PR</td>
      <td>SMGCY4PR</td>
    </tr>
    <tr>
      <th>Andorra</th>
      <td>ADWD8I78R</td>
      <td>ADWDNYMYR</td>
      <td>ADWDYTADR</td>
      <td>ADACPOVBR</td>
      <td>ADWDUGUER</td>
      <td>ADWDA131R</td>
      <td>ADWDUGUER</td>
      <td>ADWDA903R</td>
      <td>ADWDQOG4R</td>
      <td>ADWDMD93R</td>
      <td>ADWDLBORR</td>
      <td>ADWDA378P</td>
      <td>ADWD1JCLR</td>
      <td>ADWD2KTMR</td>
      <td>ADWDYI32R</td>
      <td>ADGFF42G</td>
      <td>ADGFF42O</td>
      <td>ADWDPVWXR</td>
      <td>ADWD5MSAR</td>
      <td>ADWD7X4NP</td>
      <td>ADWDA141R</td>
      <td>ADWD153FR</td>
      <td>ADWD6MMLR</td>
      <td>ADWDHGSZR</td>
      <td>ADWDXB26P</td>
      <td>ADWDANDRP</td>
      <td>ADWDZMVPA</td>
      <td>ADWDZPVLR</td>
      <td>ADWDCU7JR</td>
      <td>ADWD1LNHR</td>
      <td>ADWD67W8P</td>
      <td>ADWDTMWJR</td>
      <td>ADWDTMWJR</td>
      <td>ADWDTMWJR</td>
      <td>ADWDTMWJR</td>
      <td>ADWDTMWJR</td>
      <td>ADWDTMWJR</td>
      <td>ADGCIBXV</td>
      <td>ADWDUGUER</td>
      <td>ADWDSWAWR</td>
      <td>ADWD3PM6R</td>
      <td>ADWDNNO1R</td>
      <td>ADWD24FSR</td>
      <td>ADWDGC8WR</td>
      <td>ADWDBSXAR</td>
      <td>ADWDA903R</td>
      <td>ADWDMD93R</td>
      <td>ADWDBIZXR</td>
      <td>ADWDOOGER</td>
      <td>ADWDLP15P</td>
      <td>ADWDDLO5P</td>
      <td>ADWD8FD7P</td>
      <td>ADWDZPVLR</td>
      <td>ADWDZPVLR</td>
      <td>ADWDL6Z8R</td>
      <td>ADWDA121R</td>
      <td>ADWD9I7VR</td>
      <td>ADWDA124R</td>
      <td>ADWD8S7HR</td>
      <td>ADXTPEN.</td>
      <td>ADWD2OO1A</td>
      <td>ADWDB1HMR</td>
      <td>ADWDWSO8R</td>
      <td>ADWD8FD7P</td>
      <td>ADWD3QBGR</td>
      <td>ADWDDK5XR</td>
      <td>ADWDAJTXR</td>
      <td>ADWDZP5NR</td>
      <td>ADWDFCM9R</td>
      <td>ADWD7YW0R</td>
      <td>ADWDIT9QR</td>
      <td>ADWDA443R</td>
      <td>ADWDA444R</td>
      <td>ADWDA445R</td>
      <td>ADWDVMS9R</td>
      <td>ADWDVM7CR</td>
      <td>ADWD6GL2R</td>
      <td>ADWDEZ1JR</td>
      <td>ADWDVVWFR</td>
      <td>ADWDL1BLR</td>
      <td>ADWDYPL5R</td>
      <td>ADWD3OFRR</td>
      <td>ADWDSUEDR</td>
      <td>ADGCWQYS</td>
      <td>ADWDA151P</td>
      <td>ADWDA039P</td>
      <td>ADGFF0XL</td>
      <td>ADWD64LUP</td>
      <td>ADWDXXOQP</td>
      <td>ADWD5U24P</td>
      <td>ADWDKN5RR</td>
      <td>ADWD7MZ1A</td>
      <td>ADWD8FD7P</td>
      <td>ADWDL0XWR</td>
      <td>ADWDIVNLP</td>
      <td>ADWDQ4L5R</td>
      <td>ADWDALCKR</td>
      <td>ADWDGJ88P</td>
      <td>ADWDA432R</td>
      <td>ADWDA433R</td>
      <td>ADWDFW5SF</td>
      <td>ADWDMD93R</td>
      <td>ADIFCLPNR</td>
      <td>ADCRCPRSR</td>
      <td>ADCRECOSR</td>
      <td>ADPWLBSH</td>
      <td>ADCRILLBR</td>
      <td>ADWD7OM6R</td>
      <td>ADAMTOTSR</td>
      <td>ADWDA441P</td>
      <td>ADWD7TPDP</td>
      <td>ADWDUYDZR</td>
      <td>ADWDOS8LR</td>
      <td>ADWDIVNLP</td>
      <td>ADWDLIKMP</td>
      <td>ADWDE0D5R</td>
      <td>ADWDA150R</td>
      <td>ADWDY9K8R</td>
      <td>ADWD1OBUR</td>
      <td>ADWDA075R</td>
      <td>ADWDODTKR</td>
      <td>ADWDJZYDR</td>
      <td>ADWDGHL4P</td>
      <td>ADWD3IS4P</td>
      <td>ADWD75RDP</td>
      <td>ADWDU54MP</td>
      <td>ADGFF54G</td>
      <td>ADGFF54O</td>
      <td>ADGFF54G</td>
      <td>ADGFF54O</td>
      <td>ADWDA160P</td>
      <td>ADWDA011P</td>
      <td>ADWD8FD7P</td>
      <td>ADCRHMTSR</td>
      <td>ADCRILLSR</td>
      <td>ADCRGEXSR</td>
      <td>ADTCPSCO</td>
      <td>ADACAMLSR</td>
      <td>ADTCPSCO</td>
      <td>ADACAMLSR</td>
      <td>ADCRGVESR</td>
      <td>ADWDCSMYR</td>
      <td>ADGCI9SS</td>
      <td>ADCRHMROR</td>
      <td>ADWDMD93R</td>
      <td>ADIFCLPNR</td>
      <td>ADGFREVO</td>
      <td>ADGFTRBO</td>
      <td>ADGFGRVO</td>
      <td>ADGFORVO</td>
      <td>ADINVR..</td>
      <td>ADWDVN9PR</td>
      <td>ADGCY4PR</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>Virgin Islands (United States)</th>
      <td>VGWD8I78R</td>
      <td>VGWDNYMYR</td>
      <td>VGWDYTADR</td>
      <td>VGACPOVBR</td>
      <td>VGWDUGUER</td>
      <td>VGWDA131R</td>
      <td>VGWDUGUER</td>
      <td>VGWDA903R</td>
      <td>VGWDQOG4R</td>
      <td>VGWDMD93R</td>
      <td>VGWDLBORR</td>
      <td>VGWDA378P</td>
      <td>VGWD1JCLR</td>
      <td>VGWD2KTMR</td>
      <td>VGWDYI32R</td>
      <td>VGGFF42G</td>
      <td>VGGFF42O</td>
      <td>VGWDPVWXR</td>
      <td>VGWD5MSAR</td>
      <td>VGWD7X4NP</td>
      <td>VGWDA141R</td>
      <td>VGWD153FR</td>
      <td>VGWD6MMLR</td>
      <td>VGWDHGSZR</td>
      <td>VGWDXB26P</td>
      <td>VGWDANDRP</td>
      <td>VGWDZMVPA</td>
      <td>VGWDZPVLR</td>
      <td>VGWDCU7JR</td>
      <td>VGWD1LNHR</td>
      <td>VGWD67W8P</td>
      <td>VGWDTMWJR</td>
      <td>VGWDTMWJR</td>
      <td>VGWDTMWJR</td>
      <td>VGWDTMWJR</td>
      <td>VGWDTMWJR</td>
      <td>VGWDTMWJR</td>
      <td>VGGCIBXV</td>
      <td>VGWDUGUER</td>
      <td>VGWDSWAWR</td>
      <td>VGWD3PM6R</td>
      <td>VGWDNNO1R</td>
      <td>VGWD24FSR</td>
      <td>VGWDGC8WR</td>
      <td>VGWDBSXAR</td>
      <td>VGWDA903R</td>
      <td>VGWDMD93R</td>
      <td>VGWDBIZXR</td>
      <td>VGWDOOGER</td>
      <td>VGWDLP15P</td>
      <td>VGWDDLO5P</td>
      <td>VGWD8FD7P</td>
      <td>VGWDZPVLR</td>
      <td>VGWDZPVLR</td>
      <td>VGWDL6Z8R</td>
      <td>VGWDA121R</td>
      <td>VGWD9I7VR</td>
      <td>VGWDA124R</td>
      <td>VGWD8S7HR</td>
      <td>VGXTPEN.</td>
      <td>VGWD2OO1A</td>
      <td>VGWDB1HMR</td>
      <td>VGWDWSO8R</td>
      <td>VGWD8FD7P</td>
      <td>VGWD3QBGR</td>
      <td>VGWDDK5XR</td>
      <td>VGWDAJTXR</td>
      <td>VGWDZP5NR</td>
      <td>VGWDFCM9R</td>
      <td>VGWD7YW0R</td>
      <td>VGWDIT9QR</td>
      <td>VGWDA443R</td>
      <td>VGWDA444R</td>
      <td>VGWDA445R</td>
      <td>VGWDVMS9R</td>
      <td>VGWDVM7CR</td>
      <td>VGWD6GL2R</td>
      <td>VGWDEZ1JR</td>
      <td>VGWDVVWFR</td>
      <td>VGWDL1BLR</td>
      <td>VGWDYPL5R</td>
      <td>VGWD3OFRR</td>
      <td>VGWDSUEDR</td>
      <td>VGGCWQYS</td>
      <td>VGWDA151P</td>
      <td>VGWDA039P</td>
      <td>VGGFF0XL</td>
      <td>VGWD64LUP</td>
      <td>VGWDXXOQP</td>
      <td>VGWD5U24P</td>
      <td>VGWDKN5RR</td>
      <td>VGWD7MZ1A</td>
      <td>VGWD8FD7P</td>
      <td>VGWDL0XWR</td>
      <td>VGWDIVNLP</td>
      <td>VGWDQ4L5R</td>
      <td>VGWDALCKR</td>
      <td>VGWDGJ88P</td>
      <td>VGWDA432R</td>
      <td>VGWDA433R</td>
      <td>VGWDFW5SF</td>
      <td>VGWDMD93R</td>
      <td>VGIFCLPNR</td>
      <td>VGCRCPRSR</td>
      <td>VGCRECOSR</td>
      <td>VGPWLBSH</td>
      <td>VGCRILLBR</td>
      <td>VGWD7OM6R</td>
      <td>VGAMTOTSR</td>
      <td>VGWDA441P</td>
      <td>VGWD7TPDP</td>
      <td>VGWDUYDZR</td>
      <td>VGWDOS8LR</td>
      <td>VGWDIVNLP</td>
      <td>VGWDLIKMP</td>
      <td>VGWDE0D5R</td>
      <td>VGWDA150R</td>
      <td>VGWDY9K8R</td>
      <td>VGWD1OBUR</td>
      <td>VGWDA075R</td>
      <td>VGWDODTKR</td>
      <td>VGWDJZYDR</td>
      <td>VGWDGHL4P</td>
      <td>VGWD3IS4P</td>
      <td>VGWD75RDP</td>
      <td>VGWDU54MP</td>
      <td>VGGFF54G</td>
      <td>VGGFF54O</td>
      <td>VGGFF54G</td>
      <td>VGGFF54O</td>
      <td>VGWDA160P</td>
      <td>VGWDA011P</td>
      <td>VGWD8FD7P</td>
      <td>VGCRHMTSR</td>
      <td>VGCRILLSR</td>
      <td>VGCRGEXSR</td>
      <td>VGTCPSCO</td>
      <td>VGACAMLSR</td>
      <td>VGTCPSCO</td>
      <td>VGACAMLSR</td>
      <td>VGCRGVESR</td>
      <td>VGWDCSMYR</td>
      <td>VGGCI9SS</td>
      <td>VGCRHMROR</td>
      <td>VGWDMD93R</td>
      <td>VGIFCLPNR</td>
      <td>VGGFREVO</td>
      <td>VGGFTRBO</td>
      <td>VGGFGRVO</td>
      <td>VGGFORVO</td>
      <td>VGINVR..</td>
      <td>VGWDVN9PR</td>
      <td>VGGCY4PR</td>
    </tr>
    <tr>
      <th>West Bank and Gaza</th>
      <td>WBWD8I78R</td>
      <td>WBWDNYMYR</td>
      <td>WBWDYTADR</td>
      <td>WBACPOVBR</td>
      <td>WBWDUGUER</td>
      <td>WBWDA131R</td>
      <td>WBWDUGUER</td>
      <td>WBWDA903R</td>
      <td>WBWDQOG4R</td>
      <td>WBWDMD93R</td>
      <td>WBWDLBORR</td>
      <td>WBWDA378P</td>
      <td>WBWD1JCLR</td>
      <td>WBWD2KTMR</td>
      <td>WBWDYI32R</td>
      <td>WBGFF42G</td>
      <td>WBGFF42O</td>
      <td>WBWDPVWXR</td>
      <td>WBWD5MSAR</td>
      <td>WBWD7X4NP</td>
      <td>WBWDA141R</td>
      <td>WBWD153FR</td>
      <td>WBWD6MMLR</td>
      <td>WBWDHGSZR</td>
      <td>WBWDXB26P</td>
      <td>WBWDANDRP</td>
      <td>WBWDZMVPA</td>
      <td>WBWDZPVLR</td>
      <td>WBWDCU7JR</td>
      <td>WBWD1LNHR</td>
      <td>WBWD67W8P</td>
      <td>WBWDTMWJR</td>
      <td>WBWDTMWJR</td>
      <td>WBWDTMWJR</td>
      <td>WBWDTMWJR</td>
      <td>WBWDTMWJR</td>
      <td>WBWDTMWJR</td>
      <td>WBGCIBXV</td>
      <td>WBWDUGUER</td>
      <td>WBWDSWAWR</td>
      <td>WBWD3PM6R</td>
      <td>WBWDNNO1R</td>
      <td>WBWD24FSR</td>
      <td>WBWDGC8WR</td>
      <td>WBWDBSXAR</td>
      <td>WBWDA903R</td>
      <td>WBWDMD93R</td>
      <td>WBWDBIZXR</td>
      <td>WBWDOOGER</td>
      <td>WBWDLP15P</td>
      <td>WBWDDLO5P</td>
      <td>WBWD8FD7P</td>
      <td>WBWDZPVLR</td>
      <td>WBWDZPVLR</td>
      <td>WBWDL6Z8R</td>
      <td>WBWDA121R</td>
      <td>WBWD9I7VR</td>
      <td>WBWDA124R</td>
      <td>WBWD8S7HR</td>
      <td>WBXTPEN.</td>
      <td>WBWD2OO1A</td>
      <td>WBWDB1HMR</td>
      <td>WBWDWSO8R</td>
      <td>WBWD8FD7P</td>
      <td>WBWD3QBGR</td>
      <td>WBWDDK5XR</td>
      <td>WBWDAJTXR</td>
      <td>WBWDZP5NR</td>
      <td>WBWDFCM9R</td>
      <td>WBWD7YW0R</td>
      <td>WBWDIT9QR</td>
      <td>WBWDA443R</td>
      <td>WBWDA444R</td>
      <td>WBWDA445R</td>
      <td>WBWDVMS9R</td>
      <td>WBWDVM7CR</td>
      <td>WBWD6GL2R</td>
      <td>WBWDEZ1JR</td>
      <td>WBWDVVWFR</td>
      <td>WBWDL1BLR</td>
      <td>WBWDYPL5R</td>
      <td>WBWD3OFRR</td>
      <td>WBWDSUEDR</td>
      <td>WBGCWQYS</td>
      <td>WBWDA151P</td>
      <td>WBWDA039P</td>
      <td>WBGFF0XL</td>
      <td>WBWD64LUP</td>
      <td>WBWDXXOQP</td>
      <td>WBWD5U24P</td>
      <td>WBWDKN5RR</td>
      <td>WBWD7MZ1A</td>
      <td>WBWD8FD7P</td>
      <td>WBWDL0XWR</td>
      <td>WBWDIVNLP</td>
      <td>WBWDQ4L5R</td>
      <td>WBWDALCKR</td>
      <td>WBWDGJ88P</td>
      <td>WBWDA432R</td>
      <td>WBWDA433R</td>
      <td>WBWDFW5SF</td>
      <td>WBWDMD93R</td>
      <td>WBIFCLPNR</td>
      <td>WBCRCPRSR</td>
      <td>WBCRECOSR</td>
      <td>WBPWLBSH</td>
      <td>WBCRILLBR</td>
      <td>WBWD7OM6R</td>
      <td>WBAMTOTSR</td>
      <td>WBWDA441P</td>
      <td>WBWD7TPDP</td>
      <td>WBWDUYDZR</td>
      <td>WBWDOS8LR</td>
      <td>WBWDIVNLP</td>
      <td>WBWDLIKMP</td>
      <td>WBWDE0D5R</td>
      <td>WBWDA150R</td>
      <td>WBWDY9K8R</td>
      <td>WBWD1OBUR</td>
      <td>WBWDA075R</td>
      <td>WBWDODTKR</td>
      <td>WBWDJZYDR</td>
      <td>WBWDGHL4P</td>
      <td>WBWD3IS4P</td>
      <td>WBWD75RDP</td>
      <td>WBWDU54MP</td>
      <td>WBGFF54G</td>
      <td>WBGFF54O</td>
      <td>WBGFF54G</td>
      <td>WBGFF54O</td>
      <td>WBWDA160P</td>
      <td>WBWDA011P</td>
      <td>WBWD8FD7P</td>
      <td>WBCRHMTSR</td>
      <td>WBCRILLSR</td>
      <td>WBCRGEXSR</td>
      <td>WBTCPSCO</td>
      <td>WBACAMLSR</td>
      <td>WBTCPSCO</td>
      <td>WBACAMLSR</td>
      <td>WBCRGVESR</td>
      <td>WBWDCSMYR</td>
      <td>WBGCI9SS</td>
      <td>WBCRHMROR</td>
      <td>WBWDMD93R</td>
      <td>WBIFCLPNR</td>
      <td>WBGFREVO</td>
      <td>WBGFTRBO</td>
      <td>WBGFGRVO</td>
      <td>WBGFORVO</td>
      <td>WBINVR..</td>
      <td>WBWDVN9PR</td>
      <td>WBGCY4PR</td>
    </tr>
    <tr>
      <th>Yemen</th>
      <td>YAWD8I78R</td>
      <td>YAWDNYMYR</td>
      <td>YAWDYTADR</td>
      <td>YAACPOVBR</td>
      <td>YAWDUGUER</td>
      <td>YAWDA131R</td>
      <td>YAWDUGUER</td>
      <td>YAWDA903R</td>
      <td>YAWDQOG4R</td>
      <td>YAWDMD93R</td>
      <td>YAWDLBORR</td>
      <td>YAWDA378P</td>
      <td>YAWD1JCLR</td>
      <td>YAWD2KTMR</td>
      <td>YAWDYI32R</td>
      <td>YAGFF42G</td>
      <td>YAGFF42O</td>
      <td>YAWDPVWXR</td>
      <td>YAWD5MSAR</td>
      <td>YAWD7X4NP</td>
      <td>YAWDA141R</td>
      <td>YAWD153FR</td>
      <td>YAWD6MMLR</td>
      <td>YAWDHGSZR</td>
      <td>YAWDXB26P</td>
      <td>YAWDANDRP</td>
      <td>YAWDZMVPA</td>
      <td>YAWDZPVLR</td>
      <td>YAWDCU7JR</td>
      <td>YAWD1LNHR</td>
      <td>YAWD67W8P</td>
      <td>YAWDTMWJR</td>
      <td>YAWDTMWJR</td>
      <td>YAWDTMWJR</td>
      <td>YAWDTMWJR</td>
      <td>YAWDTMWJR</td>
      <td>YAWDTMWJR</td>
      <td>YAGCIBXV</td>
      <td>YAWDUGUER</td>
      <td>YAWDSWAWR</td>
      <td>YAWD3PM6R</td>
      <td>YAWDNNO1R</td>
      <td>YAWD24FSR</td>
      <td>YAWDGC8WR</td>
      <td>YAWDBSXAR</td>
      <td>YAWDA903R</td>
      <td>YAWDMD93R</td>
      <td>YAWDBIZXR</td>
      <td>YAWDOOGER</td>
      <td>YAWDLP15P</td>
      <td>YAWDDLO5P</td>
      <td>YAWD8FD7P</td>
      <td>YAWDZPVLR</td>
      <td>YAWDZPVLR</td>
      <td>YAWDL6Z8R</td>
      <td>YAWDA121R</td>
      <td>YAWD9I7VR</td>
      <td>YAWDA124R</td>
      <td>YAWD8S7HR</td>
      <td>YAXTPEN.</td>
      <td>YAWD2OO1A</td>
      <td>YAWDB1HMR</td>
      <td>YAWDWSO8R</td>
      <td>YAWD8FD7P</td>
      <td>YAWD3QBGR</td>
      <td>YAWDDK5XR</td>
      <td>YAWDAJTXR</td>
      <td>YAWDZP5NR</td>
      <td>YAWDFCM9R</td>
      <td>YAWD7YW0R</td>
      <td>YAWDIT9QR</td>
      <td>YAWDA443R</td>
      <td>YAWDA444R</td>
      <td>YAWDA445R</td>
      <td>YAWDVMS9R</td>
      <td>YAWDVM7CR</td>
      <td>YAWD6GL2R</td>
      <td>YAWDEZ1JR</td>
      <td>YAWDVVWFR</td>
      <td>YAWDL1BLR</td>
      <td>YAWDYPL5R</td>
      <td>YAWD3OFRR</td>
      <td>YAWDSUEDR</td>
      <td>YAGCWQYS</td>
      <td>YAWDA151P</td>
      <td>YAWDA039P</td>
      <td>YAGFF0XL</td>
      <td>YAWD64LUP</td>
      <td>YAWDXXOQP</td>
      <td>YAWD5U24P</td>
      <td>YAWDKN5RR</td>
      <td>YAWD7MZ1A</td>
      <td>YAWD8FD7P</td>
      <td>YAWDL0XWR</td>
      <td>YAWDIVNLP</td>
      <td>YAWDQ4L5R</td>
      <td>YAWDALCKR</td>
      <td>YAWDGJ88P</td>
      <td>YAWDA432R</td>
      <td>YAWDA433R</td>
      <td>YAWDFW5SF</td>
      <td>YAWDMD93R</td>
      <td>YAIFCLPNR</td>
      <td>YACRCPRSR</td>
      <td>YACRECOSR</td>
      <td>YAPWLBSH</td>
      <td>YACRILLBR</td>
      <td>YAWD7OM6R</td>
      <td>YAAMTOTSR</td>
      <td>YAWDA441P</td>
      <td>YAWD7TPDP</td>
      <td>YAWDUYDZR</td>
      <td>YAWDOS8LR</td>
      <td>YAWDIVNLP</td>
      <td>YAWDLIKMP</td>
      <td>YAWDE0D5R</td>
      <td>YAWDA150R</td>
      <td>YAWDY9K8R</td>
      <td>YAWD1OBUR</td>
      <td>YAWDA075R</td>
      <td>YAWDODTKR</td>
      <td>YAWDJZYDR</td>
      <td>YAWDGHL4P</td>
      <td>YAWD3IS4P</td>
      <td>YAWD75RDP</td>
      <td>YAWDU54MP</td>
      <td>YAGFF54G</td>
      <td>YAGFF54O</td>
      <td>YAGFF54G</td>
      <td>YAGFF54O</td>
      <td>YAWDA160P</td>
      <td>YAWDA011P</td>
      <td>YAWD8FD7P</td>
      <td>YACRHMTSR</td>
      <td>YACRILLSR</td>
      <td>YACRGEXSR</td>
      <td>YATCPSCO</td>
      <td>YAACAMLSR</td>
      <td>YATCPSCO</td>
      <td>YAACAMLSR</td>
      <td>YACRGVESR</td>
      <td>YAWDCSMYR</td>
      <td>YAGCI9SS</td>
      <td>YACRHMROR</td>
      <td>YAWDMD93R</td>
      <td>YAIFCLPNR</td>
      <td>YAGFREVO</td>
      <td>YAGFTRBO</td>
      <td>YAGFGRVO</td>
      <td>YAGFORVO</td>
      <td>YAINVR..</td>
      <td>YAWDVN9PR</td>
      <td>YAGCY4PR</td>
    </tr>
    <tr>
      <th>Zambia</th>
      <td>ZMWD8I78R</td>
      <td>ZMWDNYMYR</td>
      <td>ZMWDYTADR</td>
      <td>ZMACPOVBR</td>
      <td>ZMWDUGUER</td>
      <td>ZMWDA131R</td>
      <td>ZMWDUGUER</td>
      <td>ZMWDA903R</td>
      <td>ZMWDQOG4R</td>
      <td>ZMWDMD93R</td>
      <td>ZMWDLBORR</td>
      <td>ZMWDA378P</td>
      <td>ZMWD1JCLR</td>
      <td>ZMWD2KTMR</td>
      <td>ZMWDYI32R</td>
      <td>ZMGFF42G</td>
      <td>ZMGFF42O</td>
      <td>ZMWDPVWXR</td>
      <td>ZMWD5MSAR</td>
      <td>ZMWD7X4NP</td>
      <td>ZMWDA141R</td>
      <td>ZMWD153FR</td>
      <td>ZMWD6MMLR</td>
      <td>ZMWDHGSZR</td>
      <td>ZMWDXB26P</td>
      <td>ZMWDANDRP</td>
      <td>ZMWDZMVPA</td>
      <td>ZMWDZPVLR</td>
      <td>ZMWDCU7JR</td>
      <td>ZMWD1LNHR</td>
      <td>ZMWD67W8P</td>
      <td>ZMWDTMWJR</td>
      <td>ZMWDTMWJR</td>
      <td>ZMWDTMWJR</td>
      <td>ZMWDTMWJR</td>
      <td>ZMWDTMWJR</td>
      <td>ZMWDTMWJR</td>
      <td>ZMGCIBXV</td>
      <td>ZMWDUGUER</td>
      <td>ZMWDSWAWR</td>
      <td>ZMWD3PM6R</td>
      <td>ZMWDNNO1R</td>
      <td>ZMWD24FSR</td>
      <td>ZMWDGC8WR</td>
      <td>ZMWDBSXAR</td>
      <td>ZMWDA903R</td>
      <td>ZMWDMD93R</td>
      <td>ZMWDBIZXR</td>
      <td>ZMWDOOGER</td>
      <td>ZMWDLP15P</td>
      <td>ZMWDDLO5P</td>
      <td>ZMWD8FD7P</td>
      <td>ZMWDZPVLR</td>
      <td>ZMWDZPVLR</td>
      <td>ZMWDL6Z8R</td>
      <td>ZMWDA121R</td>
      <td>ZMWD9I7VR</td>
      <td>ZMWDA124R</td>
      <td>ZMWD8S7HR</td>
      <td>ZMXTPEN.</td>
      <td>ZMWD2OO1A</td>
      <td>ZMWDB1HMR</td>
      <td>ZMWDWSO8R</td>
      <td>ZMWD8FD7P</td>
      <td>ZMWD3QBGR</td>
      <td>ZMWDDK5XR</td>
      <td>ZMWDAJTXR</td>
      <td>ZMWDZP5NR</td>
      <td>ZMWDFCM9R</td>
      <td>ZMWD7YW0R</td>
      <td>ZMWDIT9QR</td>
      <td>ZMWDA443R</td>
      <td>ZMWDA444R</td>
      <td>ZMWDA445R</td>
      <td>ZMWDVMS9R</td>
      <td>ZMWDVM7CR</td>
      <td>ZMWD6GL2R</td>
      <td>ZMWDEZ1JR</td>
      <td>ZMWDVVWFR</td>
      <td>ZMWDL1BLR</td>
      <td>ZMWDYPL5R</td>
      <td>ZMWD3OFRR</td>
      <td>ZMWDSUEDR</td>
      <td>ZMGCWQYS</td>
      <td>ZMWDA151P</td>
      <td>ZMWDA039P</td>
      <td>ZMGFF0XL</td>
      <td>ZMWD64LUP</td>
      <td>ZMWDXXOQP</td>
      <td>ZMWD5U24P</td>
      <td>ZMWDKN5RR</td>
      <td>ZMWD7MZ1A</td>
      <td>ZMWD8FD7P</td>
      <td>ZMWDL0XWR</td>
      <td>ZMWDIVNLP</td>
      <td>ZMWDQ4L5R</td>
      <td>ZMWDALCKR</td>
      <td>ZMWDGJ88P</td>
      <td>ZMWDA432R</td>
      <td>ZMWDA433R</td>
      <td>ZMWDFW5SF</td>
      <td>ZMWDMD93R</td>
      <td>ZMIFCLPNR</td>
      <td>ZMCRCPRSR</td>
      <td>ZMCRECOSR</td>
      <td>ZMPWLBSH</td>
      <td>ZMCRILLBR</td>
      <td>ZMWD7OM6R</td>
      <td>ZMAMTOTSR</td>
      <td>ZMWDA441P</td>
      <td>ZMWD7TPDP</td>
      <td>ZMWDUYDZR</td>
      <td>ZMWDOS8LR</td>
      <td>ZMWDIVNLP</td>
      <td>ZMWDLIKMP</td>
      <td>ZMWDE0D5R</td>
      <td>ZMWDA150R</td>
      <td>ZMWDY9K8R</td>
      <td>ZMWD1OBUR</td>
      <td>ZMWDA075R</td>
      <td>ZMWDODTKR</td>
      <td>ZMWDJZYDR</td>
      <td>ZMWDGHL4P</td>
      <td>ZMWD3IS4P</td>
      <td>ZMWD75RDP</td>
      <td>ZMWDU54MP</td>
      <td>ZMGFF54G</td>
      <td>ZMGFF54O</td>
      <td>ZMGFF54G</td>
      <td>ZMGFF54O</td>
      <td>ZMWDA160P</td>
      <td>ZMWDA011P</td>
      <td>ZMWD8FD7P</td>
      <td>ZMCRHMTSR</td>
      <td>ZMCRILLSR</td>
      <td>ZMCRGEXSR</td>
      <td>ZMTCPSCO</td>
      <td>ZMACAMLSR</td>
      <td>ZMTCPSCO</td>
      <td>ZMACAMLSR</td>
      <td>ZMCRGVESR</td>
      <td>ZMWDCSMYR</td>
      <td>ZMGCI9SS</td>
      <td>ZMCRHMROR</td>
      <td>ZMWDMD93R</td>
      <td>ZMIFCLPNR</td>
      <td>ZMGFREVO</td>
      <td>ZMGFTRBO</td>
      <td>ZMGFGRVO</td>
      <td>ZMGFORVO</td>
      <td>ZMINVR..</td>
      <td>ZMWDVN9PR</td>
      <td>ZMGCY4PR</td>
    </tr>
    <tr>
      <th>Zimbabwe</th>
      <td>ZIWD8I78R</td>
      <td>ZIWDNYMYR</td>
      <td>ZIWDYTADR</td>
      <td>ZIACPOVBR</td>
      <td>ZIWDUGUER</td>
      <td>ZIWDA131R</td>
      <td>ZIWDUGUER</td>
      <td>ZIWDA903R</td>
      <td>ZIWDQOG4R</td>
      <td>ZIWDMD93R</td>
      <td>ZIWDLBORR</td>
      <td>ZIWDA378P</td>
      <td>ZIWD1JCLR</td>
      <td>ZIWD2KTMR</td>
      <td>ZIWDYI32R</td>
      <td>ZIGFF42G</td>
      <td>ZIGFF42O</td>
      <td>ZIWDPVWXR</td>
      <td>ZIWD5MSAR</td>
      <td>ZIWD7X4NP</td>
      <td>ZIWDA141R</td>
      <td>ZIWD153FR</td>
      <td>ZIWD6MMLR</td>
      <td>ZIWDHGSZR</td>
      <td>ZIWDXB26P</td>
      <td>ZIWDANDRP</td>
      <td>ZIWDZMVPA</td>
      <td>ZIWDZPVLR</td>
      <td>ZIWDCU7JR</td>
      <td>ZIWD1LNHR</td>
      <td>ZIWD67W8P</td>
      <td>ZIWDTMWJR</td>
      <td>ZIWDTMWJR</td>
      <td>ZIWDTMWJR</td>
      <td>ZIWDTMWJR</td>
      <td>ZIWDTMWJR</td>
      <td>ZIWDTMWJR</td>
      <td>ZIGCIBXV</td>
      <td>ZIWDUGUER</td>
      <td>ZIWDSWAWR</td>
      <td>ZIWD3PM6R</td>
      <td>ZIWDNNO1R</td>
      <td>ZIWD24FSR</td>
      <td>ZIWDGC8WR</td>
      <td>ZIWDBSXAR</td>
      <td>ZIWDA903R</td>
      <td>ZIWDMD93R</td>
      <td>ZIWDBIZXR</td>
      <td>ZIWDOOGER</td>
      <td>ZIWDLP15P</td>
      <td>ZIWDDLO5P</td>
      <td>ZIWD8FD7P</td>
      <td>ZIWDZPVLR</td>
      <td>ZIWDZPVLR</td>
      <td>ZIWDL6Z8R</td>
      <td>ZIWDA121R</td>
      <td>ZIWD9I7VR</td>
      <td>ZIWDA124R</td>
      <td>ZIWD8S7HR</td>
      <td>ZIXTPEN.</td>
      <td>ZIWD2OO1A</td>
      <td>ZIWDB1HMR</td>
      <td>ZIWDWSO8R</td>
      <td>ZIWD8FD7P</td>
      <td>ZIWD3QBGR</td>
      <td>ZIWDDK5XR</td>
      <td>ZIWDAJTXR</td>
      <td>ZIWDZP5NR</td>
      <td>ZIWDFCM9R</td>
      <td>ZIWD7YW0R</td>
      <td>ZIWDIT9QR</td>
      <td>ZIWDA443R</td>
      <td>ZIWDA444R</td>
      <td>ZIWDA445R</td>
      <td>ZIWDVMS9R</td>
      <td>ZIWDVM7CR</td>
      <td>ZIWD6GL2R</td>
      <td>ZIWDEZ1JR</td>
      <td>ZIWDVVWFR</td>
      <td>ZIWDL1BLR</td>
      <td>ZIWDYPL5R</td>
      <td>ZIWD3OFRR</td>
      <td>ZIWDSUEDR</td>
      <td>ZIGCWQYS</td>
      <td>ZIWDA151P</td>
      <td>ZIWDA039P</td>
      <td>ZIGFF0XL</td>
      <td>ZIWD64LUP</td>
      <td>ZIWDXXOQP</td>
      <td>ZIWD5U24P</td>
      <td>ZIWDKN5RR</td>
      <td>ZIWD7MZ1A</td>
      <td>ZIWD8FD7P</td>
      <td>ZIWDL0XWR</td>
      <td>ZIWDIVNLP</td>
      <td>ZIWDQ4L5R</td>
      <td>ZIWDALCKR</td>
      <td>ZIWDGJ88P</td>
      <td>ZIWDA432R</td>
      <td>ZIWDA433R</td>
      <td>ZIWDFW5SF</td>
      <td>ZIWDMD93R</td>
      <td>ZIIFCLPNR</td>
      <td>ZICRCPRSR</td>
      <td>ZICRECOSR</td>
      <td>ZIPWLBSH</td>
      <td>ZICRILLBR</td>
      <td>ZIWD7OM6R</td>
      <td>ZIAMTOTSR</td>
      <td>ZIWDA441P</td>
      <td>ZIWD7TPDP</td>
      <td>ZIWDUYDZR</td>
      <td>ZIWDOS8LR</td>
      <td>ZIWDIVNLP</td>
      <td>ZIWDLIKMP</td>
      <td>ZIWDE0D5R</td>
      <td>ZIWDA150R</td>
      <td>ZIWDY9K8R</td>
      <td>ZIWD1OBUR</td>
      <td>ZIWDA075R</td>
      <td>ZIWDODTKR</td>
      <td>ZIWDJZYDR</td>
      <td>ZIWDGHL4P</td>
      <td>ZIWD3IS4P</td>
      <td>ZIWD75RDP</td>
      <td>ZIWDU54MP</td>
      <td>ZIGFF54G</td>
      <td>ZIGFF54O</td>
      <td>ZIGFF54G</td>
      <td>ZIGFF54O</td>
      <td>ZIWDA160P</td>
      <td>ZIWDA011P</td>
      <td>ZIWD8FD7P</td>
      <td>ZICRHMTSR</td>
      <td>ZICRILLSR</td>
      <td>ZICRGEXSR</td>
      <td>ZITCPSCO</td>
      <td>ZIACAMLSR</td>
      <td>ZITCPSCO</td>
      <td>ZIACAMLSR</td>
      <td>ZICRGVESR</td>
      <td>ZIWDCSMYR</td>
      <td>ZIGCI9SS</td>
      <td>ZICRHMROR</td>
      <td>ZIWDMD93R</td>
      <td>ZIIFCLPNR</td>
      <td>ZIGFREVO</td>
      <td>ZIGFTRBO</td>
      <td>ZIGFGRVO</td>
      <td>ZIGFORVO</td>
      <td>ZIINVR..</td>
      <td>ZIWDVN9PR</td>
      <td>ZIGCY4PR</td>
    </tr>
  </tbody>
</table>
<p>211 rows × 153 columns</p>
</div>



Certain metrics have to be manipulated for them to be comparable to all others. We thus add columns for these additional metrics in the list of columns 'Country_Values_Table_Columns'


```python
Country_Values_Table_Columns = ['1.1.1', '1.1.1.1', '1.2.1', '1.2.2', '1.3.1', '1.5.3', '1.A.1', '1.A.2', '1.A.2.1', '1.B.1',
                                '2.1.1', '2.1.2', '2.2.1', '2.2.2', '2.2.2.1', '2.A.1', '2.A.2',
                                '3.1.1', '3.1.2', '3.2.1', '3.2.2', '3.3.1', '3.3.1.1', '3.3.1.2', '3.3.2', '3.7.2', '3.8.2', '3.9.2', '3.A.1', '3.A.1.1', '3.C.1',
                                '4.1.1', '4.1.1.1', '4.1.1.2', '4.1.1.3', '4.1.1.4', '4.1.1.5', '4.4.1', '4.5.1', '4.6.1', '4.6.1.1', '4.6.1.2', '4.6.1.3', '4.6.1.4', '4.6.1.5', '4.A.1',
                                '5.1.1', '5.5.1', '5.6.1', '5.6.1.1', '5.B.1', '5.B.1.1', '5.B.1.2',
                                '6.1.1', '6.2.1', '6.4.2',
                                '7.1.1', '7.1.2', '7.2.1', '7.2.1.1', '7.3.1', '7.3.1.1', '7.3.1.2',
                                '8.1.1', '8.2.1', '8.2.1.1', '8.2.1.2', '8.2.1.3', '8.5.2', '8.5.2.1', '8.5.2.2', '8.5.2.3', '8.5.2.4', '8.5.2.5', '8.6.1', '8.6.1.1', '8.6.1.2', '8.7.1', '8.7.1.1', '8.7.1.2', '8.7.1.3', '8.7.1.4', '8.7.1.5', '8.7.1.6', '8.7.1.7', '8.7.1.8', '8.8.2', '8.10.1', '8.10.1.1', '8.B.1',
                                '9.1.2.', '9.1.2..1', '9.1.2..2', '9.2.1.', '9.2.1..1', '9.2.1..2', '9.2.1..3', '9.2.2.', '9.4.1', '9.5.1', '9.5.2', '9.C.1',
                                '10.1.1', '10.1.1.1', '10.2.1', '10.3.1', '10.3.1.1', '10.3.1.2', '10.3.1.3', '10.4.1', '10.5.1', '10.5.1.1', '10.5.1.2',
                                '11.6.2', '11.6.2.1',
                                '12.7.1', '12.C.1',
                                '13.2.1', '13.2.1.1', # There indeed is no 14th category
                                '15.1.1', '15.1.2', '15.1.2.1', '15.1.2.2', '15.2.1', '15.3.1', '15.5.1', '15.9.1', '15.9.1.1', '15.9.1.2', '15.9.1.3', '15.A.1', '15.A.1.1', '15.B.1', '15.B.1.1',
                                '16.1.1', '16.1.2', '16.1.2.1', '16.1.2.2', '16.2.2', '16.4.1', '16.4.2', '16.5.1', '16.5.1.1', '16.5.2', '16.5.2.1', '16.6.2', '16.9.1', '16.10.2', '16.A.1', '16.B.1', '16.B.1.1',
                                '17.1.1', '17.1.1.1', '17.1.1.2', '17.1.1.3', '17.3.1', '17.4.1', '17.6.2', '17.8.1']
```

Bellow we can see the discrepancies in columns


```python
# yields the elements in `Country_Values_Table_Columns` that are NOT in `df.columns`.
columns_in_Country_Values_Table_not_in_df = list(np.setdiff1d(Country_Values_Table_Columns,
                                                              df.columns))
```

$$ \\ $$
#### Now we can collect our data from DSWS


```python
# First: create a pandas data-frame to be populated with our data.
Country_Values_Table = pd.DataFrame(index = df.index, columns = Country_Values_Table_Columns)
```


```python
# Now: Populated the Country_Values_Table data-frame
for j in range(len(df.columns)):
    Country_Values_Table_Column_j = [] # List to be populated in this loop
    for i in range(len(df.index)):
        # Call DSWS for each ticker/code in df, by column and row (in that order).
        data_point = ds.get_data(tickers = df.iloc[i,j], fields = "X", start = '1950-01-01', freq = 'Y').dropna()
        
        try:
            # The ' [-1] ' here means that we are taking the last value in the series called of DSWS.
            Country_Values_Table_Column_j.append(float(data_point.iloc[-1]))
        except:
            # Due to idiosyncrasies in ESG reporting, not all countries report on all categories.
            # This ' except ' logic allows us to account for that.
            Country_Values_Table_Column_j.append(np.nan)
    Country_Values_Table[df.columns[j]] = Country_Values_Table_Column_j
```


```python
Country_Values_Table
```

$$ \\ $$
## Pickle

It is quite time consuming to request DSWS for all these codes. Let's save our progress this far using [Pickle](https://pythonprogramming.net/python-pickle-module-save-objects-serialization/):


```python
pickle_out = open("ESG-DS.pickle","wb")
pickl = (df, Country_Values_Table_Columns,
         columns_in_Country_Values_Table_not_in_df,
         Country_Values_Table)
pickle.dump(pickl, pickle_out)
pickle_out.close()
```

The cell bellow can be run to load these variables back into the kernel


```python
# pickle_in = open("ESG-DS.pickle","rb")
# df, Country_Values_Table_Columns, columns_in_Country_Values_Table_not_in_df, Country_Values_Table = pickle.load(pickle_in)
# pickle_in.close() # We ought to close the file we opened to allow any other programs access if they need it.
```

$$ \\ $$
## Sorting Out Our Data

The use of ' Country_Values_Table2 ' is only there to deliminate between before and after our 'pickling':


```python
Country_Values_Table2 = Country_Values_Table.copy()
```

Certain metrics have to be manipulated for them to be comparable to all others:


```python
# Going through the mathematical manipulation:
Country_Values_Table2["5.B.1.2"] = Country_Values_Table2["5.B.1"] / Country_Values_Table2["5.B.1.1"]
# Removing unnecessary columns:
Country_Values_Table2 = Country_Values_Table2.drop(columns = ["5.B.1", "5.B.1.1"])
```


```python
Country_Values_Table2["7.3.1.2"] = Country_Values_Table2["7.3.1"] / Country_Values_Table2["7.3.1.1"]
Country_Values_Table2 = Country_Values_Table2.drop(columns = ["7.3.1", "7.3.1.1"])
```


```python
Country_Values_Table2["8.2.1.3"] = Country_Values_Table2["8.2.1.2"] / (Country_Values_Table2["8.2.1"] * Country_Values_Table2["8.2.1.1"])
Country_Values_Table2 = Country_Values_Table2.drop(columns = ["8.2.1.2", "8.2.1" , "8.2.1.1"])
```


```python
Country_Values_Table2["9.2.1..3"] = Country_Values_Table2["9.2.1..1"] / Country_Values_Table2["9.2.1..2"]
Country_Values_Table2 = Country_Values_Table2.drop(columns = ["9.2.1..1", "9.2.1..2"])
```


```python
Country_Values_Table2["16.1.2.2"] = Country_Values_Table2["16.1.2"] / Country_Values_Table2["16.1.2.1"]
Country_Values_Table2 = Country_Values_Table2.drop(columns = ["16.1.2", "16.1.2.1"])
```

$$ \\ $$
## Country Points Table

We can now apply the points system defined above to our data in ' Country_Values_Table2 '


```python
Country_Points_Table1 = pd.DataFrame(index = df.index)

# NOTE THAT THIS IS NOT THE SAME WRANKING AS IN THE EXCEL SHEET, BUT IT IS A MORE MATHEMATICALLY COMMON AND LESS AMBIGUOUS ONE
for j in range(len(Country_Values_Table2.columns)):
    Country_Points_Table1[Country_Values_Table2.columns[j]] = list(Country_Values_Table2.iloc[:,j].rank(method = "dense",
                                                                                                        na_option = "keep",
                                                                                                        pct = True))

Country_Points_Table1.head()
```


```python
Country_Points_Table2 = pd.DataFrame(index = Country_Values_Table2.index)

# NOTE THAT THIS IS NOT THE SAME WRANKING AS IN THE EXCEL SHEET, BUT IT IS A MORE MATHEMATICALLY COMMON AND LESS AMBIGUOUS ONE
for j in range(len(Country_Values_Table2.columns)):
    Country_Points_Table2_column_j = [] # Create a list to be populated
    for i in range(len(Country_Values_Table2.index)):
        
        if math.isnan(Country_Values_Table2.iloc[i,j]):
            # Accounting fo rhte possibility that we did not collect a value
            val = np.nan
            
        else:
            
            # The following 3 lines are used to recreate Excel's ' PERCENTRANK(...) ' function which is different to
            # median percentage, percentile, ' scipy.stats.percentileofscore ' and ' pd.DataFrame.rank(pct = True)) '.
            # Note also that valus might differ slightly due to (backend) rounding errors on Excel
            array_of_lower_vals = Country_Values_Table2.iloc[:,j][Country_Values_Table2.iloc[:,j] < Country_Values_Table2.iloc[i,j]]
            column_len_no_na = len(Country_Values_Table2.iloc[:,j].dropna()) - 1
            val = len(array_of_lower_vals) / column_len_no_na
        
        Country_Points_Table2_column_j.append(val)
    
    Country_Points_Table2[Country_Values_Table2.columns[j]] = Country_Points_Table2_column_j

Country_Points_Table2.head()
```

$$ \\ $$
## Polarity

Certain data-points are better when lower (e.g.: poverty levels), others better when heigher (e.g.: availability of affordable and clean energy). We thus use a Polarity array to denote when which rule is applied.


```python
# You will need the ' ESG-DS.csv ' file
Polarity = pd.read_csv("ESG-DS.csv", header = 1).iloc[:,1:3].dropna()
```


```python
Polarity
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>polar</th>
      <th>Country Points Table full</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Negative</td>
      <td>1.1.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Negative</td>
      <td>1.1.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Negative</td>
      <td>1.2.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Negative</td>
      <td>1.2.2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Positive</td>
      <td>1.3.1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>143</th>
      <td>Positive</td>
      <td>17.1.1</td>
    </tr>
    <tr>
      <th>144</th>
      <td>Positive</td>
      <td>17.3.1</td>
    </tr>
    <tr>
      <th>145</th>
      <td>Positive</td>
      <td>17.4.1</td>
    </tr>
    <tr>
      <th>146</th>
      <td>Negative</td>
      <td>17.6.2</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Positive</td>
      <td>17.8.1</td>
    </tr>
  </tbody>
</table>
<p>148 rows × 2 columns</p>
</div>




```python
# Lists to be populated.
Negatives, Positives = [], []

# Copy of the ' Polarity ' table to delaminate 'before' and 'after' this point in the code
Polarity_temp = Polarity.copy()

for i in range(len(Country_Points_Table2.columns)):
    for j in range(len(Polarity_temp["Country Points Table full"])):
        if Country_Points_Table2.columns[i].split(".")[0:3] == Polarity_temp["Country Points Table full"][j].split(".")[0:3]:
            # For each ESG category in ' Country_Points_Table2 ', find this category in ' Polarity_temp '
            # and save the placement of the specified polarity of that category.
            if Polarity_temp.polar[j] == "Negative":
                Negatives.append(j)
            if Polarity_temp.polar[j] == "Positive":
                Positives.append(j)
            # Once that placement is saved, replace ' Polarity_temp ' data so that it isn't mistakenly used again
            Polarity_temp["Country Points Table full"][j] = "u.s.e.d"
```

$$ \\ $$
## Polarised Table

We may now apply polarity rules to our data-points


```python
# Create a data-frame to be populated
Country_Polarised_Points_Table1 = pd.DataFrame(index = Country_Points_Table2.index,
                                               columns = Country_Points_Table2.columns)
```


```python
for k in [[Negatives, "Negative"], [Positives, "Positive"]]:
    for j in k[0]:
        Country_Polarised_Points_Table1_column_j = []
        for i in range(len(Country_Points_Table2.index)):
            Country_Polarised_Points_Table1_column_j.append(Point(Country_Points_Table2.iloc[i,j], polarity = k[1]))
        Country_Polarised_Points_Table1[Country_Points_Table2.columns[j]] = Country_Polarised_Points_Table1_column_j
```

Note that not all the individual country series codes are retrievable (e.g.: 'AAGFORVO')

## SDG Aggregate Ranks


```python
# Create a data-frame to be populated
SDG_Aggregate_Ranks = pd.DataFrame({("No Poverty", "Scores Available") : list(np.full(len(Country_Polarised_Points_Table1.index), np.nan))},
                                   index = Country_Polarised_Points_Table1.index)

for j,k,m in zip(range(1,18),
                 ["No Poverty", "Zero Hunger", "Good Healthcare and Wellbeing", "Quality of Education",
                  "Gender Equality", "Clean Water and Sanitation", "Affordable and Clean Energy", "Decent Work and Economic Growth",
                  "Industry, Innovation and Infrastructure", "Reduced Inequalities", "Sustainable Cities and Communities", "Responsible Consumption",
                  "Climate Action", "Life Bellow Water", "Life on Land", "Peace, Justice and Strong Institutions", "Partnerships for the Goals"],
                 [3,3,3,5,
                  2,1,1,7,
                  3,4,1,0,
                  0,0,5,5,3]):
    
    # Create lists to be populated:
    col, SDG_Aggregate_Ranks_col1_j, SDG_Aggregate_Ranks_col2_j = [], [], []
    for i in range(len(Country_Polarised_Points_Table1.columns)):
        if Country_Polarised_Points_Table1.columns[i].split(".")[0:3][0] == str(j):
            # I fhte three fist strings (delimited by a full stop '.') ar the same,
            # then it must be the same ESG category. Here we focus on each category
            col.append(i)
    
    for i in range(len(Country_Polarised_Points_Table1.iloc[:,col])):
        # For each category, we tally up the number of observations we have
        SDG_Aggregate_Ranks_col1_j.append(str(len(Country_Polarised_Points_Table1.iloc[i,col].dropna())) + "/" + 
                                         str(len(Country_Polarised_Points_Table1.iloc[i,col])))
        
        # It was decided that only if enough records are found should we consider
        # the median score for a country's ESG category to contain significant insight:
        if len(Country_Polarised_Points_Table1.iloc[i,col].dropna()) > m:
            SDG_Aggregate_Ranks_col2_j.append(Country_Polarised_Points_Table1.iloc[i,col].median())
        else:
            SDG_Aggregate_Ranks_col2_j.append("insufficient scores (<=" + str(m+1) + ")")
    
    SDG_Aggregate_Ranks[(k, "Scores Available")] = SDG_Aggregate_Ranks_col1_j
    SDG_Aggregate_Ranks[(k, "Median Points (Higher is better)")] = SDG_Aggregate_Ranks_col2_j
```

Now we tally up the scores:


```python
# Create lists to be populated
list_of_scores, country_median = [], []

for j in range(len(SDG_Aggregate_Ranks.index)):
    
    # Create lists to be populated
    scores, scores_max, country_median_i = [], [], []
    
    for i in range(0,len(SDG_Aggregate_Ranks.columns),2):
        scores.append(int(SDG_Aggregate_Ranks.iloc[j,i].split("/")[0]))
        scores_max.append(int(SDG_Aggregate_Ranks.iloc[j,i].split("/")[-1]))
    list_of_scores.append(str(sum(scores)) + "/" + str(sum(scores_max)))
    
    for i in range(1,len(SDG_Aggregate_Ranks.columns),2):
        try:
            # The try loop here allows us to account for the lack of sufficient data-points
            country_median_i.append(float(SDG_Aggregate_Ranks.iloc[j,i]))
        except:
            pass
        
    country_median.append(statistics.median(country_median_i))
    
SDG_Aggregate_Ranks[("Overall", "Scores Available")] = list_of_scores
SDG_Aggregate_Ranks[("Overall", "Median Points (Higher is better)")] = country_median
```

$$ \\ $$
# Overall Results Chart

We can now proceed in creating a horizontal bar graph to overview results on a country level


```python
# Create a data-frame from the list of our results this far
Overall_Results_Chart = pd.DataFrame(country_median,
                                     index = Country_Polarised_Points_Table1.index,
                                     columns = ["Overall Results"])
Overall_Results_Chart["Countries"] = list(Country_Polarised_Points_Table1.index)
```

Let's view our results in assending order


```python
Overall_Results_Chart.dropna().sort_values(ascending = True, by = ["Overall Results"])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Overall Results</th>
      <th>Countries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Somalia</th>
      <td>1.00</td>
      <td>Somalia</td>
    </tr>
    <tr>
      <th>Yemen</th>
      <td>2.00</td>
      <td>Yemen</td>
    </tr>
    <tr>
      <th>Eritrea</th>
      <td>2.00</td>
      <td>Eritrea</td>
    </tr>
    <tr>
      <th>Central African Republic</th>
      <td>2.00</td>
      <td>Central African Republic</td>
    </tr>
    <tr>
      <th>Chad</th>
      <td>2.25</td>
      <td>Chad</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>Iceland</th>
      <td>9.00</td>
      <td>Iceland</td>
    </tr>
    <tr>
      <th>Switzerland</th>
      <td>9.00</td>
      <td>Switzerland</td>
    </tr>
    <tr>
      <th>Sweden</th>
      <td>9.00</td>
      <td>Sweden</td>
    </tr>
    <tr>
      <th>Norway</th>
      <td>9.00</td>
      <td>Norway</td>
    </tr>
    <tr>
      <th>Monaco</th>
      <td>9.50</td>
      <td>Monaco</td>
    </tr>
  </tbody>
</table>
<p>211 rows × 2 columns</p>
</div>



## Plots


```python
ax1 = Overall_Results_Chart.dropna().sort_values(ascending = True,
                                                by = ["Overall Results"]).plot.barh(x = "Countries",
                                                                                    y = "Overall Results",
                                                                                    figsize = (10,40),
                                                                                    title = "Overall ESG Scores (out of 10)\n",
                                                                                    grid = True)
```


![png](output_62_0.png)


The cell bellow produces the same chart via Plotly. Plotly offers a range of advantaged and is more dynamic than what is show this far. Note that in order to use plotly you will need the jupyterlab-chart-editor and @jupyterlab/plotly-extension extensions


```python
import plotly.graph_objects as go

dataf = Overall_Results_Chart.dropna().sort_values(ascending = True, by = ["Overall Results"])

fig = go.Figure(go.Bar(x = dataf["Overall Results"],
                       y = dataf["Countries"],
                       orientation = "h"))

fig.show()
```
