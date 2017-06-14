---
layout: datasource
tablename: zone_housingunit_bedrm_count
title: Housing Unit and Bedroom Count by Zone
---
<!--No need to put a header; the title in the front matter (above) will be used as a header-->

This table, "zone_housingunit_bedrm_count", details total number of housing units and bedrooms in a specific zone (example: 2500 housing units and 4000 bedrooms in ward 1).

The individual building data was acquired from Computer Assisted Mass Appraisal (CAMA) (http://opendata.dc.gov/datasets/computer-assisted-mass-appraisal-residential). Each of these building data points were given the Square Suffix Lot (SSL) ID, number of housing units in the building, number of bedrooms in the building, and the residential use code.

CAMA did not provide in what specific zone the building is located in. We took each CAMA datapoint and accessed Master Address Repository (MAR) API; more specifically we used MAR findAddFromSSL2 (http://citizenatlas.dc.gov/newwebservices/locationverifier.asmx?op=findAddFromSSL2). When CAMA data points were searched via MAR, we were able to find the building's specific zone (zip code, ward, census tract, ANC, neighborhood cluster).

We took the CAMA data point (total number of housing units and bedrooms for that building) and MAR data (where the building is located) to count how many housing units and bedrooms are in each specific zone (zip code, ward, census tract, ANC, neighborhood cluster).



## Assumptions made for CAMA data points with EYB of 2018
CAMA data includes buildings under construction. CAMA's data includes EYB of 2018 as of June 2017. We eliminate all data points that are under construction and don't provide any housing units and bedrooms at this time.


## Assumptions made for CAMA data points with 0 value for housing units
In the CAMA data, buildings with 0 num_units (the data for number of housing units) falls under number of cases. There are about 177 data points in CAMA (as of June 2017) with 0 housing units.

First set of CAMA data has 0 num_units and 0 bedrooms. However, they have associated MAR data points. Also their use codes are residential. Some examples are:
(SSL: 1264 0808) // according to Integrated Tax System API: land use code 023 - Residential Flats-Less than 5
(SSL: 1256 0078) // land use code 012 - Residential-Detached-Single-Family
The assumption was that these buildings are used for the purpose of residential because they are taxed as such. Therefore there should be at least 1 bedroom and 1 housing unit. There should be less than 10 data points that meet this criteria, so even if the assumption is off, the error should be negligible in a data set with 107,000+ data points.

Second set is certain data points for single family homes (and the most numerous data points with 0 num_units). They have multiple bedrooms but they show up as 0 num_units even though it should be 1. They also have MAR data point when you call it on the MAR API. An example is (SSL: 0069 0805). CAMA data shows 0 num_units and 3 bedrooms. MAR API has appropriate data. When looking at Integrated Tax System API you see building code 011 for Residential-Row-Single-Family. These housing units should be counted as 1 housing unit instead of 0, as they contain bedrooms and serve as a housing unit. There are about 154 data points out of 177.

Third set of data point with 0 housing units are data point with 0 housing units and 0 bedroom but has no MAR data associated with them. An example is (SSL: 0514 0871). They are excluded from our data points as there is no way to figure out which zone they belong in. This is done by attaching "Warning" tag in our MarApiConn which is then excluded in get_data method of CamaApiConn via if 'Warning' not in mar_return.keys():. There are less than 20 data points are in CAMA.


## Assumptions made for CAMA data points with 0 value for bedrooms
In the CAMA data, there are about 207 data points that have 0 bedrooms.

172 out of 207 have 1 housing unit but contains "0 bedroom". Many has land use code of 011 - residential single family housing. Assumption was made that 1 housing unit should be equivalent to 1 bedroom.

There are also data points like above with 0 housing units and 0 bedrooms. Assumption was made that if the MAR data exists then there should be at least 1 housing unit and 1 bedroom.




## Remaining issues
* Currently trying to refactor the cama.py and mar.py as discussed in issue 336 https://github.com/codefordc/housing-insights/issues/336
