# Standardized Precipitation Index (SPI) Dashboard 

> The Standardized Precipitation Index (SPI), a drought index widely used for drought monitoring, is estimated based on the probability of  deficit in precipitation at any time scale. It is a simple and flexible  drought index as it requires only precipitation as input data and it can quantify the degree of dryness/wetness on multiple timescales. *(Mohapatra et al., 2019)* 

## Introduction

In the world of Data Science, the Dashboards has been a popular option for visualizing data on a web platform. With the availability of high level web frameworks and APIs, making a dashboard is not that difficult now-a-days. I have been a hardcore Python developer for a lot of time now, web development was never a topic that I was interested in. Recently i came across Django, Flask and Dash web frameworks, which enabled Python developers to develop responsive, rich and functional web pages. The one library, that stood apart from others was Dash. Dash is like Django and Flask, but has a special tendency towards data visualization. Dash is a really powerful framework for creating responsive and good looking Dashboard; along with the power of Bootstrap and CSS3. A list of cool Dash apps can be seen on the [Dash App Gallery](https://dash-gallery.plotly.host/Portal/). If you want to learn Dash, the Udemy course ["Interactive Python Dashboards with Plotly and Dash" by Jose Portilla](https://www.udemy.com/share/101WaO/) is by far the best resource i have com across and i recommend it.

## Objective of this project

The current project aims to develop a dashboard to visualize annual SPI value rasters for various timescales generated using [SPIUtility](https://manaruchi.github.io/spiutility.html) plugin for QGIS along with some extra data visualization here-there. If you want to know more about how the SPI values are generated using SPIUtility, visit the [SPIUtility documentation.](https://manaruchi.github.io/spiutility.html) 

## The final product

After the end of this article, we should have a Dashboard which would look like the following. The dashboard is now up and live at [SPIDash](http://spidash.herokuapp.com/)

![final_layout](/images/final_layout.png)

The main elements in the layout are, the top navigation bar (aka the title bar), a control bar (with options to choose the timescale, month and year for which the SPI values to be loaded), a choropleth map and a Pie chart inside a Tab element.

## The Python Script

At first, lets build the skeleton of the python script that will be used. Initially, we import the required libraries.

```python
import dash
import dash_html_components as html
import dash_core_components as dcc
import dash_bootstrap_components as dbc
import plotly.graph_objs as go
from dash.dependencies import Input, Output, State
import numpy as np
import pandas as pd
import json
```

The Dash object definition has to include the Bootstrap component external stylesheet as:

```python
app = dash.Dash(external_stylesheets=[dbc.themes.BOOTSTRAP])
```

In the next step, we load the GeoJSON file and one corresponding CSV file.

```python
#IMPORT JSON FILE
with open(r"G:\Manav\IMD\JSON\ID_INDIA_MOD.json") as fp:
    india = json.load(fp)

#IMPORT CSV FILE
df = pd.read_csv(r"G:\Manav\IMD\SPI_CSV\SPI_1901_1.csv")
```

Now, let's define the `go.Choroplethmapbox` figure object.

```python
trace = [go.Choroplethmapbox(geojson=india, locations=df.ID, z=df.SPI,
                                    colorscale="RdBu", zmin=-2, zmax=2, marker_opacity=0.7, 		marker_line_width = 0)]
fig = go.Figure(data=trace)
fig.update_layout(mapbox_style="carto-positron",
                  mapbox_zoom=3.5, mapbox_center = {"lat": 23, "lon": 80})
fig.update_layout(height = 550, margin={"r":0,"t":0,"l":0,"b":0})
```

For the pie chart, we will have to classify the SPI values in the dataframe into 9 classes and find the number of values in each classes. Then we need to define the pie chart figure object.

```python
#Data for Pie Chart
SPI_values = df['SPI'].values

spi_categories_for_pie_chart = ['Extreme Wet', 'Severe Wet', 'Moderate Wet', 'Mild Wet', 'Normal', 'Mild Dry', 'Moderate Dry', 'Severe Dry', 'Extreme Dry']
count_extreme_wet = len(np.where(SPI_values >= 2)[0])
count_severe_wet = len(np.where((SPI_values >= 1.5)&(SPI_values < 2))[0])
count_mod_wet = len(np.where((SPI_values >= 1.0)&(SPI_values < 1.5))[0])
count_mild_wet = len(np.where((SPI_values >= 0.5)&(SPI_values < 1))[0])
count_normal = len(np.where((SPI_values >= -0.5)&(SPI_values < 0.5))[0])
count_mild_dry = len(np.where((SPI_values >= -1.0)&(SPI_values < -0.5))[0])
count_mod_dry = len(np.where((SPI_values >= -1.5)&(SPI_values < -1.0))[0])
count_severe_dry = len(np.where((SPI_values >= -2.0)&(SPI_values < -1.5))[0])
count_extreme_dry = len(np.where(SPI_values < -2.0)[0])
values_for_pie_chart = [count_extreme_wet, count_severe_wet, count_mod_wet, count_mild_wet, count_normal,count_mild_dry,count_mod_dry, count_severe_dry, count_extreme_dry]

pie_figure = go.Figure(data=[go.Pie(labels=spi_categories_for_pie_chart, values=values_for_pie_chart, 			hole=.3)], layout = go.Layout(title = "Timescale: 1 month, Month: January, Year: 2018"))

```

To add a navigation bar, we need to use `dbc.NavbarSimple()` element.

```python
navigation_bar_element = dbc.NavbarSimple(children=[dbc.DropdownMenu(children=[
                dbc.DropdownMenuItem("SPIUtility", href="#"),
                dbc.DropdownMenuItem("Github Source", href="#"),
            ],
            nav=True,
            in_navbar=True,
            label="Resources",)
                              
            ], brand="Drought Monitoring Dashboard", color = 'dark', dark = True)
```

Below the navigation bar, we have 3 combo boxes (`timescale-picker`, `month-picker` and `year-picker`) and one button in one `html.Div()`. The combo boxes are from `dcc.Dropdown()` and the button is from `dbc.Button()`.

```python
#Lets define the options in the dropdowns first.
#Define timescale picker options
timescale_options = [{'label': '1 month', 'value': '1'},
                    {'label': '3 months', 'value': '3'},
                    {'label': '12 months', 'value': '12'}]

#Define month picker options - initially for 1 monthly SPI, later the list options will be updated according to timescale
month_options = [{'label': 'January', 'value': '1'},
                {'label': 'February', 'value': '2'},
                {'label': 'March', 'value': '3'},
                {'label': 'April', 'value': '4'},
                {'label': 'May', 'value': '5'},
                {'label': 'June', 'value': '6'},
                {'label': 'July', 'value': '7'},
                {'label': 'August', 'value': '8'},
                {'label': 'September', 'value': '9'},
                {'label': 'October', 'value': '10'},
                {'label': 'November', 'value': '11'},
                {'label': 'December', 'value': '12'}]

#Define year-picker options
year_options = []

for year in range(1901,2019):
    year_options.append({'label': str(year), 'value': str(year)})
    
#Define the timescale-picker element
timescale_picker_element = dcc.Dropdown(id = 'timescale-picker', options = timescale_options, value = '1', searchable=False, style = {'font-family': 'tahoma','font-size': '12px'})

#Define the month-picker element
month_picker_element = dcc.Dropdown(id = 'month-picker', options = month_options, value = '1', searchable=False, style = {'font-family': 'tahoma','font-size': '12px'})

#Define the year-picker element
year_picker_element = dcc.Dropdown(id = 'year-picker', options = year_options, value = '2018', searchable=False, style = {'font-family': 'tahoma','font-size': '12px'})

#Define the Load Data Button
load_data_button_element = dbc.Button("Load Data", id = 'submit-button', color="primary", className="mr-1")

#Put all of these into a html.Div() with some CSS styling
toolbox_div = html.Div([
            
 		html.Div(html.Label('Timescale ', style = {'font-family': 'tahoma', 'font-size': '14px', 'text-align': 'right', 'color': 'white'}), style = {'float': 'left', 'width': '8%', 'padding-top': '7px', 'padding-bottom': '5px', 'padding-left': '5px', 'padding-right': '5px'}),
            
    timescale_picker_element,
    
            html.Div(html.Label('Month ', style = {'font-family': 'tahoma', 'font-size': '14px', 'text-align': 'right', 'color': 'white'}), style = {'float': 'left', 'width': '5%', 'padding-top': '7px', 'padding-bottom': '5px', 'padding-left': '10px', 'padding-right': '5px'}),
    
     month_picker_element,
    
            html.Div(html.Label('Year ', style = {'font-family': 'tahoma', 'font-size': '14px', 'text-align': 'right', 'color': 'white'}), style = {'float': 'left', 'width': '3%', 'padding-top': '7px', 'padding-bottom': '5px', 'padding-left': '5px', 'padding-right': '5px'}),
            
    year_picker_element, load_data_button_element], 
    
    style = {'float': 'left', 'width': '100%', 'padding-top': '5px', 'padding-bottom': '5px', 'background-color': '#343a40'}),
        
```

Now putting every element in `app.layout` along with some CSS styling, we get the main layout of the Dashboard. To show that certain element is loading, `dbc.Spinner` can be used.

```python
#Define Right hand side tabs
tab1_content = dbc.Card(
    dbc.CardBody(
        [
            dbc.Spinner(dcc.Graph(id='pie-chart', figure = pie_figure),color="primary", type="grow")
        ]
    ),
    className="mt-3",
)

tab2_content = dbc.Card(
    dbc.CardBody(
        [
            html.Div(id='spi-timeseries', children = [html.P("Click on the map to see the time series annual SPI value graph.", className="card-text")])
            
        ]
    ),
    className="mt-3",
)
```

```python
app.layout = dbc.Spinner(html.Div([navigation_bar_element,
                                   toolbox_div,
                                   html.Div([html.Div(dbc.Spinner(dcc.Graph(id = 'map-plot', figure = fig),color="primary", type="grow"), style = {'float': 'left','height': '80%', 'width': '65%'}),
    html.Div(dbc.Tabs([dbc.Tab(tab1_content, label="Drought Categories"),dbc.Tab(tab2_content, label="Point SPI Timeseries")]), style = {'float': 'right', 'height': '80%','width': '35%','padding-top': '10px','padding-right': '10px','padding-bottom': '10px','padding-left': '10px','text-align': 'center'})], style = {'height': '90%', 'width': '100%'})]),color="primary", type="grow")
```

## Callbacks

When user selects timescale using the `timescale_picker_element`, the values in `month_picker_element` should change. The decorator for this can be written as:

```python
@app.callback(Output('month-picker','options'),
             [Input('timescale-picker','value')])
def update_month_picker(timescale):
    if(timescale == '1'):
        
        return month_options
    
    elif(timescale == '3'):
        
        return [{'label': 'March-May', 'value': '3_4_5'},
                {'label': 'June-August', 'value': '6_7_8'},
                {'label': 'July-September', 'value': '7_8_9'},
                {'label': 'September-November', 'value': '9_10_11'},
                {'label': 'December-February', 'value': '12_1_2'}]
    
    elif(timescale == '4'):
        
        return [{'label': 'January-April', 'value': '1_2_3_4'},
                {'label': 'February-May', 'value': '2_3_4_5'},
                {'label': 'March-June', 'value': '3_4_5_6'},
                {'label': 'April-July', 'value': '4_5_6_7'},
                {'label': 'May-August', 'value': '5_6_7_8'},
                {'label': 'June-September', 'value': '6_7_8_9'},
                {'label': 'July-October', 'value': '7_8_9_10'},
                {'label': 'August-November', 'value': '8_9_10_11'},
                {'label': 'September-December', 'value': '9_10_11_12'},
                {'label': 'October-January', 'value': '10_11_12_1'},
                {'label': 'November-February', 'value': '11_12_1_2'},
                {'label': 'December-March', 'value': '12_1_2_3'}]
    
    elif(timescale == '6'):
        
        return [{'label': 'January-June', 'value': '1_6'},
                {'label': 'February-July', 'value': '2_7'},
                {'label': 'March-August', 'value': '3_8'},
                {'label': 'April-September', 'value': '4_9'},
                {'label': 'May-October', 'value': '5_10'},
                {'label': 'June-November', 'value': '6_11'},
                {'label': 'July-December', 'value': '7_12'},
                {'label': 'August-January', 'value': '8_1'},
                {'label': 'September-February', 'value': '9_2'},
                {'label': 'October-March', 'value': '10_3'},
                {'label': 'November-April', 'value': '11_4'},
                {'label': 'December-May', 'value': '12_5'}]
    
    elif(timescale == '9'):
        
        return [{'label': 'January-September', 'value': '1_9'},
                {'label': 'February-October', 'value': '2_10'},
                {'label': 'March-November', 'value': '3_11'},
                {'label': 'April-December', 'value': '4_12'},
                {'label': 'May-January', 'value': '5_1'},
                {'label': 'June-February', 'value': '6_2'},
                {'label': 'July-March', 'value': '7_3'},
                {'label': 'August-April', 'value': '8_4'},
                {'label': 'September-May', 'value': '9_5'},
                {'label': 'October-June', 'value': '10_6'},
                {'label': 'November-July', 'value': '11_7'},
                {'label': 'December-August', 'value': '12_8'}]
    
    elif(timescale == '12'):
        
        return [{'label': 'January-December', 'value': '1_12'}]

#Callback to update the default value in Month picker when a certain timescale is selected

@app.callback(Output('month-picker','value'),
             [Input('timescale-picker','value')])
def update_month_picker_value(timescale):
    if(timescale == '1'):
        return '1'
    
    elif(timescale == '3'):
        return '1_2_3'
    
    elif(timescale == '4'):
        return '1_2_3_4'
    
    elif(timescale == '6'):
        return '1_6'
    
    elif(timescale == '9'):
        return '1_9'
    
    elif(timescale == '12'):
        return '1_12'
    
```

When the `load_data_button_element` is clicked, the `map-plot` choropleth object should updated with a new CSV file. The `pie_figure` object should also be updated after the calculation of the values for each drought category.

```python
#Load new csv according to the timescale, month range and year
@app.callback(Output('map-plot', 'figure'), 
             [Input('submit-button', 'n_clicks')],
             [State('timescale-picker', 'value'), State('month-picker','value'), State('year-picker','value')])
def change_map_data(n_clicks, ts, month, year):
    df = pd.read_csv(csv_folder_prefix + "SPI_" + str(year) + "_" + str(month) + ".csv")
    trace = [go.Choroplethmapbox(geojson=india, locations=df.ID, z=df.SPI,
                                    colorscale="RdBu", zmin=-2, zmax=2, marker_opacity=0.7, marker_line_width = 0)]
    fig = go.Figure(data=trace)
    fig.update_layout(mapbox_style="carto-positron",
                      mapbox_zoom=3.5, mapbox_center = {"lat": 23, "lon": 80})
    fig.update_layout(height = 550, margin={"r":0,"t":0,"l":0,"b":0})
    
    return fig

#Load new PieChart according to the timescale, month range and year
@app.callback(Output('pie-chart', 'figure'), 
             [Input('submit-button', 'n_clicks')],
             [State('timescale-picker', 'value'), State('month-picker','value'), State('year-picker','value')])
def change_pie_chart_data(n_clicks, ts, month, year):
    df = pd.read_csv(csv_folder_prefix + "SPI_" + str(year) + "_" + str(month) + ".csv")
    
    SPI_values = df['SPI'].values

    spi_categories_for_pie_chart = ['Extreme Wet', 'Severe Wet', 'Moderate Wet', 'Mild Wet', 'Normal', 'Mild Dry', 'Moderate Dry', 'Severe Dry', 'Extreme Dry']
    count_extreme_wet = len(np.where(SPI_values >= 2)[0])
    count_severe_wet = len(np.where((SPI_values >= 1.5)&(SPI_values < 2))[0])
    count_mod_wet = len(np.where((SPI_values >= 1.0)&(SPI_values < 1.5))[0])
    count_mild_wet = len(np.where((SPI_values >= 0.5)&(SPI_values < 1))[0])
    count_normal = len(np.where((SPI_values >= -0.5)&(SPI_values < 0.5))[0])
    count_mild_dry = len(np.where((SPI_values >= -1.0)&(SPI_values < -0.5))[0])
    count_mod_dry = len(np.where((SPI_values >= -1.5)&(SPI_values < -1.0))[0])
    count_severe_dry = len(np.where((SPI_values >= -2.0)&(SPI_values < -1.5))[0])
    count_extreme_dry = len(np.where(SPI_values < -2.0)[0])
    values_for_pie_chart = [count_extreme_wet, count_severe_wet, count_mod_wet, count_mild_wet, count_normal,count_mild_dry,count_mod_dry, count_severe_dry, count_extreme_dry]

    pie_figure = go.Figure(data=[go.Pie(labels=spi_categories_for_pie_chart, values=values_for_pie_chart, hole=.3)],
                      layout = go.Layout(title = "Timescale: {} month, Month: {}, Year: {}".format(ts, month, year)))

    return pie_figure
    
```

Finally, to run the server,

```
if(__name__ == '__main__'):
    app.run_server()
```

### References

Mohapatra, Manaruchi & Nikam, Bhaskar & Kour, Gunjan & Garg, Vaibhav & Thakur, Praveen & Aggarwal, Shiv. (2019). Development of an Open-Source Tool for Handling Big Data Problem in Estimation of Standardized Precipitation Index: SPI-Utility. 
