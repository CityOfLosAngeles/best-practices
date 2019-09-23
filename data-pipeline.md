# Data Pipelines 

Having good practices around data pipelines help accomplish a number of goals in a data science team:
1. They help with project reproducibility.
Without reproducibility, an analysis project is difficult to trust,
and challenging to share and transfer between different people and teams.
2. They help with efficiency.
A good data pipeline can make it so time-consuming steps are only run as often as they need to be.
3. They help with documentation.
Good data practices involve documenting the data sources that are used and how they are loaded into an analysis environment.
3. They help with security.
Establishing good practices with data ingest can prevent leaking of authentication tokens and passwords.

The following are some tools that can help with establishing a robust data pipeline:

* [Makefiles](#makefiles)
* [Catalogs](#catalogs)
  * [Open Data Portals](#open-data-portals)
  * [Intake Catalogs](#intake-catalogs)
* [Version Control](#version-control)
* [Credentials](#credentials)


## Makefiles

Makefiles provide a plain-text way to describe data pipelines that run locally.
They describe a series of rules that tell the computer how to execute commands to produce target files.
These rules can declare other rules as dependencies, allowing `make` to only
run the commands necessary to produce a given target.

Consider an analysis project that has separate scripts for downloading
a series of data files, converting them to a new format, cleaning them,
and then inserting them in a database.
Each of these tasks might take minutes to hours,
and rely on the results of the tasks that come before them.
This data pipeline may be a good target to describe with a Makefile,
which might look something like

```bash
.PHONY clean insert

raw_data.csv:
  wget https://data-source.org/raw_data.csv

cleaned_data.csv: raw_data.csv
  python clean_data.py --input raw_data.csv --output cleaned_data.csv

insert: cleaned_data.csv
  python insert_data_in_database.py cleaned_data.csv

clean:
  rm *.csv
```
This makefile provides rules for how to produce the `raw_data.csv` and `cleaned_data.csv` files,
as well as rules for inserting the cleaned data into a database and cleaning up after ourselves.
A shorthand for producing all the data and inserting it can thus be run by invoking `make insert`. If the user wants to just clean the data without inserting it into a database, they can run `make cleaned_data.csv`.

More reading on why Makefiles are useful and best-practice guides for using them may be found here:

* [Datamade Guide](https://github.com/datamade/data-making-guidelines)
* [Why use Make](https://bost.ocks.org/mike/make/)
* [Style Guide for Make](http://clarkgrubb.com/makefile-style-guide#data-workflows)

## Catalogs

One major difficulty with conducting reproducible analyses is the location of data.
If a data scientist downloads a CSV on their local system,
but does not document its provenance or access,
the analysis becomes very difficult to reproduce.

One strategy to deal with this is to create data catalogs for projects,
which describe the data sources used and how to access them.

### **Open Data Portals**

Open data portals (such as the LA GeoHub) usually provide a
[DCAT](https://www.w3.org/TR/vocab-dcat) catalog for their datasets,
including links for downloading them and metadata describing them.

Many civic data analysis projects end up using these open datasets.
When they do, it should be clearly documented.

### **Intake Catalogs**

Data scientists tend to load their data from many heterogeneous sources (Databases, CSVs, JSON, etc),
but at the end of the day, they often end up with the data in dataframes
or numpy arrays.
One tool for managing that in Python is the relatively new project
[Intake](https://intake.readthedocs.io/en/latest/).
Intake provides a way to make data catalogs can then be used load sources
into dataframes and arrays.
These catalogs are plain text and can then be versioned and published,
allowing for more ergonomic documentation of the data sources used for a project.

[`intake-dcat`](https://github.com/CityOfLosAngeles/intake-dcat)
is a tool for allowing intake to more easily interact with DCAT catalogs.

## Version Control

If a dataset is relatively small (of order a few megabytes) and not changing too often,
then it may be appropriate to include it in a code repository.
However, it is important to document in that same repository where the data was downloaded, and any metadata that is important to the analysis.

## Credentials

Sometimes if data access is expensive, or if there is sensitive data,
then accessing it will require some sort of credentials
(which may take the form of passwords or tokens).

There is a fundamental tension between data access restrictions and analysis reproducibility.
If credentials are required, then an analysis is not reproducible out-of-the-box.
However, including these credentials in scripts and notebooks is a security risk.

Most projects should store the authentication credentials in environment variables,
which can then be read by scripts and notebooks.
The environment variables that are required for an analysis to work
*should be clearly documented*.
If you are using a Dockerized setup, you may want to consider making the build 
[fail](https://docs.docker.com/compose/compose-file/#variable-substitution)
if the required variables are not present.
