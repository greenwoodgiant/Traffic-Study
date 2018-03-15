

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
import plotly.plotly as py
import plotly
import plotly.graph_objs as go
import plotly.figure_factory as ff
%matplotlib inline
```


```python
plotly.tools.set_credentials_file(username='hoanganh3530', api_key='tFEHTE3m211l4G3RFkmI')
```


```python
# Read csv files
speed_10 = pd.read_csv("TX 2010 Speed Related Crashes Data.csv",low_memory=False)
speed_11 = pd.read_csv("TX 2011 Speed Related Crashes Data.csv",low_memory=False)
speed_12 = pd.read_csv("TX 2012 Speed Related Crashes Data.csv",low_memory=False)
speed_13 = pd.read_csv("TX 2013 Speed Related Crashes Data.csv",low_memory=False)
speed_14 = pd.read_csv("TX 2014 Speed Related Crashes Data.csv",low_memory=False)
speed_15 = pd.read_csv("TX 2015 Speed Related Crashes Data.csv",low_memory=False)
speed_16 = pd.read_csv("TX 2016 Speed Related Crashes Data.csv",low_memory=False)
speed_17 = pd.read_csv("TX 2017 Speed Related Crashes Data.csv",low_memory=False)
```


```python
#Read the fips file for Texas and modify it
fips = pd.read_excel("Texas_FIPS.xlsx").astype(str)
county_fips = []
for county in fips['COUNTYFP']:
    if len(str(county)) == 1: 
        county = str("00") + str(county) 
        county_fips.append(county)
    elif len(str(county)) == 2:
        county = str("0") + str(county)
        county_fips.append(county)
    else: 
        county = str(county)
        county_fips.append(county)
fips['COUNTYFP'] = county_fips
fips['FIPS'] = fips['STATEFP'] + fips['COUNTYFP']
fips.head()
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
      <th>STATE</th>
      <th>STATEFP</th>
      <th>COUNTYFP</th>
      <th>COUNTYNAME</th>
      <th>CLASSFP</th>
      <th>FIPS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>TX</td>
      <td>48</td>
      <td>001</td>
      <td>Anderson County</td>
      <td>H1</td>
      <td>48001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TX</td>
      <td>48</td>
      <td>003</td>
      <td>Andrews County</td>
      <td>H1</td>
      <td>48003</td>
    </tr>
    <tr>
      <th>2</th>
      <td>TX</td>
      <td>48</td>
      <td>005</td>
      <td>Angelina County</td>
      <td>H1</td>
      <td>48005</td>
    </tr>
    <tr>
      <th>3</th>
      <td>TX</td>
      <td>48</td>
      <td>007</td>
      <td>Aransas County</td>
      <td>H1</td>
      <td>48007</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TX</td>
      <td>48</td>
      <td>009</td>
      <td>Archer County</td>
      <td>H1</td>
      <td>48009</td>
    </tr>
  </tbody>
</table>
</div>




```python
def clean_data(csv_file):
    csv_file = csv_file[['Crash ID','Latitude', 'Longitude', 'City', 'County','Crash Year', 'Crash Death Count','Crash Total Injury Count', 'Speed Limit', 'Weather Condition', 'Person Ethnicity', 'Person Gender', 'Person Type']]
    csv_file = csv_file[csv_file['Person Type'] == "Driver"]
    csv_file = csv_file[csv_file['Speed Limit'] >= 0]
    #Write the for loop to remove all those data that have "No Data" entry
    csv_file = csv_file.replace(to_replace = "No Data", value = np.nan)
    csv_file = csv_file.dropna(axis = 0, how = "any")
    return csv_file
        
```


```python
#Create a list contains all the file and reset the data with a clean data
csv_file = [speed_10, speed_11, speed_12, speed_13, speed_14, speed_15, speed_16, speed_17]
new_file = []
for file in csv_file: 
    file = clean_data(file)
    county = [] 
    fips_code = []
    for element in file['County']: 
        element = element + str(" County")
        county.append(element)
        # Make a new columns for the FIPS code
        if element in list(fips['COUNTYNAME']):
            i = (list(fips['COUNTYNAME']).index(element))
            fips_code.append(fips['FIPS'][i])
        else: 
            fips_code.append(np.nan)
    file['County'] = county
    file['FIPS_CODE'] = fips_code
    #Drop NA again
    file = file.dropna(axis = 0, how = "any")
    new_file.append(file)
```


```python
# Combine all the dataframe into 1 dataframe
for i in range (0, len(new_file) - 1):
    new_file[i+1] = pd.concat([new_file[i], new_file[i + 1]])
combine_data = new_file[7]
```


```python
#Drop duplicate Crash ID
new_data = combine_data.drop_duplicates('Crash ID')
new_data.head()
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
      <th>Crash ID</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
      <th>County</th>
      <th>Crash Year</th>
      <th>Crash Death Count</th>
      <th>Crash Total Injury Count</th>
      <th>Speed Limit</th>
      <th>Weather Condition</th>
      <th>Person Ethnicity</th>
      <th>Person Gender</th>
      <th>Person Type</th>
      <th>FIPS_CODE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11154479</td>
      <td>33.66384399</td>
      <td>-95.53569239</td>
      <td>Paris</td>
      <td>Lamar County</td>
      <td>2010</td>
      <td>0</td>
      <td>1</td>
      <td>30</td>
      <td>Cloudy</td>
      <td>Black</td>
      <td>Male</td>
      <td>Driver</td>
      <td>48277</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11154515</td>
      <td>34.13157119</td>
      <td>-99.13679543</td>
      <td>Rural Wilbarger County</td>
      <td>Wilbarger County</td>
      <td>2010</td>
      <td>0</td>
      <td>0</td>
      <td>70</td>
      <td>Cloudy</td>
      <td>White</td>
      <td>Female</td>
      <td>Driver</td>
      <td>48487</td>
    </tr>
    <tr>
      <th>5</th>
      <td>11155308</td>
      <td>27.58794537</td>
      <td>-99.52219213</td>
      <td>Laredo</td>
      <td>Webb County</td>
      <td>2010</td>
      <td>0</td>
      <td>1</td>
      <td>30</td>
      <td>Clear</td>
      <td>Hispanic</td>
      <td>Female</td>
      <td>Driver</td>
      <td>48479</td>
    </tr>
    <tr>
      <th>8</th>
      <td>11156835</td>
      <td>32.83661427</td>
      <td>-97.59572236</td>
      <td>Rural Parker County</td>
      <td>Parker County</td>
      <td>2010</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Clear</td>
      <td>White</td>
      <td>Male</td>
      <td>Driver</td>
      <td>48367</td>
    </tr>
    <tr>
      <th>13</th>
      <td>11158273</td>
      <td>32.83082038</td>
      <td>-96.61662323</td>
      <td>Mesquite</td>
      <td>Dallas County</td>
      <td>2010</td>
      <td>0</td>
      <td>0</td>
      <td>45</td>
      <td>Cloudy</td>
      <td>Black</td>
      <td>Male</td>
      <td>Driver</td>
      <td>48113</td>
    </tr>
  </tbody>
</table>
</div>




```python
weather_data = new_data[['Crash Year', 'Weather Condition','Crash Total Injury Count', 'Crash Death Count']]
weather_data.head()
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
      <th>Crash Year</th>
      <th>Weather Condition</th>
      <th>Crash Total Injury Count</th>
      <th>Crash Death Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2010</td>
      <td>Cloudy</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2010</td>
      <td>Cloudy</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2010</td>
      <td>Clear</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2010</td>
      <td>Clear</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2010</td>
      <td>Cloudy</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
weather_data = weather_data.groupby(by = ['Crash Year', 'Weather Condition'])
weather_data = pd.DataFrame(weather_data['Crash Total Injury Count', 'Crash Death Count'].sum())
weather_data = weather_data.reset_index()
weather_data.head()
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
      <th>Crash Year</th>
      <th>Weather Condition</th>
      <th>Crash Total Injury Count</th>
      <th>Crash Death Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2010</td>
      <td>Blowing Sand/Snow</td>
      <td>9</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2010</td>
      <td>Clear</td>
      <td>7969</td>
      <td>458</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2010</td>
      <td>Cloudy</td>
      <td>1762</td>
      <td>107</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2010</td>
      <td>Fog</td>
      <td>166</td>
      <td>14</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2010</td>
      <td>Other (Explain In Narrative)</td>
      <td>30</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
death_count = []
for weather in weather_data['Weather Condition'].unique():
    trace = go.Bar(
        x= list(weather_data['Crash Year'].unique()),
        y= list(weather_data.loc[weather_data['Weather Condition'] == weather, ['Crash Death Count']]['Crash Death Count']),
        name= weather)
    death_count.append(trace)
data = death_count
layout = go.Layout(
    barmode='stack'
)
fig = go.Figure(data=data, layout=layout)
py.iplot(fig, filename='Crash_Death_Count')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~hoanganh3530/2.embed" height="525px" width="100%"></iframe>




```python
injury_count = []
for weather in weather_data['Weather Condition'].unique():
    trace = go.Bar(
        x= list(weather_data['Crash Year'].unique()),
        y= list(weather_data.loc[weather_data['Weather Condition'] == weather, ['Crash Total Injury Count']]['Crash Total Injury Count']),
        name= weather)
    injury_count.append(trace)
data = injury_count
layout = go.Layout(
    barmode='stack'
)
fig = go.Figure(data=data, layout=layout)
py.iplot(fig, filename='Crash_Total_Injury_Count')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~hoanganh3530/4.embed" height="525px" width="100%"></iframe>




```python
map_data = new_data[['FIPS_CODE', 'Crash Death Count', 'Crash Total Injury Count']]
map_data = map_data.groupby(by = ['FIPS_CODE'])
map_data = pd.DataFrame(map_data['Crash Total Injury Count', 'Crash Death Count'].sum())
map_data = map_data.reset_index()
map_data.head()
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
      <th>FIPS_CODE</th>
      <th>Crash Total Injury Count</th>
      <th>Crash Death Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>48001</td>
      <td>554</td>
      <td>25</td>
    </tr>
    <tr>
      <th>1</th>
      <td>48003</td>
      <td>155</td>
      <td>14</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48005</td>
      <td>407</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3</th>
      <td>48007</td>
      <td>74</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>48009</td>
      <td>71</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
values = map_data['Crash Total Injury Count'].tolist()
fips = map_data['FIPS_CODE'].tolist()
colorscale = ["#030512","#1d1d3b","#323268","#3d4b94","#3e6ab0",
              "#4989bc","#60a7c7","#85c5d3","#b7e0e4","#eafcfd"]
endpts = list(np.mgrid[min(values):max(values):4j])
fig = ff.create_choropleth(
    fips=fips, values=values, scope=['Texas'], show_state_data=True,
    colorscale=colorscale, binning_endpoints=endpts, round_legend_values=True,
    plot_bgcolor='rgb(229,229,229)',
    paper_bgcolor='rgb(229,229,229)',
    legend_title='Total Crash Injury by County',
    county_outline={'color': 'rgb(255,255,255)', 'width': 0.5},
    exponent_format=True,
)
py.iplot(fig, filename='choropleth_texas')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~hoanganh3530/6.embed" height="450px" width="900px"></iframe>


