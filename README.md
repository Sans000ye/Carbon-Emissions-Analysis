# 1. Introduction
## What is the project about?
This report aims to analyze carbon emissions to examine the carbon footprint across various industries. I aim to identify sectors with the highest levels of emissions by analyzing them across countries and years, as well as to uncover trends.

Carbon emissions play a crucial role in the environment, accounting for over 75% of global emissions and posing a significant environmental challenge. These emissions contribute to the accumulation of greenhouse gases in the atmosphere, leading to climate change, planetary warming, and involvement in various environmental disasters.
## Why do I Choose this project
Through this analysis, I hope to gain an understanding of the environmental impact of different industries and contribute to making informed decisions in sustainable development.
# 2. Data overview
## Where my Data Comes From
My dataset is compiled from publicly available data from nature.com and encompasses the product carbon footprints (PCF) for various companies. PCFs represent the greenhouse gas emissions associated with specific products, quantified in CO2 (carbon dioxide equivalent).

## Data Structure
![image](https://github.com/Sans000ye/Carbon-Emissions-Analysis/blob/main/Database%20diagram.png?raw=true)
## Table Preview
### **Product_emissions**
**id:** Identifier for each product emission record.
**company_id**: Identifier for the company associated with the product.
**country_id**: Identifier for the country where the product is being produced.
**industry_group_id**: Identifier for the industry group to which the product belongs.
**year**: The year in which the emissions data was recorded.
**product_name**: The name of the product associated with the emissions data.
**weight_kg**: The weight of the product in kilograms.
**carbon_footprint_pcf**: The carbon footprint of the product, measured in CO2 equivalent.
**upstream_percent_total_pcf**: The percentage of the total carbon footprint attributed to upstream activities.
**operations_percent_total_pcf**: The percentage of the total carbon footprint attributed to operations.
**downstream_percent_total_pcf**: The percentage of the total carbon footprint attributed to downstream activities.

### **Industry_groups**
**id**: Unique identifier for each industry group.
**industry_group**: The name of the industry group, categorizing businesses within similar sectors based on their products or services offered

### **Companies**
**id**: Unique identifier for each company.
**company_name**: The name of the company, identifying the specific organization within the dataset.

### **countries**
**id**: Unique identifier for each country.
**country_name**: The name of the country.

# 3.Pre-processing
