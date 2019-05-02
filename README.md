# CRA Data Project
My conference session is based on a project I undertook over the summer of 2018. The project was about efficiently finding a way to compare Canadian Revenue Agency's (CRA) charity data to the BCIT's internal data. This project had the following outcomes:

1. Identify foundations with a pre-existing relationship with BCIT that have given to other comparable charities in greater amounts.
2. Identify directors who may also be alumni.

This document discusses

- **Population Characteristics:** the population characteristics of the CRA and BCIT data
- **Methodology:** the methods and tools used to access, stage, and compare data
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

The query for this project resulted in 200MB of data and required a physical disk shipped to my office. The request was made in April of 2018, and a package was mailed from the CRA on June 8, 2018.

### Staging Data
Staging of data (preparing it for comparison with BCIT's internal data) was approached differently for the foundation and director comparisons.

#### Foundation Comparison
BCIT has an attribute attached to each registered charity record in its internal database that details the business number of that charity. Therefore, staging of data for this comparison was negligible as an equality could be established between this attribute and the business number provided from the CRA.

#### Director comparison
Staging was slightly more complex for the Directors comparison. Names, especially when viewed with little or no other additional factors of identification (contact information, addresses, employment positions) can be incredibly difficult to match with confidence. 

To support the fuzzy matching of names, all information BCIT held internally about constituent records was included in an export and combined in separate fields in several ways:

- Individual field names (fname, mname, lname, nickname)
- combined names (fullname with/without mname, nickname in place of fname)
- Acronyms

These three types of name representations conformed to what was available from the CRA.

### Comparison
As mentioned, the Foundation comparison was fairly straight forward. BCIT maintains the business numbers of charities it has engagement with as a matter of policy. For this specific project, an export from our database and the CRA data were loaded into Microsoft Access and a duplication query was run on the business number fields of each dataset.

The Directors comparison used the tool [CSVDedupe](https://github.com/dedupeio/csvdedupe), a simplified commandline tool based on the python library [Dedupe](https://dedupe.io/).

#### CSVDedupe
CSVDedupe explicitly supports two functions: 

1. **Deduplication:** tagging rows in a CSV file it suspects are duplicates of other rows in the same file.
2. **Linking:** links together two CSV files on rows it believes are duplicates.

In each case CSVDedupe requires a configuration file that defines which fields it should take into consideration when determining whether or not rows are duplicates in addition to a training period.

##### Configuration File
The configuration file is written in JSON, but is incredibly simple. The project file is quoted below and included as file in this repository.

```Javascript
{
	"field_names_1":	["fullName", "firstName", "middleName", "lastName", "nickname", "acr"],
	"field_names_2":	["Full Name", "First Name", "Initial", "Last Name"],
	"field_definition":	[{"field" : "fullName", "type" : "String"},
						 {"field" : "firstName", "type" : "String"},
						 {"field" : "middleName", "type" : "String"},
						 {"field" : "lastName", "type" : "String"},
						 {"field" : "nickname", "type" : "String"},
						 {"field" : "acr", "type" : "String"}],
	"output_file":		"C:\\Users\\$USER_ACCOUNT\\Desktop\\csvdedupe\\output_cracom2017.csv",
	"skip_training":	false,
	"training_file":	"C:\\Users\\$USER_ACCOUNT\\Desktop\\csvdedupe\\training.json",
	"sample_size":		150000,
	"recall_weight":	2
}
```

A detailed explanation for this file is available [here](https://github.com/dedupeio/csvdedupe#csvlink), but as you will notice, many of the options didn't change from what is included in the tutorial documentation. I highly recommend copying what is provided there and editing to your needs if this is your first experience with JSON.

##### Launching CSVDedupe
I used the following one-liner in my commandline to launch the training session:

```
csvlink C:\\Users\\$USER_ACCOUNT\\Desktop\\csvdedupe\\input_bcit.csv C:\\Users\\$USER_ACCOUNT\\Desktop\\csvdedupe\\input_cra.csv --config_file=config.json --inner_join
```

I included the ```inner_join``` flag because without it the output file would include all the concatenated rows between the files in addition to all the rows that didn't have any matches. Considering both compared files initially had ~200K rows, I didn't want to tempt Excel with a crash by combining them.

##### Training
Here is where the machine learning magic is created. CSVDedupe needs to be taught what is and isn't a match. It accomplishes this by presenting you with matching candidates and asks you whether they match, don't match, or if you're unsure. My prompt looked similar to this.

```
fullname :  John Burrard Smith
firstname :  John
middlename : Burrard
lastname :  Smith
nickname : Jelly
acr: JBS

Full Name :  John A. Smith
First Name :  John
Initial : JAS
Last Name :  Smith

Do these records refer to the same thing?
(y)es / (n)o / (u)nsure / (f)inished
```

The documentation for CSVDedupe [recommends](https://github.com/dedupeio/csvdedupe#training) at least 10 positive and negative matches. However, this recommendation is heavily dependent on your source data. For example, it required some 200 negative matches before I could confirm 10 positive matches. This is likely the result of my single point of comparison being names. 

If I had additional points of comparison, like addresses, contact information, or occupations the system would have had a far easier time presenting me with better candidates.

### Results
#### Foundation Comparison

#### Director Comparison

#### Limitations and Issues

#### Opportunities
