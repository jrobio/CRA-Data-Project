# CRA Data Project
My conference session is based on a project I undertook over the summer of 2018. The project was about efficiently finding a way to compare Canadian Revenue Agency's (CRA) charity data to the BCIT's internal data. This project had the following outcomes: Identify directors who may also be alumni.

This document discusses

- **Population Characteristics:** the population characteristics of the CRA and BCIT data
- **Methodology:** the methods and tools used to access, stage, and compare data
- **Results:** What the project resulted in as well as limitations and opportunities for further exploration with this data.

## Population Characteristics
### CRA Data
- 2017
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

The query for this project resulted in ~200MB of data and required a physical disk shipped to my office. The request was made in April of 2018, and a package was mailed from the CRA on June 8, 2018.

### Staging Data
Names, especially when viewed with little or no other additional factors of identification (contact information, addresses, employment positions) can be incredibly difficult to match with confidence. 

To support the fuzzy matching of names, all information BCIT held internally about constituent records was included in an export and combined in separate fields in several ways:

- Individual field names (fname, mname, lname, nickname)
- combined names (fullname with/without mname, nickname in place of fname)
- Acronyms

These three types of name representations conformed to what was available from the CRA.

### Comparison
The Directors comparison used the tool [CSVDedupe](https://github.com/dedupeio/csvdedupe), a simplified commandline tool based on the python library [Dedupe](https://dedupe.io/).

#### CSVDedupe
CSVDedupe explicitly supports two functions: 

1. **Deduplication:** tagging rows in a CSV file it suspects are duplicates of other rows in the same file.
2. **Linking:** links together two CSV files on rows it believes are duplicates.

In each case CSVDedupe requires a configuration file that defines which fields it should take into consideration when determining whether or not rows are duplicates in addition to a training period.

##### Installation

1. Install [Python](https://www.python.org/)
2. Add ```C:\Program Files\Python36``` and ```C:\Program Files\Python36\Scripts``` to your [PATH](https://superuser.com/questions/949560/how-do-i-set-system-environment-variables-in-windows-10) variable (your Python location may vary depending on your OS type and version)
3. Launch a commandline and run ```pip install csvdedupe```

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
CSVDedupe needs to be taught what is and isn't a match. It accomplishes this by presenting you with matching candidates and asks you whether they match, don't match, or if you're unsure. Through this exercise, CSVDedupe is able to apply that same level of decision making to much larger datasets. My prompt looked similar to this.

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

The documentation for CSVDedupe [recommends](https://github.com/dedupeio/csvdedupe#training) at least 10 positive and negative matches.

### Results
- 5,266 candidate matches between BCIT's internal data and CRA directors
	- **ON:** 2,232
	- **BC:** 1,943
	- **AB:** 685
	- **QC:** 383
- Manual effort was spent on confirming the 1,943 BC directors, of which 60 could be positively confirmed (3% hit rate)

#### Limitations and Issues
- Factors of comparison were an issue for this project. CSVDedupe adequately identified similiar names, but it didn't have enough information to find a difference between people with effectively the same names.
- CSVDedupe is a wonderfully streamlined interface for the Dedupe library. However, it was not without its issues. I had edit the code to ensure it ran smoothly on my machine.

#### Opportunities
- Comparing Gifts towards foundations of comparable size/focus to BCIT to identify funding sources that are not engaged or donating to BCIT in lower amounts.
- Trying different schemes to introduce more factors of identification to raise the confidence of matches. This could look like:
	- Giving point values to matched individuals who are older. The thinking being that older individuals are more likely to serve on charitable boards.
	- Further match by comparing cities in addition to provinces.
- Identifying which directors in each market oversee the largest charities by size of their capital reserves/throughput of their gifts to qualified donees.
