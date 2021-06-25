---
layout: post
title:  "Input from Tableau Server into Alteryx Workflow"
date:   2021-04-10
image:  images/100421-cover.png
---
A week or so ago I accidentally claimed an Alteryx tool existed that didn't actually exist. I promise I had no intention to fool anyone, I was simply labouring under a misapprehension. For some reason, I was convinced that sitting beside the *Publish to Tableau Server* tool was an equally useful *Input from Tableau Server* tool. While I'm sure somewhere in the ideas forums this has been requested and re-requested, it does not - as of yet - exist.

Nevertheless, one can see the use case. If you have data available on Tableau Server and nowhere else, or you want to append to a Tableau data source but have no idea what data already exists in that data source then having an input tool connecting straight to Tableau Server is ideal.

In that vein, I decided to use the Python tool to write a script that would download a Tableau Server data source into a temp folder, produce the filepath of the hyper file, so a dynamic input or input macro could then input the hyper file into an Alteryx workflow.

First of all, you need to know the datasource ID - to find this I created another [standalone script](https://github.com/CRMarland/tableauservertoalteryx/blob/master/tabserverdatasourceids.py) that I think would work best in an Alteryx app (otherwise you're running the same thing over and over again to get an ID that never changes).

```python
from ayx import Alteryx
import pandas as pd
import numpy as np
import tableauserverclient as TSC

tableau_auth = TSC.TableauAuth('USERNAME', 'PASSWORD', site='SITE')
server = TSC.Server('https://SERVER-URL')

server.auth.sign_in(tableau_auth)

with server.auth.sign_in(tableau_auth):
    request_options = TSC.RequestOptions(pagesize=1000)
    all_datasources = list(TSC.Pager(server.datasources))
    index = 0
    datasources = {}
    for ds in all_datasources:
        datasources[index] = [ds.name, ds.id]
        index +=1
        print(datasources)
       
df = pd.DataFrame.from_dict(datasources, orient='index')

df.rename({0:'Datasource_Name', 1:'Datasource_ID'}, axis='columns', inplace=True)

Alteryx.write(df, 1)
```
<br/>
This code is very simple and very brief so I've (*naughty, I know*) not documented it throughout. In terms of understanding it, what I'm doing is:

1. Importing packages

2. Storing credentials and the server's URL into variables, then using them to start a with statement

3. Setting the age size of the request to 1,000 (datasources are paginated by the TSC library and, by default, the page size is 300)

4. Using TSC.Pager() to retrieve all datasources and storing them in the all_datasources variable

5. Storing each datasource's name and ID into a dictionary

6. Creating a dataframe from that dictionary and renaming the column headers before writing back to Alteryx

Now we have the datasourcID, we can plug it into the [next bit of code](https://github.com/CRMarland/tableauservertoalteryx/blob/master/tableaudatasourcedownload.py) that I put into a macro.

```python
#Import packages

from ayx import Package
from ayx import Alteryx
import tableauserverclient as TSC
import pandas as pd
import numpy as np
import zipfile
import re
from os import listdir
from os.path import isfile, join

# Authenticate with your Tableau Server

tableau_auth = TSC.TableauAuth('USERNAME', 'PASSWORD', site='SITE')
server = TSC.Server('https://SERVER-URL')

# Download datasource and save the filepath in data_loc variable

with server.auth.sign_in(tableau_auth):
    data_loc = server.datasources.download('DATASOURCE-ID')

# Use RegEx to extract the filepath minus the filename from the data_loc variable

m = re.match('(.*)\\\\.*$', data_loc)
filepath = m.group(1)

# Unzip the tdsx file (using the data_loc variable) and extract it to the folder
# (using the filepath variable)

with zipfile.ZipFile(data_loc, 'r') as zip_ref:
    zip_ref.extractall(filepath)

# Create a data_path variable that points to the location of the extracted hyper file
# created by the previous step.
# Get the name of the original download and compare that to the files in the 
# unzipped folder - it may have a suffix. 

hyper_location = filepath + '\Data\Extracts'

tdsx_name = re.match(r'.*\\(.*)\.tdsx', data_loc)[1]

hypers_list = os.listdir(hyper_location)

for item in hypers_list:
    tdsx_name_len = len(tdsx_name)
    match_item = item[0:tdsx_name_len]
    if match_item == tdsx_name:
        hyper = item

# Combine the hyper location with the name of the hyper file to create the full path to
# use in an input data tool

final_filepath = hyper_location + '\\' + hyper

# Put the final_filepath variable into a dict so it can be turned into a df and exported
# to Alteryx canvas

dict_to_df = {'data_location': [final_filepath]}

output = pd.DataFrame(data=dict_to_df)

Alteryx.write(output,1)
```
<br/>
From here, you can simply put a dynamic input tool or a batch input macro and your Tableau Server datasource will be input into Alteryx.