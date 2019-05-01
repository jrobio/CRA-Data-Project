# CRA Data Project
My conference session is based on a project I undertook in August of 2018. The project was about efficiently finding a way to compare Canadian Revenue Agency's (CRA) charity data to the BCIT's internal data. This project had the following outcomes:

1. Identify Foundation that have given to other comparable charities in greater amounts
2. Identify Directors who may also be alumni

This document discusses

- **Population Characteristics:** the population characteristics of the CRA and BCIT data
- **Methodology:** the tools used to access, stage, and comparing of data
- **Results:** What the project resulted in as well as limitations and opportunities for further exploration with this data.

## Population Characteristics
### CRA Data
- year of 2017
- Charities headquartered in: Alberta, British Columbia, Ontario, and Quebec
- Charities Types: Benefits to Community, Welfare, Education, and Other
- 24,640 Foundations
- 200,016 Directors

### BCIT Data
- ~190,000 records
- 90%+ of which are individuals

## Methodology
### Accessing Data
The CRA provides access to the public portions of a registered charity's T3010 tax return. This return includes detailed information on donations to Qualified Donees (organizations that have charitable business numbers themselves) and a listing of directors.

This information is available for querying by the public through a [search interface](https://www.canada.ca/en/revenue-agency/services/charities-giving/charities-listings.html) for individual entities or in bulk through its [data store](http://www.cra-arc.gc.ca/chrts-gvng/lstngs/rqstfrm-eng.html).

The query for this project was large enough that it required a physical disk shipped to my office. The request was made in April of 2018, and a package was mailed from the CRA on June 8, 2018.

