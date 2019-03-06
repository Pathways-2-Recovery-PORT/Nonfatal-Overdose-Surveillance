# Nonfatal-Overdose-Surveillance
 Code for performing space-time cluster analysis of non-fatal overdoses and e-mailing the results.  Project SOON was funded by the Council of State and Territorial Epidemiologists (CSTE).

## Nonfatal-Overdose-Surveillance: Project SOON
Python, R code, and optional ArcGIS Toolbox/ModelBuilder Model for performing space-time cluster analysis of non-fatal overdoses and e-mailing the results. Public Health Surveillance of Opioid Overdose and Notification (Project SOON) was funded by the Council of State and Territorial Epidemiologists (CSTE):https://www.cste.org/.

## Automation of Non-fatal Overdose Surveillance and Notification
Baltimore City had 761 drug and alcohol overdose deaths in 2017, a continuing epidemic and public health emergency. Nationally, more than 70,000 lives were lost in 2017 to drug overdoses.

The goal of this project is to automate (1) the process of conducting non-fatal overdose surveillance and (2) the notification of non-fatal overdose “spikes” (i.e., clusters of non-fatal overdoses in space and time) to stakeholders. The project uses data generated by EMS (Emergency Medical Services) through its provision of services to patients. The project is based at the Baltimore City Health Department and is conducted in close partnership with the Baltimore City Fire Department Emergency Medical Services in Baltimore, Maryland, United States. This work was funded by the Council of State and Territorial Epidemiologists in 2017-2018 as Project SOON (Surveillance of Opioid Overdose and Notification). Questions regarding the project may be directed to the Baltimore City Health Department at health_research@baltimorecity.gov and +1-410-361-9580.

## Getting Started
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

Since development environments and software settings can vary, this code comes with no guarantees.

## Prerequisites
A non-fatal overdose is defined as improvement in an EMS patient’s condition after EMS’s administration of naloxone to the patient.

An overdose spike is defined as a cluster of non-fatal overdoses in space and time with a p-value of less than 0.05 as determined by SaTScan analysis. Our non-reporting of a cluster has a cutoff of 0.4. Given the small numbers of incidents involved, clusters that are not statistically significant can still be meaningful for outreach activities. Other clusters that do not reach statistical significance may be meaningful based on the number of patients, geographic size, and temporal window. Cutoffs for jurisdictions will vary based on their population size and extent of the overdose problem and geographic scope (local, state, or regional).

The below software and resources are needed for replication of this project.

-	ArcGIS Desktop version 10.6 
-	SaTScan version 9.6 
	SaTScan settings are set in the R script via the rsatscan package
-	Python version 2.7 (Note: ArcGIS still uses Python 2 while Python 3 has become more of the standard.)  Some Python packages are already included with ArcGIS Python installation. Others can be downloaded using a Python Package Manager (e.g. pip, Anaconda, etc.).
-	R version 3.5
-	R studio version 1.1.447
-	Data: Fake/dummy data in CSV format of non-fatal overdose incidents is included. The addresses were derived from a US Census file for landmarks in Baltimore City.  Demographics and other included information is purely fictitious.  Each row of data should represent one non-fatal overdose incident.  Required variables are named: Incident Date [MM/DD/YYYY, this is the date the non-fatal overdose incident occurred], Incident Number [this is a unique identifier for a given non-fatal overdose incident, which may involve more than one non-fatal overdose patient], and Incident Address [this is the non-fatal overdose incident location, which will be locally geocoded].  Geocoding occurs locally (vs. in the cloud) because the data are protected under HIPAA.  We used these optional variables for exploratory analyses that are not described here: Patient Age, Patient Gender, Patient Race, Primary Impression, Times: Arrived on Scene Time, Destination Code, Medication Administered, Medication Response to Med, Medication Dosage, Destination Patient Disposition.

Although the code could run off a desktop computer, we recommend using a server. We use Windows Server 2012 R2.  

## Deploying/Installing

Download and copy files to a folder on a server or desktop computer.  We used Microsoft Server 2012 R2 with 8 GB of RAM and an Intel Xeon E5-2623 processor. Some settings in SaTScan can dramatically increase (or decrease) the computation time needed.  

IMPORTANT: Not all of the code we included is needed.  If your data are already geocoded or accessible through a database connection, you may just want to use the R script and R Markdown, files for example.  We spatially joined data to multiple geographies to prevent having to go back and do so retrospectively.  This was done in the event we used risk-based/population models using discrete geographic units and for other non-fatal overdose reporting.

## Overview of the Process

Outlook E-mail Received with Non-fatal Overdose Data->Python script for high-level data cleaning (some cleaning is also done later in R) -> rsatscan package in R calls SaTScan on Windows to perform space-time surveillance-> R is then used to create a report and e-mail the results using R and Outlook. Ideally, a database connection would be used instead.

You will have to change the directories for where you download the daily Excel file with non-fatal overdose data and the geodatabases and folders where your data will be stored.  The process is initiated when a file is received in Outlook e-mail.  Windows Task Scheduler is set to periodically check for an e-mail with the subject line “Daily Naloxone Report.” It can be set to run once a day or multiple times in a day. A Python Script is run that downloads the daily e-mail and saves the Excel file as Daily-Naloxone-Data_YYYY-MM-DD.xlsx.  Next the program opens the file and re-saves without a password—without intervention.  Then the script runs Geocoding and spatial joins (ZIP codes, etc.) eventually outputting the data into a geodatabase.
From this geodatabase, data is further formatted using R, and then SaTScan, a windows-based program for space-time detection, is run via the rsatscan R package to perform space-time surveillance for clusters of non-fatal overdoses. SaTScan utilizes the prospective space-time permutation method to detect clusters, and reports and figures are then generated in R based on these results.  Since cases are rare, the general population distribution of Baltimore is not considered in this analysis.   

For Baltimore City, the parameters used were the following. You may have to adjust these based on what you observe in your jurisdiction.

Several other packages are used in the reporting process. Tables and figures based on identified clusters are formatted into automatically generated reports via R Markdown, which are then automatically e-mailed to appropriate audiences using R in combination with outlook.  The process is repeated each day and could also be setup to check every hour or as an e-mail is received.

The space-time permutation method uses point data and circles for clusters instead of census tracts. Spikes are determined using a Prospective Scan Statistic meaning they must still be an active cluster on the day the model is run.  This is why getting daily data is important.  The cluster criteria/model parameters we use are below:
•	Prospective Scan Statistic – must be active on date of most recent data.
•	Clusters must have at least 3 cases
•	Unit of analysis = days
•	Min Temporal Size = 1 day
•	Max temporal Size = 5 days to 7 days/week
•	Max spatial cluster size = 0.75 
o	Your mileage may vary: if you are doing local or regional (county) surveillance (larger)
•	(No geographical overlap can also be important)
•	Since non-fatal overdoses on a given day and location are very small, we ignore population size and use the space-time permutation model.

Below is a step-by-step series of examples that tell you how to get a development environment running.

Create an instance of Windows Server 2012 R2.  Install the necessary software and associated packages/libraries detailed earlier in the “Pre-requisite Section.” Make sure to change file directories and names to match your working environment in the Python and R Code.

IMPORTANT:  You will have to change pathnames in the files to match your operating system.

### Create a Folder called “Production” 
- Place the DayBehindFix.pyw in it. This file controls the two scripts listed below.  Use Windows Task Scheduler to run this script.
- Place the OverdosePython.pyw Python script in the Production folder
- We’ve included a second one OverdosePython_DayBehind that allows for catch-up geoprocessing if a day’s data is missed

Create a Folder called “Daily Narcan Data”
- This is where downloaded EMS data will go

OPTIONAL:  Create a Folder called Locator
- Place your Address Locator here for geocoding 

Create a Folder called “Logs”
- Python logs will appear here and show what part in the process completed

Create a folder called “satscan_wd”
- This is what R will use to temporarily run SaTScan each day.

Create a folder called “Workspace”
- Place the run_reports.R (for Windows Task Scheduler), outreach_report.RMD, epi_report.RMD, run_report_bad_batch.R (For Windows Task Scheduler), and bad_batch.RMD in this folder.

#### Create subfolders called:
Bad Batch: Stores PDFs that were used to notify a public messaging service
Outreach: Stores Outreach PDF Reports created each day, intended for the outreach team (no point locations)
Epidemiology: Stores Epidemiology PDF Reports created eatch day. Epidemiology reports contain all the data that the Outreach reports contain, but additionally include point locations of the overdoses, demographic statistics for each cluster, and a summary graduated circles plot of cases over the previous 30 days. 

Reference Data
- Add Baltimore_Census_Tracts_Project.shp 
- Add Baltimore_coordinates.feet.csv
- Add Vital_Signs_16_Census_Demographics.shp
- Add Bad_Batch_Regions_Revised.shp

Create a Folder called “Historical” 
- Place the Historical_Clusters.csv file in it. Can be used in R, ArcGIS, QGIS, etc.
- There is a also a historical_clusters.R file for running analysis

The R Task Scheduler Setup.R can help you setup the R scripts in Windows Task Scheduler.

In ArcCatalog or ArcMap, create a geodatabase called “Master.gdb.” We recommend backing up this geodatabase periodically.

Place the sample data created from US Census data landmarks but formatted to like EMS data in an e-mail or “Daily Narcan Data” Folder and try running the scripts. If successful: PDFs will be generated showing clusters.

Review the R scripts and Markdown files.  Change the start dates to reflect the start dates in the sample data.

At the end of the project, several additional features were added to scripts:
•	Sending e-mails only when clusters were identified to certain groups
•	Performing a ‘catch-up’ of geocoding if a daily e-mail was sent late.

REMINDER:  You will have to change pathnames in the scripts to match your operating system.

## Contributing

When contributing to this repository, please first discuss the change you wish to make via issue, email, or any other method with the owners of this repository before making a change. 

Please note we have a code of conduct, please follow it in all your interactions with the project.

## Pull Request Process

1. Ensure any install or build dependencies are removed before the end of the layer when doing a build.
2. Update the README.md with details of changes to the interface, this includes new environment variables, exposed ports, useful file locations and container parameters.
3. Increase the version numbers in any examples files and the README.md to the new version that this Pull Request would represent. The versioning scheme we use is [SemVer](http://semver.org/).
4. You may merge the Pull Request in once you have the sign-off of the Baltimore City Health Department.

## Code of Conduct

### Our Pledge

In the interest of fostering an open and welcoming environment, we as contributors and maintainers pledge to making participation in our project and our community a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

Examples of behavior that contributes to creating a positive environment include:

* Using welcoming and inclusive language
* Being respectful of differing viewpoints and experiences
* Gracefully accepting constructive criticism
* Focusing on what is best for the community
* Showing empathy towards other community members

Examples of unacceptable behavior by participants include:

* The use of sexualized language or imagery and unwelcome sexual attention or advances
* Trolling, insulting/derogatory comments, and personal or political attacks
* Public or private harassment
* Publishing others' private information, such as a physical or electronic   address, without explicit permission
* Other conduct which could reasonably be considered inappropriate in a   professional setting

### Our Responsibilities

Project maintainers are responsible for clarifying the standards of acceptable behavior and are expected to take appropriate and fair corrective action in response to any instances of unacceptable behavior.

Project maintainers have the right and responsibility to remove, edit, or reject comments, commits, code, wiki edits, issues, and other contributions that are not aligned to this Code of Conduct, or to ban temporarily or permanently any contributor for other behaviors that they deem inappropriate, threatening, offensive, or harmful.

### Scope

This Code of Conduct applies both within project spaces and in public spaces when an individual is representing the project or its community. Examples of representing a project or community include using an official project e-mail address, posting via an official social media account, or acting as an appointed representative at an online or offline event. Representation of a project may be further defined and clarified by project maintainers.

### Enforcement

Instances of abusive, harassing, or otherwise unacceptable behavior may be reported by contacting the project team at health_research@baltimorecity.gov. All complaints will be reviewed and investigated and will result in a response that is deemed necessary and appropriate to the circumstances. The project team is obligated to maintain confidentiality with regard to the reporter of an incident.
Further details of specific enforcement policies may be posted separately.

Project maintainers who do not follow or enforce the Code of Conduct in good faith may face temporary or permanent repercussions as determined by other members of the project's leadership.

### Attribution

This Code of Conduct is adapted from the [Contributor Covenant][homepage], version 1.4, available at [http://contributor-covenant.org/version/1/4][version]

[homepage]: http://contributor-covenant.org
[version]: http://contributor-covenant.org/version/1/4/

## Versioning

We use [SemVer](http://semver.org/) for versioning. 

## Authors

Baltimore City Health Department (BCHD)

Contributors:  Jonathan Gross (BCHD) and Anton Kvit (Johns Hopkins Bloomberg School of Public Health)
 
## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* This work was funded by the Council of State and Territorial Epidemiologists (CSTE) as Project SOON (Surveillance of Opioid Overdose and Notification) in 2017-2018.

* Team members include Jonathan Gross (BCHD), Darcy Phelan-Emrick (BCHD), Mike Fried (BCHD), Frank Curriero (Johns Hopkins Bloomberg School of Public Health (JHBSPH)), Anton Kvit (JHBSPH), Tim Shields (JHBSPH), Amanda Latimore (formerly Behavioral Health System Baltimore), Kelly King (Baltimore City Fire Department (BCFD) EMS), Bryan Johnson (BCFD EMS), James Matz (BCFD EMS), William McCarren (BCFD EMS), Code in the Schools “Bad Batch” team, led by Mike LeGrand.

* Thank you to PurpleBooth for the README.md template and Good-CONTRIBUTING.md-template.md template. (https://gist.github.com/PurpleBooth)
© 2019 GitHub, Inc.
