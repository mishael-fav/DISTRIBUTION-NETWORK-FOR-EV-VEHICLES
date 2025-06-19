# Distribution Network Analysis – Capacity Planning for EV Vehicles 🚗⚡

This project helps PowerCharge Utilities address challenges from growing EV adoption. Leveraging Python and data analytics, it evaluates grid capacity, detects bottlenecks, and optimizes upgrades to ensure reliability, sustainability, and compliance.

## Objective

To perform a comprehensive capacity planning analysis for electric vehicle distribution by:

- Assess Network Capacity: Conduct a comprehensive analysis of the distribution network's current capacity to handle increased load from EV charging stations.
- Identify Bottlenecks: Identify potential bottlenecks and vulnerabilities within the distribution network that could hinder reliable electricity delivery to EV charging stations.
- Optimize Network Upgrades: Develop a data-driven strategy for network upgrades that maximizes efficiency, minimizes costs, and ensures grid reliability.

## Tools & Technologies Used

- Python (Pandas, NumPy, Seaborn, Matplotlib)
- Jupyter Notebook / Google Colab
-  GeoPandas: For Distribution Network Geographies 

---

## Project Scope

- **Exploratory Data Analysis:** Conduct EDA to gain insights into electricity consumption patterns and network performance. 
- **Capacity Assessment:** Utilize historical data to assess the current distribution network's capacity and identify areas with high demand growth. Network Optimization: Utilize optimization algorithms to identify cost-effective network upgrades that address capacity shortfalls.
- **Reporting and Recommendations:** Present findings, including capacity assessments and upgrade recommendations, to the executive team for decision-making. Exploratory Data Analysis: Explore the data to understand its characteristics and discover patterns. 
- **Data Transformation:** Prepare the data for analysis by transforming, encoding, or normalizing it. 
- **Data Analysis:** Analyze data to understand pattern in order to generate insights that will be visualized. 
- **Data Visualization:** Create visual representations to communicate insights effectively. Interpretation and Insight Generation: Extract meaningful insights and interpret the results.

---

## Process

1. **Data Loading**
    ```python
   #import the necessary python libraries
	import pandas as pd
	import numpy as np
	import matplotlib.pyplot as plt
	import seaborn as sns
	import geopandas as gpd
    
    #Load the dataset
	Distribution_Data = pd.read_csv('/content/drive/MyDrive/EV ANALYSIS/ev_distribution_dataset.csv')
	Geospatial_Data = pd.read_csv('/content/drive/MyDrive/EV ANALYSIS/geospatial_dataset.csv')
	Weather_Data = pd.read_csv('/content/drive/MyDrive/EV ANALYSIS/weather_dataset.csv')
    ```

2. **Data Exploration**
  - Univariate Analysis:
  	- Visualize the distribution of electric consumption
  	- Analyze the distribution of EV types, charging habit and consumption type.
    ![Distribution of EV types, charging habit and consumption type](images/Distribution_of_EV_types,_charging_habit_and_consumption_type.png)
  - Code Snippet:
	```python
    sns.set(style="whitegrid")
    sns.set_palette("pastel")
    
    #create a 2x2 subplot grid
    fig, axes = plt.subplots(2, 2, figsize=(15, 10))
    
    #plot the distribution of electricity consumption
    sns.histplot(data= Distribution_Data, x='Electricity_Consumption (kWh)', bins= 30, kde=True, ax=axes[0, 0])
    axes[0,0].set_title('Distribution of Electricity Consumption')
    axes[0,0].set_xlabel('Electricity Consumption (kWh)')
    axes[0,0].set_ylabel('Frequency')
    
    #plot the distribution of EV types
    sns.countplot(data= Distribution_Data, x='EV_Type', ax=axes[0, 1])
    axes[0,1].set_title('Distribution of EV Types')
    axes[0,1].set_xlabel('Count')
    axes[0,1].set_ylabel('EV Type')
    
    #plot the distribution of the Charging habit
    sns.countplot(data= Distribution_Data, x='Charging_Habit', ax=axes[1, 0])
    axes[1,0].set_title('Distribution of Charging Habit')
    axes[1,0].set_xlabel('Count')
    axes[1,0].set_ylabel('Charging habit')
    
    #plot the distribution of the Customer type
    sns.countplot(data= Distribution_Data, x='Customer_Type', ax=axes[1, 1])
    axes[1,1].set_title('Distribution of Customer Type')
    axes[1,1].set_xlabel('Count')
    axes[1,1].set_ylabel('Customer Type')
    
    plt.tight_layout()
    plt.show() ```

- Bivariate Analysis
  	- Use geospatial data to visualize the locations of substations and EV charging stations
  	- Analyze the Capacity of transmission lines
  ![EV Substations and charging station](images/EV%20Substations%20and%20charging%20station.png)
- Code Snippet:
   ```python
      #Convert the Dataframes into Geodataframes
      EV_gdf = gpd.GeoDataFrame(Distribution_Data, geometry=gpd.points_from_xy(Distribution_Data.EV_longitude, Distribution_Data.EV_latitude))
      substation_gdf = gpd.GeoDataFrame(Geospatial_Data, geometry=gpd.points_from_xy(Geospatial_Data.substation_longitude, Geospatial_Data.substation_latitude))
      
      #Load the world map data
      world = gpd.read_file("/content/drive/MyDrive/EV ANALYSIS/ne_110m_admin_0_countries.shp")
      
      
      #Zoom in to North America (area of focus)
      north_america = world[world.CONTINENT == 'North America']
      
      #plotting the map for north america
      fig, ax = plt.subplots(figsize=(15, 10))
      north_america.boundary.plot(ax=ax, linewidth=0.5, color='lightgrey')
      north_america.plot(ax=ax, color='lightblue', edgecolor='black')
      
      #plotting the substations on the map
      substation_gdf.plot(ax=ax, color='blue', marker='s', markersize=100, label='Substations')
      
      #plotting the EV charging stations
      EV_gdf.plot(ax=ax, color='red', marker='s', markersize=10, label='EV charging station', alpha=0.5)
      
      #set title
      plt.title('North America EV Substations with its designated EV Charging Stations')
      plt.xlabel('Longitude')
      plt.ylabel('Latitude')
      
      plt.legend()
      plt.tight_layout()
      plt.show()
   ```

3. **Network Capacity Assessment**
- To perform Network Capacity Assessment:
	  - Calculate the Total Electricity Consumption for each substation
	  - Compare the Total Electricity Consumption with the transmission line capacity
	```Python
	#group the ev distribution data by substation id and calculate the total electricity consumption for each substation
    total_consumption_per_substation = Distribution_Data.groupby('Substation_ID')['Electricity_Consumption (kWh)'].sum().reset_index()
    
    #merge the total consumption data with geospatial data
    network_capacity_data = pd.merge(Geospatial_Data, total_consumption_per_substation, on='Substation_ID')
    
    network_capacity_data.rename(columns={'Electricity_Consumption (kWh)': 'Total_Electricity_Consumption (kWh)'}, inplace=True)
    
    #Calculating the ratio of total consumption to transmission line Capacity
    #the conversion rate of 1MW = 1000 kWh
    network_capacity_data['Consumption_to_Capacity_Ratio'] = network_capacity_data['Total_Electricity_Consumption (kWh)'] / (network_capacity_data['Transmission_Line_Capacity (MW)'] *1000)
	```
4. **Identify Bottlenecks**
- By analyzing the map, we can identify the substations and the areas that are potential bottle necks in the distribution network. These are areas where the consumption are high

	```Python
	#filtering the c2c greater than 1
	bottleneck_substations = network_capacity_data[network_capacity_data['Consumption_to_Capacity_Ratio'] >= 0.9]
	```


## Key Insights:

✅ Electricity Consumption: The elctricity consumption is mostly centerd around 500 kWh, with certain instances of higher consumption. This indicates varied demand at different times and locations.

✅ EV Types and Charging Habits: Electric Scooters is tyh most common types of EV's daily, indicating a consistent daily load on the distribution network.

✅ Consumer Type: Most Customers are Commercial consumers.

✅ Geospatial Distribution: The Geographical distribution of EV substations; where EV charging stations are quite far from the substations.

✅ Network Capacity: Some substations have a high Consumption_to_Capacity_Ratio, indicating potential bottlenecks and overloads in the network. There is also no correlation with the number of EVs per substation and the Consumption to Capcity Ratio, this shows that Number of EVs is not a factor for overload.

✅ Weather Correlation: The correlation between weather conditions (temperature and precipitation) and electricity consumption is weak in the current dataset, suggesting that other factors might be more influential in affecting electricity consumption.


## Based on the analysis done and the business problems at hand, all these should be incorporated into the business to Optimizing Network Upgrades:

1. **Prioritize Substation Upgrades:** Prioritize upgrades at substations where the Consumption_to_Capacity_Ratio is high, indicating potential overloads. Upgrade the transmission lines because the EV Charging Stations are too far from their corresponding Substations.

2. **Geospatial Analysis for Upgrade Planning:** Use geospatial analysis to determine the optimal locations for new substations or upgrades to existing ones. Consider factors like the proximity to high load demand areas (areas with high consumption to capcity ratio) and geographical constraints.

3. **Demand Side Management:** Implement demand-side management strategies to balance the load on the grid. Encourage customers to charge their EVs during off-peak hours through
Incentives or dynamic pricing.

4. **Advanced Monitoring and Analytics:** Deploy advanced monitoring systems to continuously monitor the health and performance of the distribution network. Use analytics to predict potential issues and take preventive action.

5. **Cost-Benefit Analysis:** Conduct a comprehensive cost-benefit analysis for different upgrade options. Consider factors like the cost of upgrades, operational costs, potential revenue from increased capacity, and the impact on service reliability and customer satisfaction.

6. **Customer Engagement:** Engage with customers to understand their needs and expectations. Provide clear communication about network upgrades and how they will enhance service reliability and meet the growing demand for EV charging.

7. **Continuous Improvement:** Continuously monitor and assess the performance of the distribution network. Gather feedback from customers and other stakeholders, and use this feedback to make further improvements and optimizations.

