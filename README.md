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
in the raw data, there are a lot of "same id" in the dataset
e.g:
```
SELECT * FROM product_emissions pe LIMIT 5
```
### result:
|id|company_id|country_id|industry_group_id|year|product_name|weight_kg|carbon_footprint_pcf|upstream_percent_total_pcf|operations_percent_total_pcf|downstream_percent_total_pcf|rn|
|--|----------|----------|-----------------|----|------------|---------|--------------------|--------------------------|----------------------------|----------------------------|--|
|10056-1-2014|82|28|2|2014|Frosted Flakes(R) Cereal|0.7485|2|57.50|30.00|12.50|1|
|10056-1-2015|82|28|15|2015|"Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)"|0.7485|2|57.50|30.00|12.50|1|
|10222-1-2013|83|28|8|2013|Office Chair|20.68|73|80.63|17.36|2.01|1|
|10261-1-2017|14|16|25|2017|Multifunction Printers|110.0|1488|30.65|5.51|63.84|1|
|10261-2-2017|14|16|25|2017|Multifunction Printers|110.0|1818|25.08|4.51|70.41|1|

Solution? use a "cleaner"
if i implement this in every query i got:
```
WITH handle AS (
SELECT * FROM(
  SELECT *, 
  			ROW_NUMBER() OVER (PARTITION BY id ) as rn
  FROM 
  	product_emissions pe 
)AS raw
WHERE
	raw.rn = 1
)
```
result:
|id|company_id|country_id|industry_group_id|year|product_name|weight_kg|carbon_footprint_pcf|upstream_percent_total_pcf|operations_percent_total_pcf|downstream_percent_total_pcf|rn|
|--|----------|----------|-----------------|----|------------|---------|--------------------|--------------------------|----------------------------|----------------------------|--|
|10056-1-2014|82|28|2|2014|Frosted Flakes(R) Cereal|0.7485|2|57.50|30.00|12.50|1|
|10056-1-2015|82|28|15|2015|"Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)"|0.7485|2|57.50|30.00|12.50|1|
|10222-1-2013|83|28|8|2013|Office Chair|20.68|73|80.63|17.36|2.01|1|
|10261-1-2017|14|16|25|2017|Multifunction Printers|110.0|1488|30.65|5.51|63.84|1|
|10261-2-2017|14|16|25|2017|Multifunction Printers|110.0|1818|25.08|4.51|70.41|1|

# 4.Analyse
## Which products contribute the most to carbon emissions?
### SQL query used:
```
WITH handle AS (
SELECT * FROM(
  SELECT *, 
  	 ROW_NUMBER() OVER (PARTITION BY id ) as rn
  FROM 
  	product_emissions pe 
)AS raw
WHERE
	raw.rn = 1
)
SELECT 
    product_name,
    weight_kg,
    carbon_footprint_pcf
FROM handle
WHERE carbon_footprint_pcf IS NOT NULL 
ORDER BY carbon_footprint_pcf DESC
LIMIT 1;
```

result: 
|product_name|weight_kg|carbon_footprint_pcf| 
|------------|---------|--------------------| 
|Wind Turbine G128 5 Megawats|600000.0|3718044|

The Wind Turbine G128 (5 MW) shows an extremely high carbon footprint (3,718,044 kg CO₂e) mainly due to its massive weight and materials, indicating large-scale production and transport impacts.

## What are the industry groups of these products?
### SQL query used:
```
WITH handle AS (
  SELECT *
  FROM (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY id) AS rn
    FROM product_emissions pe
  ) AS raw
  WHERE raw.rn = 1
)
SELECT 
    GROUP_CONCAT(h.product_name ORDER BY h.product_name SEPARATOR ', ') AS product_name,
    ig.industry_group
FROM handle h
JOIN industry_groups ig 
  ON h.industry_group_id = ig.id
GROUP BY ig.industry_group;
```

result: 
|product_name|industry_group|
|------------|--------------|
|"Sayl Work Chair, Suspension Back", Aeron Chair, Caper Stacking Chair, Celle Chair, Embody Chair, Mirra 2 Chair, Setu Multipurpose Chair with Lyris 2, Vaccum Cleaners|"Consumer Durables, Household and Personal Products"|
|"1 dl cup of spray dried soluble Nescafé coffee ready to be drunk, at the consumer’s home", "Carling brewed in the UK, 24 440ml cans in 4 packs", "Carling brewed in the UK, 24 440ml cans in 4 packs", "Carling brewed in the UK, 24 568ml cans in 4 packs", "Carling brewed in the UK, 24 568ml cans in 4 packs", "Coors Light brewed in the UK, 24 500ml cans in 4 packs", "EcoShape  bottle of water (0.5 L): Serving of 500 ml of hydration to the consumer. The study examines the extraction and processing of all upstream raw materials to produce the products and packaging components; the manufacturing process; distribution logistics; product use and packaging disposal. The work has been conducted using the Ecoinvent database and SimaPro software.  Climate change impacts are measured in kg CO2 equivalents, following the IPCC’s most recent methodology).  Scenario evaluations have been used to examine the importance of several variables in the product comparison. These include a range of possible consumer behaviours regarding the use of the products, such as conditions of refrigeration, washing of glasses, and disposal routes for packaging, among others.", "Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b", "Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b", "Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b", 5 oz can lightmeat tuna, 5 oz can whitemeat tuna, 500ml Coors Light brewed in the UK (steal can) CO2e expressed in CO2e/ hl, AGF Blendy Stick Cafe au Lait, Aji-ngon Pork, Ajinomoto KK Consomme(Granules), Ajinomoto KK Shirogayu 250g, Aspartame, Average SKU, Coca-Cola (all packaging and sizes), Coca-Cola (all packaging and sizes), Coca-Cola 2 litre plastic bottle, Coca-Cola 300ml can, Coca-Cola 300ml glass bottle, Coca-Cola 500ml plastic bottle, Coke Zero 300ml can, Coke Zero 330 ml glass bottle, Coke Zero 500ml PET, Coke Zero- 2 litre plastic bottle, Cook Do(R) Hoikoro, Cook Do(R) kyo-no Ozara Butabara Daikon, Crown caps, Di-sodium 5'-Inosinate, Diced tomato, Diet Coke - 2 Litre Plastic Bottle, Diet Coke 300ml can, Diet Coke 330ml glass bottle, Diet Coke 500 ml plastic bottle, Drink yoghourt, EcoShape  bottle of water (0.5 L), EcoShape bottle of water (0.5 L), Extra Neutral Potable Ethanol 96.4%, Extra Neutral Potable Ethanol 96.4%, Frosted Flakes(R) Cereal, GMP (5'-Guanylate), HON-DASHI(R), I&G (5'-Ribonucleotide), IMP (5'-Inosinate), Ingeo, Ingeo--biopolymer (polylactid) made from corn starch, Knorr(R) Cup Soup Tsubu Tappuri Corn Cream, L-Arginine, L-Arginine, L-Glutamine, L-Isoleucine, L-leucine, L-Lysine Monohydrochloride(For Feed), L-Valine, Lemon and Basil Fried Chicken, Malt extract, Malt flour in sack, Malted Barley, Masako Ayam, Mentsuyu, metal crown caps, Monosodium L-Glutamate, MSG (Monosodium Glutamate), Nabe Cube(R) Toridashi Umashio, Natural Flavors, Nescafé soluble coffee, Nescafé soluble coffee, Nicotine, Oasis Summer Fruits  - 375ml glass, Oasis Summer Fruits - 500ml plastic bottle, Peppermint tea infusion, Peppermint tea infusion, Plain Water (SKU: Bonafont 1L film bottle), Quaker Oats 1Kg box sold in the United Kingdom, Quaker Oats 1Kg box sold in the United Kingdom, Quaker Oats So Simple Original and Golden Syrup products, Quaker Oats So Simple Original and Golden Syrup products, Rosdee Pork, See attached sheet, Set yoghourt, Stevia Sweetener, Stevia Sweeteners, Stevia Sweeteners, SUGAR, Sugar, Sugar, This is the average footprint of the 31 products we provided Walmart in 2012. The unit is in kg CO2e/tonne of product., Tomato fibre, Tomato paste, Tomato powder, Unmalted barley at harvest, Walkers Crisps 32.5 g sold in the United Kingdom, Walkers Crisps 32.5 g sold in the United Kingdom, Walkers Crisps 50g bag sold in the United Kingdom, Walkers Crisps 50g bag sold in the United Kingdom, Walmart Brand Products, White crystalline sugar|"Food, Beverage & Tobacco"|
|Avanta Prima, Carta Elega, Carta Integra, Carta Solida, Folding boxboard, Incada, Invercote, Simcote, Tako CX Lite, Tako CX Lite OBA, Tako CX Lite S, Tako CX Lite S2, Tako CX White S|"Forest and Paper Products - Forestry, Timber, Pulp and Paper, Rubber"|
|Cold Rolled Steel (CR), Galvanized Steel (Galv), Hot Rolled Steel (HR)|"Mining - Iron, Aluminum, Other Metals"|
|"ACQUITY® UPLC, ACQUITY® I-Class, ACQUITY® H-Class", Alliance (HPLC), Alliance HPLC (High Peformance Liquid Chromatography)  The Alliance is an HPLC that is unique in that it has a single set of electronic boards that control the functions for both the solvent delivery system and the autosampler in the liquid chromatograph.|"Pharmaceuticals, Biotechnology & Life Sciences"|
|501® Original Jeans – Dark Stonewash, 501® Original Jeans – Light Stonewash, 501® Original Jeans – Medium Stonewash, 501® Original Jeans – Rinse Run, Interface Americas Carpet, Interface Asia Carpet, Interface Australia Carpet, Interface China Carpet, Interface EMEA Carpet, Regular Straight 505® Jeans – House Cat, Regular Straight 505® Jeans – Range, Regular Straight 505® Jeans – Range (Water<Less™), Regular Straight 505® Jeans – Steel (Water<Less™), Slim Straight 514™ Jeans – Indigo Wash, Slim Straight 514™ Jeans – Rigid Tank, Slim Straight 514™ Jeans – Tumbled Rigid|"Textiles, Apparel, Footwear and Luxury Goods"|
|1 fuel-efficient tire for passenger cars (乗用車用低燃費タイヤ), 1 fuel-efficient truck and buss tireトラック・バス用低燃費タイヤ１本当たり, Audi A3 2.0 TDI, Audi A6, Audi A6, Audi A6, Average GM Vehicle, Average of all GM vehicles produced and used in the 10 year life-cycle., Average of all GM vehicles produced and used in the 10 year life-cycle., Average of all GM vehicles produced and used in the 10 year life-cycle., Beedwire 0.96, Beedwire 1.27, Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit., Mean value for a vehicle within the Volkswagen Group according to CDP Report 2013 (global scope), Mean value for a vehicle within the Volkswagen Group according to CDP Report 2014 (global scope), Mercedes-Benz A-Class, Mercedes-Benz A-Class (A 180 BlueEFFICIENCY), Mercedes-Benz B-Class, Mercedes-Benz B-Class (B 180 BlueEFFICIENCY), Mercedes-Benz B-Class electric drive, Mercedes-Benz B-Class Electric Drive (B 250 e), Mercedes-Benz C-Class, Mercedes-Benz C-Class (C 180), Mercedes-Benz C-Class (C 250), Mercedes-Benz C-Class e, Mercedes-Benz C-Class Plug-In Hybrid (C 350 e), Mercedes-Benz CLA (CLA 180), Mercedes-Benz CLA-Class, Mercedes-Benz CLS (350 BlueEFFICIENCY), Mercedes-Benz CLS-Class, Mercedes-Benz E-Class (E 200), Mercedes-Benz E-Class (E 220 d (Estate)), Mercedes-Benz E-Class (E 220 d), Mercedes-Benz E-Class BlueTEC Hybrid, Mercedes-Benz E-Class Plug-In Hybrid (E 350 e), Mercedes-Benz E-Class Saloon, Mercedes-Benz GLA (GLA 200), Mercedes-Benz GLA-Class, Mercedes-Benz GLC (220 d 4MATIC), Mercedes-Benz GLC Plug-In Hybrid (GLC 350 e 4MATIC), Mercedes-Benz GLE (GLE 500 4MATIC), Mercedes-Benz GLE Plug-In Hybrid - GLE 500 e 4MATIC, Mercedes-Benz GLK-Class, Mercedes-Benz M-Class - Passenger Car, Mercedes-Benz S-Class, Mercedes-Benz S-Class (S 500), Mercedes-Benz S-Class Hybrid (S 300 BlueTEC HYBRID), Mercedes-Benz S-Class Hybrid (S 400 h), Mercedes-Benz S-Class Plug-In Hybrid (S 500 e), Mercedes-Benz SL (SL 350), Mercedes-Benz SL-Class, Mercedes-Benz SLC (SLK 200 BlueEFFICIENCY), Passat B7 2.0 TDI BlueMotion Technology, Tire, Tire including low fuel consumption tire; e.g. ECOPIA series, Volkswagen Caddy, Volkswagen Caddy, Volkswagen e-up, Volkswagen Golf, Volkswagen Golf, Volkswagen Passat, Volkswagen Passat, Volkswagen Polo, Volkswagen Polo, Volkswagen up!, Volkswagen up!, VW Caddy, VW Golf, VW Golf VII 1.6 TDI BlueMotion Technology, VW Passat, VW Polo, VW Polo V 1.6 TDI BlueMotion Technology, VW up!|Automobiles & Components|
|"Agricultural tires, BlueTire Yechnology", "Gas Product - Small 555 series cast iron softseal valve, PN16. This series 555 softseal valve is a double-faced, resilient seat wedge gate valve. It is designed primarily for the isolation of natural gas and towns gas. Kitemark approved for GIS/V7 part 1.", "Series 29 Hydrant - Squat Hydrant suitable for use with water and neutral liquids, to a maximum temperature of 70 degrees centigrade. Complies with requirements of BS750:2006 and BSEN1074-2:2004 and EN14339:2005, underground hydrants. Also to BS EN 1074-6 for potable (drinking) water. This product has kitemark and WRAS approval. The 2938832112 replaces the 2928832112.", "The series 21/75 is a resilient seat, wedge gate valve for isolation purposes, suitable for use with water and neutral liquids (sewage), to a maximum temperature of +70 degrees centegrade. The series 21/75 is a new alternative to a series 21/50 in some applications. Therefore where a 21/75 is sold we can calculate the emissions saved in contrast to the sale of a 21/50.", 33-725 Tape measure family, Acti9 – Residual current circuit breaker – iID K. The main function of the iID K product range is to ensure protection of people against electric shocks. PEP of this product is available here: http://planet.schneider-electric.com/C12577A200268439/all/14AA652B755F340C852577F3006F06D0/$File/envpep100201en.pdf, ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER, ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER, aluminium wheel, Cable, Cable of LMR-195-LLPL, Can/Bottle vending machine  -30Selections  -Cooling / warming  - R744 Heat Pump Refrigeration System  - LED Lighting, Cast iron pipes and fittings, Commercial Air Conditioner, Cordless Power Drill , D21710 Type 5 Corded Power Drill, E - Series Solar Panels (SPR-327NE-WHT-D AR modules), Electric Motor, Electric Motor, Light commercial Air Conditioner, Office Chair, Polyethylene pipe, Residential Air Conditioner, Residential Air Conditioner, Residential Air Conditioner, Residential Air Conditioner, Series 202/31 Repair Clamp, Series 21/50 Gate Valve, Series 29/388 Squat Hydrant, Series 555/300 Gate Valve, T300, Vending Machine, Zinc Oxide|Capital Goods|
|"Soda Ash (Tata Chemicals Mithapur, India)", ABS (Acrylonitrile Butadiene Styrene), Aggregated Global Sales for Customer CNH Industrial NV, Aggregated Global Sales for Johnson & Johnson, Aggregated Global Sales for L'Oreal, Aggregated Global Sales for Unilever, Automotive Batteries, Average of all products sold to Braskem, Beverage can inside spray lacquer aluminium, Dense Sodium Carbonate, Fungicide, Herbicide, Insecticide, Latex, Light Sodium Carbonate, Mobile Batteries, Overprint varnishes, Plastics, Polyethylene, Polypropylene, Polyvinyl Chloride, Rubber, Seed treatment, Soda Ash, Soda Ash (Tata Chemicals Europe), Soda Ash (Tata Chemicals Magadi), Soda Ash (Tata Chemicals North America), Sodium Bicarbonate, Sodium sulphate|Chemicals|
|"Kimberley-Clark toilet tissue, 10,000 sheets", "Share It is a modular storage system. It offers personal storage, team storage, meeting point solutions and lockers.  Share It can be also used as space dividers, structuring workspaces. - It is modular and offers endless planning possibilities. - The range enhances collaboration providing communication platforms. - It helps people concentrate thanks to acoustics absorbing surfaces. - It offers wide range of finishes for different workplace ambiances.", "The Activa desks are highly ergonomic and offer different height adjustable versions in a single design. Its modular construction allows very easy assembly and reconfiguration.  The model chosen for analysis is the most frequently ordered one (reference W6412700) from the Activa Telescopic range. Standard features on this model include: - Top dimensions: 1600 mm x 800 mm - Melamine top, Snow WY - Steel leg and frame, Pearl Snow ZW - Telescopic height adjustment from 620 mm to 900 mm - Steel cable tray.", 32 Seconds (European supply chain), Activa (European supply chain), Activa desk, Amia (European supply chain), B-Free big cube (European supply chain), B-Free Desk (European supply chain), B-Free small cube (European supply chain), B-Free Table (European supply chain), Dell Laptop, Eastside (European supply chain), Flip Top Twin (European supply chain), Float Table Top & Base, FrameOne Bench (European supply chain), Fusion (European supply chain), Gesture (European supply chain), Gesture (North American supply chain), GL film packages, Implicit (European supply chain), Kalidro (European supply chain), Leap (European supply chain), Let's B (European supply chain), Let’s B task chair, Let’s B task chair, Movida (European supply chain), MR-PET film package, New Think (European supply chain), Ology (European supply chain), Partito (European supply chain), Please (European supply chain), Qivi (European supply chain), Reply (European supply chain), Share It modular storage system, Smart Chair, Tenaro (European supply chain), The model chosen for analysis is the most popular model Think work chair.  It is a highly adjustable ergonomic chair equipped as follows: 1. Your PowerTM weight activated mechanism 2. Your ProfileTM seat and back flexors  3. Your PreferenceTM control  4. Adjustable seat depth 5. Adjustable seat height 6. Adjustable lumbar support 7. Adjustable armrests 8. Plastic base, Think work chair, TNT* (European supply chain), Universal Storage EU (European supply chain), Westside (European supply chain)|Commercial & Professional Services|
|"Sayl Work Chair, Suspension Back", 501® Original Jeans – Dark Stonewash, 501® Original Jeans – Dark Stonewash, 501® Original Jeans – Light Stonewash, 501® Original Jeans – Medium Stonewash, 501® Original Jeans – Rinse Run, Aeron Chair, Aeron Work Stool, Air Purifier, Canvas Wood Credenza 2016, Canvas Wood Lateral, Canvas Wood Pedestal 2016, Canvas Wood Tower, Caper Stacking Chair, Celle Chair, Droid Razr, Ecoworx Carpet Tile with Shaw's Eco Solution Q Nylon 6 face fiber, Embody Chair, Interface Americas Carpet, Interface Asia carpet, Interface Australia Carpet, Interface Australia carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in Australia), Interface Brazil Carpet, Interface Canada Carpet, Interface China Carpet, Interface EMEAI carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in EMEAI), Interface Europe Carpet, Interface Europe carpet, Interface Global LVT, Interface Thailand Carpet, Interface Thailand carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in Southeast Asia), Interface United States carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in the United States), Interface US carpet, kettle, LG-E970, Mirra 2 Chair, New Aeron Chair, Page Printer SPEEDIA N3600, Regular Straight 505® Jeans – House Cat, Regular Straight 505® Jeans – Range, Regular Straight 505® Jeans – Range (Water<Less™), Regular Straight 505® Jeans – Steel (Water<Less™), Regular Straight 505® Jeans – Steel (Water<Less™), Setu Multipurpose chair with Lyris 2, Slim Straight 514™ Jeans – Indigo Wash, Slim Straight 514™ Jeans – Indigo Wash, Slim Straight 514™ Jeans – Rigid Tank, Slim Straight 514™ Jeans – Rigid Tank, Slim Straight 514™ Jeans – Tumbled Rigid, SPEEDIA N3600, Vaccum Cleaners, Vaccum Cleaners (Canisters & Uprights)|Consumer Durables & Apparel|
|BillerudKorsnäs Liquid LC, Corrugated box, PET preforms, Solid Board, Tetra Brik® Aseptic Base 1000ml, Tetra Brik® Aseptic Base 250ml, Tetra Brik® Aseptic Slim 200ml, Tetra Brik® Aseptic Slim 250ml|Containers & Packaging|
|ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER, Cable, D21710 Type 5 Corded Power Drill, DC988 Type 11 Cordless Power Drill equipment with rechargeable NiCd batteries, Electric Motor, T300, Vending Machine, Wind Turbine G114 2 Megawats, Wind Turbine G128 5 Megawats, Wind Turbine G132 5 Megawats, Wind Turbine G90 2 Megawats|Electrical Equipment and Machinery|
|"Ethanol. In addition to its use as fuel (in vehicles), ethanol is an input to the food, chemical and cosmetic industries.", Coal, Lineal Alkyl Bencene (LAB), Lineal Alkyl Bencene (LAB), Natural Gas|Energy|
|"Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)", Activia STR/PCH/BLU 24X4OZ CLUB_67740, BONAFONT NATURAL 6000 ML X2 FILM 84 (6L RPET Bottle)_54742, Coca-Cola (all packaging and sizes), DANONE MPCK STRAWBERRY 1960G (8X245G)_87121, EcoShape bottle of water (0.5 L), L-Lysine Monohydrochloride(For Feed), Nescafé soluble coffee, Quaker Oats 1Kg box sold in the United Kingdom, Quaker Oats So Simple Original and Golden Syrup products, SILHOUETTE ST/PE/RA/BL 100GX16_14967, Walkers Crisps 32.5 g sold in the United Kingdom, Walkers Crisps 50g bag sold in the United Kingdom|Food & Beverage Processing|
|Crisp’n light 7 grains, Crisp’n light 7 grains, Durum wheat semolina pasta in paperboard box (produced in United States), Light Rye, Light rye, Light rye, Malt non HDP, Malted barley, Multi Grain, Multi grain, Multi grain, Pasta produced in Greece, Pasta produced in Greece, Pasta produced in Italy (for export), Pasta produced in Italy (for export), Pasta produced in Italy (local consumption), Pasta produced in Italy (local consumption), Pasta produced in Turkey, Pasta produced in Turkey, Pasta produced in United States, Pasta produced in United States, Pesto alla Genovese sauce, Walmart Brand Products, Walmart Brand Products|Food & Staples Retailing|
|City gas|Gas Utilities|
|"Verre Infini, perfumery glass made only with domestic glass from segregation. Middle bottle weighing 100g", Perfumery specific glass. Middle bottle weighing 100g|Household & Personal Products|
|"Avanta Prima is a fully coated folding boxboard, GC2, based on 100 % fresh forest fibress. Typical end uses are high-quality pharmaceuticals, daily cosmetics and food packagings.", "Batabak 510 is a fully coated bleached paperboard with uncoated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging.", "Batabak 515 is a fully coated bleached paperboard with uncoated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging.", "Batabak 580 is a fully coated bleached paperboard with  a coated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging.", "Beverage can inside spray lacquer aluminium - Aqualure 900, Aqualure 2000 etc.", "Carta Elega is a fully coated folding boxboard, GC1, based on 100 % fresh forest fibres. Typical end uses are cosmetics and other luxury packagings and graphical applications.", "Carta Integra is a fully coated BCTMP board with a coated back,  based on 100 % fresh forest fibres. Carta Integra is suitable for graphical and packaging end uses and it is available in reels and sheets.", "Carta Solida is a fully coated BCTMP board with a white back, based on 100 % fresh forest fibres. Carta Solida is suitable for packaging and graphical end uses and it is available in reels and sheets.", "Kemiart Brite is a tradional white top kraftliner for retail packages with large colour-printed areas and line pictures with excellent contrast, based on 100 % fresh forest fibres. It is used for corrugated packaging.", "Leine Mühle, Leine Silk", "LG Chem’s automotive batteries are sustaining the undisputed No.1 position in the world based on more than 10 years of R&D capability. LG Chem is equipped with solutions for all types of electrical cars (Hybrid EV, Plugged-in EV, Battery EV) based on strong product development capability and manufacturing competitiveness via mass production experience. This enables LG Chem to supply battery cell, pack and BMS (Battery Management System) to global carmakers such as GM, Ford, Renault, Hyundai-Kia and Volvo. LG Chem expects the increased revenue from its automotive battery business by being awarded next-generation projects from existing customers while strengthening strategic partnerships, and securing new customers. Additionally, LG Chem plans to launch new products to meet customer needs in new applications such as SLI and garden tools.", "Organic pigments. PCF value for organic pigments, cradle to gate", "PCF value for metal and effect pigments, cradle to gate", "Polypropylene (PP) or Polypropene, which has the molecular formula of (CH (CH3) - CH2)n is a resin produced from the polymerization of monomer Propene, its main feedstock. It may also be produced from a mixture of two types of monomers (ethylene and propylene), producing the PP Copolymer. This is the carbon footprint for the Polypropylene produced in Brazil. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product.", "Simcote is a fully coated folding boxboard with a washed groundwood middle layer, based on 100 % fresh forest fibres. Typical end-uses are food and general packaging.", "Soda Ash for Wal-Mart Stores, Inc.", "Sodium chlorate as solid phase, produced in South America.", "The Polyethylene (PE), molecular formula (CH2)n, is currently the world's main plastic material. Its properties depend on the shape of the polyethylene — given that the ones of low density are more flexible while the high density polyethylenes are more rigid. This material is produced from the polymerization of the monomer ethylene. Despite the differences between the types of polyethylene produced by Braskem, this assessment was targeted at finding an average value of emissions for all of the resins produced. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product.", "The Polyvinyl Chloride (PVC; -(CH2-CHCl)n-) is the oldest plastic material. This material is produced from the polymerization of the monomer MVC. Braskem has a chlorine production unit, base material, which shall be used for the production of DCE (1,2-dichloroethane) from the mixture of chloride and ethylene. Afterwards, the DCE reacts to form MVC, the monomer for the polymerization process. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product.", 18mm Cap, 480ml Bottle, ABS(Acrylonitrile Butadiene Styrene), Aggregated global sales for Dell Inc, Aggregated Global Sales for Johnson & Johnson, Aggregated Global Sales for Johnson & Johnson, Aggregated Global Sales for Johnson & Johnson - Polyacrylates and other products, Aggregated Global Sales for KAO, Aggregated Global Sales for L'Oreal, Aggregated Global Sales for L'Oreal, Aggregated Global Sales for L'Oreal - Polyacrylates and other products, Aggregated Global Sales for Unilever, Aggregated Global Sales for Unilever, Aggregated Global Sales for Unilever - Polyacrylates and other products, Aggregated sales to Symrise, Algro Helix, AlgroDesign, Alumina, Amines, Argon, Automotive Batteries, Automotive Battery, Average of all products sold to Braskem, Average of all products sold to Braskem, Average of all products sold to Braskem, Average of all products sold to KAO, Beverage can inside spray lacquer aluminium, Beverage can inside spray lacquer steel, Beverage can inside spray lacquer steel - Aqualure 905, BillerudKorsnäs Liquid LC, Binders, Board packaging, BPA(Bisphenol-A), Carbon Black, Cast aluminum products, Caustic products delivered to ICL, Chelates and Micronutrients sold to ICL, Chlorine dioxide water solution produced in South America., Chlormethane products delivered to ICL, Cold Rolled Steel, Complete catalyst system for diesel-powered passenger car exhaust, Corrugated Box, Corrugated box, Corrugated box average Corrugated box Sab miller, cyanodiester, De-icing fluids, Dense Sodium Carbonate, ELM, Exolit ammonium polyphosphate\t, Exolit OP phosphinates\t, Fajar Board, Filtro 26 WS, Finished cold rolled coil, Folding boxboard, Folding boxboard produced at the Maule and Valdivia mills., Galvanised Steel, Glass container, Glass container, Glass container, Glass container, grinding media cast., H10311 - La vie est belle, H4953 - Armani IO 100 ml, Hot Dip Galvanised Coil, Hot rolled coil, Hot Rolled Steel, Hydrogen peroxide, Incada, Invercote, IT&New Application Battery, KURALON  fiber, Light Sodium Carbonate, Methacrylic acid, Metsä Board Classic FBB (Simcote), Metsä Board Classic FBB CX (Tako Lite CX)), Metsä Board Natural FSB Cup - new product, Metsä Board Prime FBB (Carta Elega), Metsä Board Prime FBB Bright (Carta Integra), Metsä Board Prime FBB CX (Tako CX White S), Metsä Board Prime FBB CXB (Carta Allura), Metsä Board Prime FBB CXL (Tako HiD L JTI), Metsä Board Prime FBB CXX (Tako HiD X JTI), Metsä Board Pro FBB (Avanta Prima), Metsä Board Pro FBB Bright (Carta Solida), MetsäBoard Pro FBB CX (Tako CX Lite S), Mobile Batteries, Nitrogen, One side coated flexible packaging paper, One side coated label paper, Organic coated coil for the Nature product range (which includes the products Granite® and Estetic®) having a mean surface mass of 4.7kg. The Nature range of innovative chromium-free organic coated steels for the construction industry is the result of 15 years of research in partnership with paint suppliers to develop safer and greener corrosion-inhibiting pigments., Organic Pigments, Organic Pigments, Overprint varnishes, Overprint varnishes  (also used to approximate rim varnishes), Parade Prima, Paraformaldehyde, Peroxides, PET preforms for bottles, Plastics, Polyethylene, Polyethylene, Polypropylene, Polypropylene, Polyvinyl Chloride, Polyvinyl Chloride, Portland Cement, Primary aluminum ingot, Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall, Rubber, SBS Board, Semi-finished aluminum sheet (cold rolled products), Semi-finished aluminum sheet (cold rolled products), Semi-finished aluminum sheet (hot rolled products), Semi-finished aluminum sheet (hot rolled products), Soda Ash, Soda Ash, Soda Ash for Unilever plc, Soda Ash from Tata Chemicals Europe, Sodium Bicarbonate, Sodium Bicarbonate, Sodium Carbonate, Sodium chlorate produced in North America., Sodium sulbactam 3, sodium sulphate, Solid Board, Sunliquid\t, Super-pure hydrogen peroxide, TCDE, Tetra Brik® Aseptic Base 1000ml, Tetra Brik® Aseptic Base 1000ml, Tetra Brik® Aseptic Base 1000ml, Tetra Brik® Aseptic Base 250ml, Tetra Brik® Aseptic Base 250ml, Tetra Brik® Aseptic Base 250ml, Tetra Brik® Aseptic Slim 200ml, Tetra Brik® Aseptic Slim 200ml, Tetra Brik® Aseptic Slim 200ml, Tetra Brik® Aseptic Slim 250ml, Tetra Brik® Aseptic Slim 250ml, Tetra Brik® Aseptic Slim 250ml, Three-way Catalyst for gasoline-powered passenger car exhaust, Uncoated flexible packaging paper|Materials|
|"Bloomberg's biometric device (B-Unit) distributed to all Bloomberg Terminal customers.  The functional unit has a lifespan of 3 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan.", "Bloomberg's standard keyboard which is optional hardware available for lease to all Bloomberg Terminal customers.  The functional unit has a lifespan of 3 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan.", "Bloomberg's standard-issue flat panel configuration (prior to 2010) was two 19\" panels mounted on a metal stand. In early 2010 Bloomberg engaged in the WRI Product Life Cycle Roadtest for this functional unit (cradle-to-grave). The functional unit has a lifespan of 5 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan.", "TG582n Home Gateway to provide multiple play telecom data services (data, voice, TV) in the home.", Bloomberg B-Unit, Bloomberg B-Unit, Bloomberg B-Unit, Bloomberg-Provided Flat Panel Unit, Bloomberg-Provided Flat Panel Unit, Bloomberg-Provided Flat Panel Unit, Bloomberg-Provided Keyboard, Bloomberg-Provided Keyboard, Bloomberg-Provided Keyboard, Home Gateway TG582n, Home Gateway TG582n|Media|
|Staples Sustainable Earth 12A toner cartridge, Staples Sustainable Earth 12A toner cartridge, Staples Sustainable Earth 38A toner cartridge, Staples Sustainable Earth 38A toner cartridge|Retailing|
|Desktop CPU, IC chips of Smartphone, Mobile CPU, Server CPU|Semiconductors & Semiconductor Equipment|
|4Gb LPDDR2 SDRAM, 4Gb LPDDR3 SDRAM, 64Gb NAND Flash MLC|Semiconductors & Semiconductors Equipment|
|1 DVD Software Package, ColorQube 8870, ColorQube 8870, ColorQube 8900, ColorQube 8900, ColorQube 9200, ColorQube 9200, Desk top personal computer, DVD software, DVD software (1 DVD), DVD software (2 DVDs), Phaser 6600, Phaser 6600, Phaser 7100, Phaser 7100, Server, USB software, USB software, USB Software, WorkCentre 5325, WorkCentre 5325, WorkCentre 5330, WorkCentre 5330, WorkCentre 5335, WorkCentre 5335, WorkCentre 5945/5955, WorkCentre 5945/5955|Software & Services|
|"19” LCD Monitor  Weight: 2.4Kg  Diagonal size: 19\"  Power consumption: <16.25W (on mode), <0.72W (power saving mode), <0.53W (off mode) Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide", "30L desktop model vBeijing2_I for design stage plus manufacturing stage, made by ECS's factory loacted in southern China (GHG emission inventoried in 2012 by methodology of ISO 14064-1 and allocation basis by weight)", "50” TFT-LCD TV module which consisted of the following main components: PCBA, converter, TFT-LCD panel, LED backlight", "Mouse, M185", "Nokia Asha 300. Nokia's low end device, mass = 85 g incluging battery.", "Nokia Lumia 800. Nokia's high end smart phone, mass = 142 g including battery.", "Notebook (Classmate PC) E11IS is manufactured & designed by ECS's factory located in eastern China, its GHG emission inventoried in 2012 by methodology of PAS 2050:2011 and verified by SGS ready.", "PowerEdge R710 is a typical high-volume, next-generation Intel Xeon processor-based 2U Rack Server that is representative of a range of similar server products.", "Ricoh Aficio MP 6002SP, a Mono-chrome MFP (Multi-function products) products that function as Copier, Network Printer, Scanner, (Facsimile).", "Ricoh Aficio MP C5502G, a Full-color MFP (Multi-function products) products that function as Copier, Network Printer, Scanner, (Facsimile).", "Ricoh Aficio SP 8300DN, a high-speed Mono-chrome Network Printer", "Ricoh Aficio SP C831DN, a high-speed Full-colour Network Printer", "Ricoh Pro C751EX, a Production Printer (Full Color) that is used primarily for high-speed and production type (large print volume) job", "TFT-LCD products is the use of the optical properties of liquid crystals to achieve the image display, liquid crystal main structure is poured into the vacuum between the two pieces of glass, glass, applied voltage and control the proper spacing of the deflection characteristics of the incident light can be changed to to image display. For the automobile products's TFT-LCD Module, the product is defined as B to B (Business to Business) products, ranging from the cradle to the door inventory, according to standard PCR system boundaries should be compulsory included in carbon footprint assessment phase are as follows:  . Mining and raw materials made  . Manufacture of major components  . Transport components  . Product assembly stage. The models 069LA of automobile panel was complete the dual certification of carbon footprint and water footprint  by SGS in 2011.", "The Latitude E6400 is a typical high-volume, mainstream business laptop that is representative of a range of similar laptop  products. It is Energy Star® 5.0 qualified and EPEAT Gold registered.", "The OptiPlex 780 Mini Tower is a typical high-volume, mainstream business desktop that is representative of a range of similar desktop products. It is Energy Star® 5.0 compliant and EPEAT Gold registered.", 2150cdn  printer, 42' Industrial Solutions, Acer 15.6\ Notebook", AIO, AIO, Auto, Automotive Relay, Automotive Relay, Automotive Relay, Average mobile phone, BlackBerry Classic, BlackBerry Passport, BlackBerry Passport, BlackBerry Priv, BlackBerry Z10, Blood Pressure Monitor, Blood Pressure Monitor HEM-7113, Broadband Routers, C1760nw Perinter, C2660dn, C2660dn, C2660dn, C2665dnf, C2665dnf, Camera, Camera, CD-ROM, CD-ROM, CHROMEBOX, ColorQube 8870, ColorQube 8870, ColorQube 8900, ColorQube 8900, ColorQube 9200, ColorQube 9200, CS310DN - Color Laser Printer, CS410DN - Color Laser Printer, CS510DE - Color Laser Printer, CX310DN - Color Laser Printer, CX410DE - Color Laser Printer, CX510DE - Color Laser Printer, DC fan, Desk Top PC  ESPRIMO Ｄ582/Ｅ, Desktop CPU, Digital camera, Digital Photo Frame Weight: 0.8 kg Diagonal size: 7D Overall Dimension: 238*202*39.7mm Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide., DT, ECU, ECU, Elevator, External power supply/ portable adaptor, H000 (Set-top Box), H625cdw, H815dw, H825cdw, HH DVD, HP Compaq 8200 Elite SFF Business PC, HP Compaq Elite 8300 MT PC, HP Compaq Elite 8300 USDT PC, HP Compaq LA2206xc 21.5-inch Webcam Monitor, HP Compaq LA2405x 24-inch LED Backlit Monitor, HP EliteBook 8460p Notebook PC, HP EliteBook 8570w Mobile Workstation, HP EliteBook Folio 9470m, HP EliteDisplay E201 20-inch LED Backlit Monitor, HP Envy 120 InkJet Printer, HP LaserJet Enterprise M4555fskm MFP (note that this product is representative of 61 similar products for which full LCAs were performed), HP ScanJet 7500 (note that this product is representative of 5 similar products for which full LCAs were performed), Industrial Solutions, Industrial Solutions, Ingenico terminals, IP Phone, Keyboard, Keyboards, Keyless Entry System for Automobiles(Transmitter + Receiver unit), Lamp, Laser radar, Latitude E3440, Latitude E3540, Latitude E5440, Latitude E5540, Latitude E6440, Latitude E6540, Latitude E7240, Latitude E7440, LCD PC, LED Lighting, LED Lighting, LED Lighting, M5163 - Mono Laser Printer, Mobile CPU, Mobile Infotainment, Mobile Network Equipment, Mobile Phone, Mobile Phone, Mobile Phone, Mobile phone, Mobile phone Weight: 0.11kg Screen Diagonal: 3.7\ WVGA (800*480) 16M Color TFT-LCD Power Consumption: Standby time(1300m Ah): Up to 4 hrs depends on real-network Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide.", Monitor, Monitor, Monitor, MS310DN - Mono Laser Printer, MS312DN - Mono Laser Printer, MS315DN - Mono Laser Printer, MS410DN - Mono Laser Printer, MS415DN - Mono Laser Printer, MS510DN - Mono Laser Printer, MS610DE - Mono Laser Printer, MS610DN - Mono Laser Printer, MS810DE - Mono Laser Printer, MS810DN - Mono Laser Printer, MS811DN - Mono Laser Printer, MS812DE - Mono Laser Printer, MS812DN - Mono Laser Printer, Multi Function Printer, Multi Function Printer with a built-in facsimile, Multifunction Printers, Multifunction Printers, Multifunction Printers, MX310DN - Mono Laser Printer, MX410DE - Mono Laser Printer, MX511DE - Mono Laser Printer, MX611DE - Mono Laser Printer, MX710DE - Mono Laser Printer, MX711DE - Mono Laser Printer, MX810DFE - Mono Laser Printer, MX811DFE - Mono Laser Printer, MX812DFE - Mono Laser Printer, Neosensor indoor temperature sensor (ネオセンサ 室内用温度センサ), Netbook (SKU: High efficient CPU.10.1-inch LED backlight), Note book, Notebook, Notebook, OptiPlex 3010 Minitower (MT), OptiPlex 3030 all-in-one, OptiPlex 9010 Small Form Factor (SFF), Pad, Phaser 6600, Phaser 6600, Phaser 7100, Phone, Photoelectric sensor (光電センサー), PND, POS, power supply unit (MF6004-030L), Power window switch, PowerEdge R710, Pressure transmitter (圧力発信器) GTX, Projector, Projector, Projector, Projector  Weight: 7.5Kg Lamp: 24 pcs laser with phosphor Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide., PV Inverter, Ricoh MP 6054SPG, Ricoh MP C5503, Ricoh MP C8002SP: The product is a high-speed (80 images/minute) color multi-functional product (MFP) and is a major representative of our image solution products business.  The lifecycle emissions data of this product is provided as a case example of our entire image solution business products line which covers 77% of Ricoh's total revenue., RICOH Pro C7110S QX100, Ricoh SP C352DN, S2815dn, S2825cdn, S3840cdn, S3845cdn, Safety controller, Scanner  Weight: 1.4 kg Power supply: 12V/1.5A Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide., Server, Server, Server CPU, Servers, Signal Relay Type G6Zk-1F-TR, Single Lens Reflex camera system, Small Rack Mount Switch, Speaker, Surveillance Camera & Products, Switch, Tablet, Temperature sensor for duct (ダクト用温度センサ), TPS (Telecom Power System), VR, Watch|Technology Hardware & Equipment|
|BlackBerry Bold 9900, Graphite 2500 DECT, Graphite 2500 DECT, Home Hub 2.0, Home Hub 2.0, Home Hub 3.0, Home Hub 3.0, Vision+ box, Vision+ box|Telecommunication Services|
|1 fuel-efficient tire for passenger cars (乗用車用低燃費タイヤ), 1 fuel-efficient truck and buss tireトラック・バス用低燃費タイヤ１本当たり|Tires|
|Tobacco|Tobacco|
|CHEP wooden exchange (pooled) pallet in Canada 48x20, CHEP wooden exchange (pooled) pallet in Canada 48x40, CHEP wooden exchange (pooled) pallet in USA 48x40, Foldable reusable plastic crates (RPCs) in Australia, IFCO reusable plastic crates (RPCs) in North America, Rigid side reusable plastic crates (RPCs) in Australia|Trading Companies & Distributors and Commercial Services & Supplies|
|City gas, City gas|Utilities|

The dataset provides an extensive cross-section of products and their associated industry groups, revealing how carbon emissions are embedded across global production and consumption systems. Examining these categories from a life-cycle perspective shows that certain industries concentrate significant emission profiles due to the scale, material intensity, and energy requirements of their supply chains. Food, Beverage, and Tobacco products, for example, exhibit substantial emissions because agricultural inputs, processing stages, packaging, and cold-chain logistics demand high resource use and continuous energy inputs. Energy and Chemical sectors similarly reflect the high carbon density of their extraction, transformation, and transport processes. By contrast, sectors like Technology Hardware & Equipment, while often lighter per unit, accumulate notable footprints due to complex globalized supply chains, rare material extraction, and intensive manufacturing steps.

Another striking dimension is the overlap between consumer durables and apparel, where long product lifespans may diffuse emissions per year of use but still entail considerable embedded carbon at production. Industries such as Capital Goods and Electrical Equipment and Machinery link directly to infrastructure and renewable energy supply chains, showing that even “green” technologies like wind turbines embody substantial upfront emissions through steel, concrete, and logistics. These insights point to the importance of not only measuring but differentiating between production-phase emissions and downstream usage or disposal impacts. Moreover, grouping products into industry clusters provides a clearer pathway for policy intervention, supplier engagement, and innovation targeting. Strategic decarbonization must therefore address sector-specific drivers of emissions, while promoting material efficiency, renewable energy adoption, and sustainable product design.

side note:
Instead of using the traditional listing, i thought of using GROUP_CONCAT for ease of table making, thus eliminating sspace on the table design (and saves time on writing "Technology Hardware & Equipment" 30000 times)

## What are the industries with the highest contribution to carbon emissions?
### SQL query used:
```
WITH handle AS (
  SELECT *
  FROM (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY id) AS rn
    FROM product_emissions pe
  ) AS raw
  WHERE raw.rn = 1
)
SELECT 
    ig.industry_group,
    SUM(h.carbon_footprint_pcf) AS total_carbon_emissions
FROM handle h
JOIN industry_groups ig 
  ON h.industry_group_id = ig.id
GROUP BY ig.industry_group
ORDER BY total_carbon_emissions DESC;
```

result: 
|industry_group|total_carbon_emissions|
|--------------|----------------------|
|Electrical Equipment and Machinery|9801558|
|Automobiles & Components|2582264|
|Materials|430199|
|Technology Hardware & Equipment|278650|
|Capital Goods|258633|
|"Food, Beverage & Tobacco"|109132|
|"Pharmaceuticals, Biotechnology & Life Sciences"|72486|
|Software & Services|46533|
|Chemicals|44939|
|Media|23017|
|Energy|10774|
|"Forest and Paper Products - Forestry, Timber, Pulp and Paper, Rubber"|8909|
|"Mining - Iron, Aluminum, Other Metals"|8181|
|Consumer Durables & Apparel|7097|
|Commercial & Professional Services|4925|
|Containers & Packaging|2988|
|Tires|2022|
|Food & Staples Retailing|1481|
|"Consumer Durables, Household and Personal Products"|931|
|Telecommunication Services|418|
|Trading Companies & Distributors and Commercial Services & Supplies|239|
|"Textiles, Apparel, Footwear and Luxury Goods"|228|
|Food & Beverage Processing|138|
|Utilities|122|
|Gas Utilities|61|
|Semiconductors & Semiconductor Equipment|52|
|Retailing|22|
|Semiconductors & Semiconductors Equipment|3|
|Tobacco|1|
|Household & Personal Products|0|

The ranking of industry groups by total carbon emissions underscores the unequal distribution of environmental impact across economic sectors. Electrical Equipment and Machinery leads decisively with 9,801,558 units of carbon emissions, which likely reflects the energy- and materials-intensive nature of manufacturing turbines, generators, and large-scale industrial equipment. Automobiles & Components, at 2,582,264 units, represents the significant emissions embedded in vehicle production and its complex global supply chains. Materials and Technology Hardware & Equipment also score high, indicating the emissions inherent to steel, aluminum, plastics, and electronics fabrication. These industries depend heavily on resource extraction, high-temperature processes, and extensive logistics, which amplify their carbon footprints.

By contrast, sectors such as Food, Beverage & Tobacco, Chemicals, and Pharmaceuticals exhibit far smaller totals. These industries often involve energy-intensive processing but at lower volumes of heavy raw materials compared to industrial manufacturing. Lower-emitting categories such as Telecommunications, Retailing, and Textiles show that while their operations are widespread, the per-unit material intensity and energy consumption are more moderate. The data also reveal niche sectors with negligible emissions, such as Tobacco and Household & Personal Products, either due to underreporting or minimal production scale.

This distribution illustrates that decarbonization efforts cannot be uniform; instead, they must target the high-impact sectors first. Policy makers, investors, and industry leaders can use such rankings to identify where carbon reductions yield the greatest benefit. Electrification of processes, low-carbon materials, circular manufacturing, and renewable energy integration could sharply reduce emissions in top contributors like Electrical Equipment, Automobiles, and Materials.

## What are the industry groups of these products?
### SQL query used:
```
WITH handle AS (
WITH handle AS (
  SELECT *
  FROM (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY id) AS rn
    FROM product_emissions pe
  ) AS raw
  WHERE raw.rn = 1
)
SELECT 
    c.company_name,
    SUM(pe.carbon_footprint_pcf) AS total_carbon_emissions
FROM product_emissions pe
JOIN companies c 
  ON pe.company_id = c.id
GROUP BY c.company_name
ORDER BY total_carbon_emissions DESC
LIMIT 10;
```

result: 
|company_name|total_carbon_emissions|
|------------|----------------------|
|"Gamesa Corporación Tecnológica, S.A."|9778464|
|Daimler AG|1594300|
|Volkswagen AG|655960|
|"Mitsubishi Gas Chemical Company, Inc."|212016|
|"Hino Motors, Ltd."|191687|
|Arcelor Mittal|167007|
|Weg S/A|160655|
|General Motors Company|137007|
|"Lexmark International, Inc."|132012|
|"Daikin Industries, Ltd."|105600|

The ranking of companies by total carbon emissions underscores how manufacturing scale, product type, and supply chain complexity shape environmental impact. Gamesa Corporación Tecnológica, S.A. stands out with an exceptionally high total of 9,778,464 units, reflecting the immense resource and energy requirements of producing large-scale wind turbines and related electrical equipment. While these technologies support renewable energy, their production involves steel, concrete, and global logistics chains with heavy embedded emissions. Daimler AG and Volkswagen AG follow as major automobile manufacturers whose emissions embody vehicle assembly, extensive parts networks, and energy-intensive operations.

Mitsubishi Gas Chemical Company, Inc. and Hino Motors, Ltd. represent specialized industrial producers whose processes—from chemical manufacturing to heavy-duty vehicle production—generate significant greenhouse gas outputs. Arcelor Mittal, the steel giant, shows how basic materials industries continue to anchor the carbon footprint of global manufacturing due to high-temperature smelting and fossil fuel reliance. Companies like Weg S/A, General Motors Company, and Daikin Industries, Ltd. demonstrate that even diversified industrial producers in motors, air conditioning, and related sectors carry substantial emissions profiles.

Lexmark International, Inc.’s presence highlights that emissions are not limited to heavy industry; electronics and technology companies also accumulate considerable footprints through globalized supply chains, mineral sourcing, and energy-intensive fabrication. This distribution of emissions points to the necessity of industry-specific decarbonization strategies. Targeting the highest contributors first—by redesigning products, adopting low-carbon materials, improving energy efficiency, and integrating renewable energy into production—offers the most immediate and substantial reductions in corporate carbon footprints.

