# San Francisco Housing Rental Analysis

In this assignment, you will perform basic analysis for the San Francisco Housing Market to allow potential real estate investors to choose rental investment properties. 


```python
# initial imports

import os
import pandas as pd
import matplotlib.pyplot as plt
import hvplot.pandas
import plotly.express as px
from pathlib import Path
from dotenv import load_dotenv

%matplotlib inline

import panel as pn
from panel.interact import interact
from panel import widgets

```


```python
# Read the Mapbox API key

from dotenv import load_dotenv
load_dotenv("/Users/heenaroy/Desktop/.env")
mapbox_token = os.getenv("MAPBOX_API_KEY")

```

## Load Data


```python
# Read the census data into a Pandas DataFrame

file_path = Path("Data/sfo_neighborhoods_census_data.csv")

sfo_data = pd.read_csv(file_path)
sfo_data.rename(columns={"year":"Year","sale_price_sqr_foot": "Sales Price per Square Foot", "housing_units": "Housing Units", "gross_rent": "Gross Rent"}, inplace=True)
sfo_data.set_index(["Year"], inplace=True)

sfo_data.head()

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
      <th>neighborhood</th>
      <th>Sales Price per Square Foot</th>
      <th>Housing Units</th>
      <th>Gross Rent</th>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010</th>
      <td>Alamo Square</td>
      <td>291.182945</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Anza Vista</td>
      <td>267.932583</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Bayview</td>
      <td>170.098665</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Buena Vista Park</td>
      <td>347.394919</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>Central Richmond</td>
      <td>319.027623</td>
      <td>372560</td>
      <td>1239</td>
    </tr>
  </tbody>
</table>
</div>



- - - 

## Housing Units Per Year

In this section, you will calculate the number of housing units per year and visualize the results as a bar chart using the Pandas plot function. 

Hint: Use the Pandas groupby function

Optional challenge: Use the min, max, and std to scale the y limits of the chart.


```python
# Calculate the mean number of housing units per year (hint: use groupby) 

sfo_housing = sfo_data["Housing Units"].groupby("Year").mean()

sfo_housing

```




    Year
    2010    372560
    2011    374507
    2012    376454
    2013    378401
    2014    380348
    2015    382295
    2016    384242
    Name: Housing Units, dtype: int64




```python
# Use the Pandas plot function to plot the average housing units per year.

housing_plot = sfo_housing.hvplot.bar(
    ylim=(sfo_housing.min() - sfo_housing.std(), 
          sfo_housing.max() + sfo_housing.std()), 
    xlabel="Year",
    ylabel="Housing Units", 
    rot=90,
    height=500).opts(
    yformatter="%.0f", 
    hover_line_color="red", 
    title="Housing Units in San Francisco from 2010 to 2016")

housing_plot

```

![housing_plot](/Images/housing_plot.png)


## Average Prices per Square Foot

In this section, you will calculate the average gross rent and average sales price for each year. Plot the results as a line chart.

### Average Gross Rent in San Francisco Per Year


```python
# Calculate the average gross rent and average sale price per square foot

sfo_avg_grent = round(sfo_data.drop(columns=["neighborhood","Housing Units"]).groupby("Year").mean(),2)

sfo_avg_grent

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
      <th>Sales Price per Square Foot</th>
      <th>Gross Rent</th>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2010</th>
      <td>369.34</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>2011</th>
      <td>341.90</td>
      <td>1530</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>399.39</td>
      <td>2324</td>
    </tr>
    <tr>
      <th>2013</th>
      <td>483.60</td>
      <td>2971</td>
    </tr>
    <tr>
      <th>2014</th>
      <td>556.28</td>
      <td>3528</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>632.54</td>
      <td>3739</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>697.64</td>
      <td>4390</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plot the Average Gross Rent per Year as a Line Chart 

gross_rent_plot = sfo_avg_grent["Gross Rent"].hvplot(
    xlabel="Year", 
    ylabel="Gross Rent",
    height=500).opts(
    title="Average Gross Rent in San Francisco")

gross_rent_plot

```


![gross_rent_plot](/Images/gross_rent_plot.png)



### Average Sales Price per Year


```python
# Plot the Average Sales Price per Year as a line chart

sales_price_sf_plot = sfo_avg_grent["Sales Price per Square Foot"].hvplot(
    xlabel="Year", 
    ylabel="Avg Sales Price per SqFt",
    height=500).opts(
    title="Average Sales Price per Square Foot in San Francisco")

sales_price_sf_plot

```
![sales_price_sf_plot](/Images/sales_price_sf_plot.png)


- - - 

## Average Prices by Neighborhood

In this section, you will use hvplot to create an interactive visulization of the Average Prices with a dropdown selector for the neighborhood.

Hint: It will be easier to create a new DataFrame from grouping the data and calculating the mean prices for each year and neighborhood


```python
# Group by year and neighborhood and then create a new dataframe of the mean values

sfo_data_reset_index = sfo_data.reset_index()

sfo_reset_df = round(sfo_data_reset_index.groupby(["Year", "neighborhood"], as_index=False)["Sales Price per Square Foot"].mean(),2)

sfo_reset_df.head()

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
      <th>Year</th>
      <th>neighborhood</th>
      <th>Sales Price per Square Foot</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2010</td>
      <td>Alamo Square</td>
      <td>291.18</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2010</td>
      <td>Anza Vista</td>
      <td>267.93</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2010</td>
      <td>Bayview</td>
      <td>170.10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2010</td>
      <td>Buena Vista Park</td>
      <td>347.39</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2010</td>
      <td>Central Richmond</td>
      <td>319.03</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Define function to choose a neighborhood

def choose_neighborhood(Neighborhood): 

    neighborhood_df = sfo_reset_df.loc[sfo_reset_df["neighborhood"] ==  Neighborhood]

    neighborhood_plot = neighborhood_df.hvplot(
        y="Sales Price per Square Foot", 
        x="Year",
        ylim=(0, 2500), 
        by="neighborhood").opts(
        title=f"Neighborhood: {Neighborhood}")
    
    return neighborhood_plot


# Call the choose_neighborhood function to return plot with interactivity

neighborhood_list=sfo_data.groupby(["neighborhood"], as_index=False).mean()["neighborhood"].tolist()

interact(choose_neighborhood, Neighborhood=neighborhood_list)

```

![interact_neighborhood](/Images/interact_neighborhood.png)


- - - 

## The Top 10 Most Expensive Neighborhoods

In this section, you will need to calculate the mean sale price for each neighborhood and then sort the values to obtain the top 10 most expensive neighborhoods on average. Plot the results as a bar chart.


```python
# Getting the data from the top 10 expensive neighborhoods

exp_neighbor_top10 = round(sfo_data.groupby(["neighborhood"]).mean().nlargest(10, columns=["Sales Price per Square Foot"]).reset_index(),2)

exp_neighbor_top10

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
      <th>neighborhood</th>
      <th>Sales Price per Square Foot</th>
      <th>Housing Units</th>
      <th>Gross Rent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Union Square District</td>
      <td>903.99</td>
      <td>377427.50</td>
      <td>2555.17</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Merced Heights</td>
      <td>788.84</td>
      <td>380348.00</td>
      <td>3414.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Miraloma Park</td>
      <td>779.81</td>
      <td>375967.25</td>
      <td>2155.25</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Pacific Heights</td>
      <td>689.56</td>
      <td>378401.00</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Westwood Park</td>
      <td>687.09</td>
      <td>382295.00</td>
      <td>3959.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Telegraph Hill</td>
      <td>676.51</td>
      <td>378401.00</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Presidio Heights</td>
      <td>675.35</td>
      <td>378401.00</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Cow Hollow</td>
      <td>665.96</td>
      <td>378401.00</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Potrero Hill</td>
      <td>662.01</td>
      <td>378401.00</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>9</th>
      <td>South Beach</td>
      <td>650.12</td>
      <td>375805.00</td>
      <td>2099.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plotting the data from the top 10 expensive neighborhoods

top_10_plot = exp_neighbor_top10.hvplot(
    kind="bar", 
    x="neighborhood", 
    y="Sales Price per Square Foot", 
    rot=75, 
    xlabel="Neighborhood", 
    ylabel="Avg. Sales Price per Square Foot", 
    height=500).opts(
    title="Top 10 Expensive Neighborhoods in SFO (Avg $/sf)")

top_10_plot

```
![top_10_plot](/Images/top_10_plot.png)



- - - 

## Parallel Coordinates and Parallel Categories Analysis

In this section, you will use plotly express to create parallel coordinates and parallel categories visualizations so that investors can interactively filter and explore various factors related to the sales price of the neighborhoods. 

Using the DataFrame of Average values per neighborhood (calculated above), create the following visualizations:
1. Create a Parallel Coordinates Plot
2. Create a Parallel Categories Plot


```python
# Parallel Coordinates Plot

top_10_pxcoord = px.parallel_coordinates(
    exp_neighbor_top10, 
    color="Sales Price per Square Foot",
    labels={"neighborhood": "Neighborhood"})

top_10_pxcoord

```
![top_10_pxcoord](/Images/top_10_pxcoord)


```python
# Parallel Categories Plot


top_10_pxcat = px.parallel_categories(
    exp_neighbor_top10, 
    dimensions=["neighborhood", "Sales Price per Square Foot", "Housing Units", "Gross Rent"], 
    color="Sales Price per Square Foot", 
    color_continuous_scale=px.colors.sequential.Inferno, 
    labels={"neighborhood": "Neighborhood"})

top_10_pxcat

```
![top_10_pxcat](/Images/top_10_pxcat.png)


- - - 

## Neighborhood Map

In this section, you will read in neighboor location data and build an interactive map with the average prices per neighborhood. Use a scatter_mapbox from plotly express to create the visualization. Remember, you will need your mapbox api key for this.

### Load Location Data


```python
# Load neighborhoods coordinates data

file_path = Path("Data/neighborhoods_coordinates.csv")
df_neighborhood_locations = pd.read_csv(file_path)

df_neighborhood_locations.head()

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
      <th>Neighborhood</th>
      <th>Lat</th>
      <th>Lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alamo Square</td>
      <td>37.791012</td>
      <td>-122.402100</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anza Vista</td>
      <td>37.779598</td>
      <td>-122.443451</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bayview</td>
      <td>37.734670</td>
      <td>-122.401060</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bayview Heights</td>
      <td>37.728740</td>
      <td>-122.410980</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bernal Heights</td>
      <td>37.728630</td>
      <td>-122.443050</td>
    </tr>
  </tbody>
</table>
</div>



### Data Preparation

You will need to join the location data with the mean prices per neighborhood

1. Calculate the mean values for each neighborhood
2. Join the average values with the neighborhood locations


```python
# Calculate the mean values for each neighborhood

sfo_neighborhoods=round(sfo_data.groupby(["neighborhood"]).mean().reset_index(),2)

sfo_neighborhoods.head()

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
      <th>neighborhood</th>
      <th>Sales Price per Square Foot</th>
      <th>Housing Units</th>
      <th>Gross Rent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alamo Square</td>
      <td>366.02</td>
      <td>378401.0</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anza Vista</td>
      <td>373.38</td>
      <td>379050.0</td>
      <td>3031.83</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bayview</td>
      <td>204.59</td>
      <td>376454.0</td>
      <td>2318.40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bayview Heights</td>
      <td>590.79</td>
      <td>382295.0</td>
      <td>3739.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bernal Heights</td>
      <td>576.75</td>
      <td>379374.5</td>
      <td>3080.33</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Join the average values with the neighborhood locations

neighborhood_loc=pd.concat((
    df_neighborhood_locations, sfo_neighborhoods), 
    axis="columns", join="inner").drop(columns=["neighborhood"])

neighborhood_loc.head()

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
      <th>Neighborhood</th>
      <th>Lat</th>
      <th>Lon</th>
      <th>Sales Price per Square Foot</th>
      <th>Housing Units</th>
      <th>Gross Rent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alamo Square</td>
      <td>37.791012</td>
      <td>-122.402100</td>
      <td>366.02</td>
      <td>378401.0</td>
      <td>2817.29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Anza Vista</td>
      <td>37.779598</td>
      <td>-122.443451</td>
      <td>373.38</td>
      <td>379050.0</td>
      <td>3031.83</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bayview</td>
      <td>37.734670</td>
      <td>-122.401060</td>
      <td>204.59</td>
      <td>376454.0</td>
      <td>2318.40</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bayview Heights</td>
      <td>37.728740</td>
      <td>-122.410980</td>
      <td>590.79</td>
      <td>382295.0</td>
      <td>3739.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bernal Heights</td>
      <td>37.728630</td>
      <td>-122.443050</td>
      <td>576.75</td>
      <td>379374.5</td>
      <td>3080.33</td>
    </tr>
  </tbody>
</table>
</div>



### Mapbox Visualization

Plot the average values per neighborhood with a plotly express scatter_mapbox visualization.


```python
# Create a scatter mapbox to analyze neighborhood info

px.set_mapbox_access_token(mapbox_token)

neighborhood_map_plot = px.scatter_mapbox(
    neighborhood_loc,
    lat="Lat",
    lon="Lon",
    size="Sales Price per Square Foot",
    color="Gross Rent",
    hover_name="Neighborhood",
    zoom=11,
    color_continuous_scale=px.colors.cyclical.IceFire,
    title="Average Sales Price per Square Foot & Gross Rent in San Francisco")

neighborhood_map_plot.show()

```
![neighborhood_map_plot](/Images/neighborhood_map_plot.png)

