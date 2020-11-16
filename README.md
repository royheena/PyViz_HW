# San Francisco Rental Prices Dashboard

In this notebook, you will compile the visualizations from the previous analysis into functions that can be used for a Panel dashboard.


```python
# initial imports

import os
import pandas as pd
import matplotlib.pyplot as plt
import hvplot.pandas
from pathlib import Path

%matplotlib inline

```










```python
# Initialize the Panel Extensions (for Plotly)

import panel as pn
import plotly.express as px
from panel.interact import interact
from panel import widgets

pn.extension('plotly')

```


```python
# Read the Mapbox API key

from dotenv import load_dotenv

load_dotenv("/Users/heenaroy/Desktop/.env")
mapbox_token = os.getenv("MAPBOX_API_KEY")

```

# Import Data


```python
# Import the CSVs to Pandas DataFrames

file_path = Path("Data/sfo_neighborhoods_census_data.csv")
sfo_data = pd.read_csv(file_path)
sfo_data.rename(
    columns={"year":"Year",
             "sale_price_sqr_foot": "Sales Price per Square Foot", 
             "housing_units": "Housing Units", 
             "gross_rent": "Gross Rent"},
    inplace=True)
sfo_data.set_index(["Year"], inplace=True)


file_path = Path("Data/neighborhoods_coordinates.csv")
df_neighborhood_locations = pd.read_csv(file_path)

```

- - -

## Panel Visualizations

In this section, you will copy the code for each plot type from your analysis notebook and place it into separate functions that Panel can use to create panes for the dashboard. 

These functions will convert the plot object to a Panel pane.

Be sure to include any DataFrame transformation/manipulation code required along with the plotting code.

Return a Panel pane object from each function that can be used to build the dashboard.

Note: Remove any `.show()` lines from the code. We want to return the plots instead of showing them. The Panel dashboard will then display the plots.


```python
# Define Panel Visualization Functions

def housing_units_per_year():
    """Housing Units Per Year."""
    
    sfo_housing = sfo_data["Housing Units"].groupby("Year").mean()

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

    return housing_plot


def average_gross_rent():
    """Average Gross Rent in San Francisco Per Year."""
    
    sfo_avg_grent = round(sfo_data.drop(
        columns=["neighborhood","Housing Units"]
        ).groupby("Year").mean(),2)
    
    gross_rent_plot = sfo_avg_grent["Gross Rent"].hvplot(
        xlabel="Year", 
        ylabel="Gross Rent",
        height=500).opts(
        title="Average Gross Rent in San Francisco")
    
    return gross_rent_plot


def average_sales_price():
    """Average Sales Price Per Year."""
    
    sfo_avg_grent = round(sfo_data.drop(
        columns=["neighborhood","Housing Units"]
        ).groupby("Year").mean(),2)
    
    sales_price_sf_plot = sfo_avg_grent["Sales Price per Square Foot"].hvplot(
        xlabel="Year", 
        ylabel="Avg. Sale Price",
        height=500).opts(
        title="Average Sales Price per Square Foot in San Francisco")
    
    return sales_price_sf_plot


def average_price_by_neighborhood():
    """Average Prices by Neighborhood."""
    
    sfo_data_reset_index = sfo_data.reset_index()
    sfo_reset_df = round(
        sfo_data_reset_index.groupby(
            ["Year", "neighborhood"], 
            as_index=False)["Sales Price per Square Foot"].mean(),2)
    neighborhood_list=sfo_data.groupby(
        ["neighborhood"], 
        as_index=False).mean()["neighborhood"].tolist()
    
    def choose_neighborhood(Neighborhood): 

        neighborhood_df = sfo_reset_df.loc[sfo_reset_df["neighborhood"] ==  Neighborhood]

        neighborhood_plot = neighborhood_df.hvplot(
            y="Sales Price per Square Foot", 
            x="Year",
            by="neighborhood",
            ylim=(0, 2500)).opts(
            title=f"Neighborhood: {Neighborhood}")
    
        return neighborhood_plot

    return interact(choose_neighborhood, Neighborhood=neighborhood_list)


def top_most_expensive_neighborhoods():
    """Top 10 Most Expensive Neighborhoods."""
    
    exp_neighbor_top10 = round(sfo_data.groupby(["neighborhood"]).mean().nlargest(10, 
        columns=["Sales Price per Square Foot"]).reset_index(),2)
    
    top_10_plot = exp_neighbor_top10.hvplot(
        kind="bar", 
        x="neighborhood", 
        y="Sales Price per Square Foot", 
        rot=75, 
        xlabel="Neighborhood", 
        ylabel="Avg. Sales Price per Square Foot", 
        height=500).opts(
        title="Top 10 Expensive Neighborhoods in SFO (Avg $/sf)")

    return top_10_plot


def parallel_coordinates():
    """Parallel Coordinates Plot."""
    
    exp_neighbor_top10 = round(sfo_data.groupby(["neighborhood"]).mean().nlargest(10, 
        columns=["Sales Price per Square Foot"]).reset_index(),2)
    
    top_10_pxcoord = px.parallel_coordinates(
        exp_neighbor_top10, 
        color="Sales Price per Square Foot",
        labels={"neighborhood": "Neighborhood"})

    return top_10_pxcoord


def parallel_categories():
    """Parallel Categories Plot."""
    
    exp_neighbor_top10 = round(sfo_data.groupby(["neighborhood"]).mean().nlargest(10, 
        columns=["Sales Price per Square Foot"]).reset_index(),2)
    
    top_10_pxcat = px.parallel_categories(
        exp_neighbor_top10, 
        dimensions=["neighborhood", "Sales Price per Square Foot", "Housing Units", "Gross Rent"], 
        color="Sales Price per Square Foot", 
        color_continuous_scale=px.colors.sequential.Inferno, 
        labels={"neighborhood": "Neighborhood"})

    return top_10_pxcat


def neighborhood_map():
    """Neighborhood Map"""
    
    sfo_neighborhoods=round(sfo_data.groupby(["neighborhood"]).mean().reset_index(),2)
    
    neighborhood_loc=pd.concat(
        (df_neighborhood_locations,sfo_neighborhoods), 
        axis="columns", join="inner").drop(columns=["neighborhood"])
    
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

    return neighborhood_map_plot


```

## Panel Dashboard

In this section, you will combine all of the plots into a single dashboard view using Panel. Be creative with your dashboard design!


```python
# Assign the Content

# row = pn.Row(housing_units_per_year(), average_gross_rent(), average_sales_price())

main_title="# San Francisco Real Estate Investment Market"

welcome = pn.Column(main_title, neighborhood_map())
neighborhood_analysis = pn.Column(main_title, 
                                  "## Average Cost by Neighborhood & 10 Most Expensive Neighborhoods", 
                                  average_price_by_neighborhood(), top_most_expensive_neighborhoods())
yearly_market_analysis = pn.Column(main_title, 
                                   "## Housing Units, Avg Monthly Rental Income & Avg Cost per SqFt", 
                                   housing_units_per_year(), average_gross_rent(), average_sales_price())
parallel_plots_analysis = pn.Column(main_title, 
                                    "## Housing Units & Monthly Rental Income/Cost per SqFt", "### 10 Most Expensive Neighborhoods", 
                                    parallel_coordinates(), parallel_categories())

```


```python
# Create the Tabs

dashboard=pn.Tabs(("Welcome", welcome), 
                  ("Neighborhood Analysis", neighborhood_analysis),
                  ("Yearly Market Analyis", yearly_market_analysis), 
                  ("Parallel Plots Analysis", parallel_plots_analysis))

```

## Serve the Panel Dashboard


```python
dashboard.servable()
```

![welcome_tab](/Images/welcome_tab.png)
![neighborhood_analysis_tab](/Images/neighborhood_analysis_tab.png)
![yearly_market_analysis_tab](/Images/yearly_market_analysis_tab.png)
![parallel_plots_analysis_tab](/Images/parallel_plot_analysis_tab.png)
