# NYC Public School Performance Analysis

## Project Overview

This project implements a **Geographic Performance Analysis** of New York City Public Schools' performance data, primarily focusing on **Quality Review (QR) Ratings**. (https://www.schools.nyc.gov/about-us/reports/school-quality/quality-review)

The core **objective** is to investigate the relationship between school performance (over time) in different **geographic groupings** (ZIP Codes, Census Tracts, and Administrative Districts) against MTA/public transport On-Time Performance data, to investigate any possible correlations.


## Data Sources from NYC OpenData:
- https://data.cityofnewyork.us/Education/NYC-DOE-Public-School-Location-Information/3bkj-34v2/about_data - School Location Data
- https://data.cityofnewyork.us/Education/2005-2020-Quality-Review-Ratings/3wfy-sn5g/about_data - School Quality Review Ratings

## Data Sources from DATA.NY.GOV:
- https://data.ny.gov/Transportation/MTA-Subway-Stations/39hk-dx4f/about_data - MTA Subway Station Locations
- https://data.ny.gov/Transportation/MTA-Subway-Terminal-On-Time-Performance-2015-2019/f6rf-2a3t/about_data - MTA Subway On-Time Performance (2015-2019)
- https://data.ny.gov/Transportation/MTA-Subway-Terminal-On-Time-Performance-2020-2024/vtvh-gimj/about_data - MTA Subway On-Time Performance (2020-2024)


-----------------------------------------------------------------------------------------------------

## Process Steps:
1. Create Exploratory Data Analysis (EDA) notebooks of school locations and QR ratings: School_Data_Analysis_Notebooks ✅

2. Create EDA notebooks of MTA station locations and On-Time Performance metrics: MTA_Data_Analysis_Notebooks

3. Clean school ✅ and MTA data; notebooks serve as the script for this cleaning to occur.

4. Display cleaned school location and QR data in Tableau, using an interactive set of bar graphs to show how QR ratings in each borough's grouped locations change over time.
    - https://public.tableau.com/app/profile/thalyann.olivo/vizzes

5. Incorporate MTA data into the previously created Tableau visualizations, for showing the impact (or lack thereof) of MTA subway performance on NYC Public School QR performance.

6. Use ML techniques (starting with K-Means Clustering) to uncover platform locations where Subway Performance impacts schools the most.

-----------------------------------------------------------------------------------------------------
## Project Goal:

1. Create an easy-to-follow set of graphs that can show if any relationship between MTA On-Time Performance and School QR ratings exists.

2. Provide meaningful, data-backed recommendations for NYC DOE data collection and an example of a python script that can take in annually updated information for further/future observation.

3. Present findings and final product to members of the City Council, MTA, and DOE; the intent would be to inform and encourage collaboration. If there is a connection between poor MTA On-Time Performance and Public School Quality Reviews, the next step would be exploring potential solutions to improve both MTA and DOE issues. 

## Ideas: 
- MTA: Express subway routes for high-traffic lines at peak morning and evening commute hours (implemented in worse performing areas/platforms)
- MTA & the Public: Implement a credit system for OMNY accounts linked to Google Maps/Apple Maps/pedometer to encourage people to walk vs take the train; user inputs stations they will walk to, and a cap on the amount of credits that can be collected in a day can be made to limit abuse of the system.
- DOE & City: Determine viable solutions for subsidizing/offering teachers housing closer to their schools (should school performance be found to be closely linked to teachers' stress/commute times)

-----------------------------------------------------------------------------------------------------

## Getting Started

### Prerequisites

You will need **Python 3.12** installed. This project relies on standard data science libraries.

You can install all required dependencies using `pip`:

```bash
pip install -r requirements.txt