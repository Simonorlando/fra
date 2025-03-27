Taxi Traffic Analysis Project
This project focuses on analyzing taxi traffic within a city to identify patterns in stop locations, peak traffic times, and critical points in the road network. By employing data science techniques, including spatial and temporal analysis, clustering, and predictive modeling, the project aims to understand and forecast taxi behavior concerning stops. The ultimate goal is to optimize traffic flow and improve the management of key areas, providing valuable insights for local authorities and mobility services.

Table of Contents
Project Overview

Dataset Description

Data Processing

Exploratory Data Analysis

Clustering Analysis

Temporal Movement Analysis

Bottleneck Identification

Conclusion

Requirements

Usage

License

Acknowledgments

Project Overview
The project's objective is to analyze taxi traffic data to uncover patterns related to stop locations, traffic congestion times, and critical points within the city's road network. Through data science methodologies, the project applies spatial and temporal analysis, clustering, and predictive modeling to comprehend and anticipate taxi stop behaviors. This analysis aims to enhance traffic flow and improve the management of critical areas, offering actionable insights for local authorities and transportation services.

Dataset Description
The dataset utilized in this project comprises taxi trajectory data, including information such as:

Longitude and Latitude: Geographic coordinates of taxi positions.

Timestamp: Date and time of recorded positions.

Driver ID: Unique identifier for each taxi driver.

The data covers a period from May 17, 2008, to June 10, 2008, and includes records for 50 unique vehicles.

Data Processing
The data processing steps involve:

Filtering: Removing records with speeds exceeding 250 km/h to eliminate anomalies.

Compression: Applying spatial compression with a radius of 0.2 km to reduce redundancy in trajectory points.

After processing, the dataset is transformed into a GeoDataFrame with associated geometries for spatial analysis.

Exploratory Data Analysis
Key analyses performed include:

Stop Count per Tile: Calculating the number of stops within each geographic tile.

Average Stop Duration: Determining the average duration of stops in each tile.

These analyses help identify areas with high stop densities and longer stop durations, which are critical for traffic management and urban planning.

Clustering Analysis
The project employs clustering techniques to group geographic tiles based on:

Number of Stops: Total stops recorded in each tile.

Entry and Exit Frequencies: Frequency of movements into and out of each tile.

Using the K-Means algorithm, the optimal number of clusters is determined, and tiles are grouped accordingly. This clustering reveals areas with similar traffic patterns, aiding in targeted traffic optimization strategies.

Temporal Movement Analysis
The dataset is segmented into four time intervals:

Morning: 6 AM to 12 PM

Afternoon: 12 PM to 6 PM

Evening: 6 PM to 12 AM

Night: 12 AM to 6 AM

For each interval, movement graphs are constructed to analyze traffic patterns, and the top 5 most frequent routes are identified. This temporal analysis provides insights into how traffic behaviors change throughout the day, informing time-specific traffic management policies.

Bottleneck Identification
By calculating the betweenness centrality within the movement graph, the project identifies critical tiles that serve as bottlenecks in the traffic network. The top 4 tiles with the highest centrality are highlighted, along with the 20 most frequent routes passing through them. Visualizations are created to map these bottlenecks and frequent routes, emphasizing areas that may require intervention to alleviate congestion.

Conclusion
The analysis offers a comprehensive view of taxi traffic patterns, highlighting high-traffic areas and potential congestion points. By applying clustering and temporal analyses, the project identifies critical zones and peak times for traffic, providing valuable insights for optimizing traffic flow and enhancing urban mobility management.

Requirements
The project requires the following Python libraries:

geopandas

pandas

scikit-mobility

folium

networkx

statsmodels

matplotlib

shapely

geopy

numpy

scikit-learn

collections

pmdarima

These dependencies are listed in the requirements.txt file.

Usage
To replicate the analysis:

Install Dependencies: Use the package manager pip to install the required libraries.

bash
Copy
Edit
pip install -r requirements.txt
Data Preparation: Load the taxi trajectory data into a DataFrame, ensuring it includes columns for longitude, latitude, timestamp, and driver ID.

Run Analysis: Execute the provided Python scripts to perform data processing, exploratory analysis, clustering, temporal movement analysis, and bottleneck identification.

Visualizations: Utilize the generated visualizations to interpret the results and derive insights for traffic optimization.

License
This project is licensed under the MIT License. See the LICENSE file for details.

