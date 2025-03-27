# Taxi Mobility Analysis
## Project Overview
This project focuses on analyzing taxi traffic within a city to identify patterns in stops across various areas, peak traffic times, and critical points in the road network. By employing data science techniques, including spatial and temporal analysis, clustering, and predictive modeling, the project aims to understand and forecast taxi behavior related to stops. The ultimate goal is to optimize traffic flow and improve the management of key areas, providing valuable insights for local authorities and mobility services.

## Data Processing
The analysis begins with loading and preprocessing the taxi trajectory data. The dataset includes records of taxi movements, detailing timestamps, driver IDs, and GPS coordinates. Initial steps involve filtering out anomalies, such as records with unrealistic speeds exceeding 250 km/h, and applying spatial compression to reduce redundancy in trajectory points within a 0.2 km radius.

## Spatial Analysis
To examine the spatial distribution of taxi stops, the city is divided into tessellated zones based on zip codes. Each taxi stop is assigned to a corresponding tile, allowing for the calculation of the number of stops per tile and the average stop duration. This approach helps identify areas with high stop densities or prolonged dwell times, which are crucial for traffic management and urban planning.

## Movement Analysis Between Tiles
A movement graph is constructed to represent taxi transitions between tiles. Nodes in the graph correspond to tiles, and edges represent movements between them, weighted by frequency. Calculating the betweenness centrality of each node identifies critical tiles that serve as key connectors in the taxi movement network. Visualizing this graph highlights significant movement patterns and potential bottlenecks within the city's transportation system.

## Clustering Analysis
Clustering techniques, specifically K-Means, are applied to group tiles based on the number of stops and movement frequencies. The optimal number of clusters is determined using the elbow method and silhouette scores. Visualizing these clusters on a map reveals areas with similar mobility characteristics, aiding in targeted traffic management strategies.

## Temporal Movement Analysis
The project segments taxi movement data into four time intervals: morning (6 AM–12 PM), afternoon (12 PM–6 PM), evening (6 PM–12 AM), and night (12 AM–6 AM). For each interval, subgraphs are created to analyze movement patterns, and the top five most frequent routes are identified. This temporal analysis provides insights into how taxi traffic varies throughout the day, informing time-specific traffic management interventions.

## Bottleneck Identification
Combining structural (centrality) and behavioral (frequency) dimensions, the analysis identifies critical points in the system. The four tiles with the highest betweenness centrality are selected, and the 20 most frequent paths passing through them are analyzed. Visualizing these on a map highlights the core interconnection network—the critical backbone that would bear the brunt of any congestion or disruption. This information is vital for prioritizing areas in need of infrastructure improvements or traffic regulation adjustments.

## Conclusion
The comprehensive analysis offers a detailed understanding of taxi traffic patterns within the city, pinpointing high-traffic areas and primary congestion points. By integrating spatial and temporal analyses with clustering and predictive modeling, the project provides actionable insights to optimize traffic flow and enhance the management of critical areas. These findings are valuable for local authorities and mobility services aiming to improve urban transportation systems.
