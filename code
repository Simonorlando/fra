import geopandas as gpd
import pandas as pd
from skmob import TrajDataFrame
from skmob.preprocessing import filtering, compression
import folium
from folium.plugins import HeatMap
import networkx as nx
from statsmodels.tsa.arima.model import ARIMA
import matplotlib.pyplot as plt
from shapely.geometry import Point
from geopy.distance import geodesic
from folium import plugins
import numpy as np
from sklearn.cluster import KMeans
from collections import defaultdict
from sklearn.metrics import silhouette_score
from statsmodels.tsa.statespace.sarimax import SARIMAX
from pmdarima import auto_arima
from statsmodels.tsa.holtwinters import ExponentialSmoothing

url = "https://raw.githubusercontent.com/scikit-mobility/tutorials/master/mda_masterbd2020/data/bay_area_zip_codes.geojson"
tessellation = gpd.read_file(url)
tessellation = tessellation.explode().groupby("zip", as_index=False).first()
tessellation.rename(columns={'zip': 'tile_ID'}, inplace=True)
tessellation.set_crs("EPSG:4326", inplace=True)

mydateparser = lambda x: pd.to_datetime(x, unit='s')
file_path = '/Users/simoneorlando/Desktop/final_exercise_tsmd (3)/cabs.csv.gz'
tdf = TrajDataFrame(
    pd.read_csv(file_path, compression='gzip', parse_dates=['timestamp'], date_parser=mydateparser),
    longitude='lon', datetime='timestamp', user_id='driver'
).sort_values(by=['uid', 'datetime'])

tdf = tdf[tdf['uid'].isin(tdf['uid'].unique()[:50])]

f_tdf = filtering.filter(tdf, max_speed_kmh=250.0)
fc_tdf = compression.compress(f_tdf, spatial_radius_km=0.2)

fc_tdf['geometry'] = fc_tdf.apply(lambda row: Point(row['lng'], row['lat']), axis=1)
fc_tdf_gdf = gpd.GeoDataFrame(fc_tdf, geometry='geometry')
fc_tdf_gdf.set_crs("EPSG:4326", inplace=True)
fc_tdf_gdf = gpd.sjoin(fc_tdf_gdf, tessellation, how="left", op="within")

conteggio_fermate = fc_tdf_gdf.groupby("tile_ID").size()
tessellation["conteggio_fermate"] = tessellation["tile_ID"].map(conteggio_fermate)
fc_tdf_gdf['durata_fermata'] = fc_tdf_gdf.groupby('uid')['datetime'].diff().dt.total_seconds() / 60
durata_media_fermata = fc_tdf_gdf.groupby('tile_ID')['durata_fermata'].mean()
tessellation["durata_media_fermata"] = tessellation["tile_ID"].map(durata_media_fermata)

def costruisci_grafo_movimenti(fc_tdf_gdf):
    G = nx.Graph()
    for uid in fc_tdf_gdf['uid'].unique():
        spostamenti = fc_tdf_gdf[fc_tdf_gdf['uid'] == uid][['tile_ID', 'datetime']].dropna()
        for i in range(len(spostamenti) - 1):
            u, v = spostamenti.iloc[i]['tile_ID'], spostamenti.iloc[i + 1]['tile_ID']
            if u != v:
                G.add_edge(u, v, weight=G.get_edge_data(u, v, {}).get('weight', 0) + 1)
    return G, nx.betweenness_centrality(G, weight="weight")

G, centralita = costruisci_grafo_movimenti(fc_tdf_gdf)

componenti_connesse = list(nx.connected_components(G))
percorsi_frequenti = sorted(G.edges(data=True), key=lambda x: x[2]['weight'], reverse=True)

mappa = folium.Map(location=[37.7749, -122.4194], zoom_start=12)
for _, row in tessellation.iterrows():
    freq = row["conteggio_fermate"]
    color = "green" if freq == 0 else "orange" if freq < 50 else "red"
    folium.GeoJson(
        row['geometry'],
        style_function=lambda feature, color=color: {
            'fillColor': color, 'color': "black", 'weight': 1, 'fillOpacity': 0.5
        },
        tooltip=f"Tessera ID: {row['tile_ID']} - Fermate: {int(freq)}"
    ).add_to(mappa)

top_percorsi = percorsi_frequenti[:10]
for u, v, peso in top_percorsi:
    c1 = tessellation.loc[tessellation['tile_ID'] == u, 'geometry'].centroid.iloc[0].coords[0]
    c2 = tessellation.loc[tessellation['tile_ID'] == v, 'geometry'].centroid.iloc[0].coords[0]
    folium.PolyLine(
        [c1, c2], color="blue", weight=2 + peso['weight'] * 0.1,
        opacity=0.6, tooltip=f"Da {u} a {v} - Peso: {peso['weight']}"
    ).add_to(mappa)

frequenza_entrata = defaultdict(int)
frequenza_uscita = defaultdict(int)
for u, v, data in G.edges(data=True):
    frequenza_uscita[u] += data['weight']
    frequenza_entrata[v] += data['weight']

tessellation["frequenza_entrata"] = tessellation["tile_ID"].map(frequenza_entrata).fillna(0)
tessellation["frequenza_uscita"] = tessellation["tile_ID"].map(frequenza_uscita).fillna(0)

X = tessellation[["conteggio_fermate", "frequenza_entrata", "frequenza_uscita"]].values
kmeans = KMeans(n_clusters=3, random_state=42)
labels = kmeans.fit_predict(X)
tessellation["KMeans"] = labels

mappa_clusters = folium.Map(location=[37.7749, -122.4194], zoom_start=12)
cluster_colors = {0: 'green', 1: 'orange', 2: 'red'}
for _, row in tessellation.iterrows():
    cluster_id = row["KMeans"]
    color = cluster_colors.get(cluster_id, 'gray')
    folium.GeoJson(
        row['geometry'],
        style_function=lambda feature, color=color: {
            'fillColor': color, 'color': "black", 'weight': 1, 'fillOpacity': 0.5
        },
        tooltip=f"Tile ID: {row['tile_ID']} - Cluster: {cluster_id} - Stops: {int(row['conteggio_fermate'])}"
    ).add_to(mappa_clusters)

fasce_orarie = {
    "mattina": (6, 12),
    "pomeriggio": (12, 18),
    "sera": (18, 24),
    "notte": (0, 6)
}
sotto_grafi_temporali = {}
for fascia, (start_hour, end_hour) in fasce_orarie.items():
    tdf_fascia = fc_tdf_gdf[(fc_tdf_gdf["datetime"].dt.hour >= start_hour) & (fc_tdf_gdf["datetime"].dt.hour < end_hour)]
    G_fascia = nx.Graph()
    for uid in tdf_fascia['uid'].unique():
        spostamenti = tdf_fascia[tdf_fascia['uid'] == uid][['tile_ID', 'datetime']].dropna()
        for i in range(len(spostamenti) - 1):
            u, v = spostamenti.iloc[i]['tile_ID'], spostamenti.iloc[i + 1]['tile_ID']
            if u != v:
                G_fascia.add_edge(u, v, weight=G_fascia.get_edge_data(u, v, {}).get('weight', 0) + 1)
    sotto_grafi_temporali[fascia] = G_fascia

for fascia, G_fascia in sotto_grafi_temporali.items():
    percorsi_frequenti = sorted(G_fascia.edges(data=True), key=lambda x: x[2]['weight'], reverse=True)[:5]
    print(f"\nTop 5 {fascia}:")
    for u, v, data in percorsi_frequenti:
        print(f"{u} → {v} - {data['weight']}")

centralita_betweenness = nx.betweenness_centrality(G, weight="weight")
top_betweenness_nodes = sorted(centralita_betweenness.items(), key=lambda x: x[1], reverse=True)[:4]

mappa_betweenness = folium.Map(location=[37.7749, -122.4194], zoom_start=12)
for _, row in tessellation.iterrows():
    folium.GeoJson(
        row['geometry'],
        style_function=lambda feature: {
            'fillColor': 'lightgray', 'color': "black", 'weight': 1, 'fillOpacity': 0.3
        }
    ).add_to(mappa_betweenness)

for nodo, centralita in top_betweenness_nodes:
    tess_geom = tessellation.loc[tessellation['tile_ID'] == nodo, 'geometry'].iloc[0]
    folium.GeoJson(
        tess_geom,
        style_function=lambda feature: {
            'fillColor': 'red', 'color': "black", 'weight': 2, 'fillOpacity': 0.7
        },
        tooltip=f"Tile {nodo} - Betweenness: {centralita:.4f}"
    ).add_to(mappa_betweenness)
