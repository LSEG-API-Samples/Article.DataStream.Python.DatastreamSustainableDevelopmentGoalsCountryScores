# DataStream Sustainable Development Goals Country Scores

Environmental, Social and Governance (ESG) data is difficult to come by. It is also becoming critical for effective investment analysis. It helps you assess the risks – and opportunities – posed by companies’ performance in critical areas such as climate change, executive remuneration, and diversity and inclusion. But a definite lack of transparency and standardization in such reporting presents major challenges for investors.
$$ \ $$
This article attempts to lay a framework to allow any investor/agent to collect, analyse and gather insight into countries' ESG metrics at granular and macro-levels. It reflects the DataStream Sustainable Development Goals Country Scores Excel capability.
$$ \\ $$
The file 'ESG-DS.csv' will be needed to collect individual country series codes from our ESG database.

$$ \\ $$
## Get Coding

### Import Libraries
math, statistics, numpy, pandas and openpyxl are needed for dataset manipulation and their statistical and mathematical manipulations.

import math
import statistics
import numpy as np

import pandas as pd
# This line will ensure that all columns of our data-frames are always shown:
pd.set_option('display.max_columns', None)

# xlrd and openpyxl is needed to export data-frames to Excel files.
# The former doesn't need to be imported and can be installed via 'pip install xlrd'
import openpyxl

$$ \\ $$
pickle is only used here to record data so that is doesn't have to be collected every time this code is ran (if it is ran several times)

# need to ' pip install pickle-mixin '
import pickle

$$ \\ $$
We need to gather our data. Since **Refinitiv's [DataStream](https://www.refinitiv.com/en/products/datastream-macroeconomic-analysis) Web Services (DSWS)** allows for access to ESG data covering nearly 70% of global market cap and over 400 metrics, naturally it is more than appropriate. We can access DSWS via the Python library "DatastreamDSWS" that can be installed simply by using $\textit{pip install}$.

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
