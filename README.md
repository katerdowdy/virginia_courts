# I Fought the Law: Predicting Outcomes in Virginia's District Courts

## Executive Summary
Each year, millions of hearings are scheduled for Virginia's District Criminal and Traffic Courts, which process charges varying from minor traffic violations to preliminary hearings for serious felonies. For those facing a ticket or charge from the State of Virginia, there is no easy access to data on how others in their shoes have been prosecuted by the state, and how and why defendants are able to overturn or minimize similar charges. A quick search of "should I contest my ticket?" or "should I hire a lawyer?" on the internet returns numerous advice columns and posts from lawyers, many of whom are actively seeking new clients and don't provide concrete data on win rates for their firms or the charges their clients are facing. 

The information imbalance between the State and citizens going through the criminal justice process sets the system up to favor those with the resources to hire seasoned and informed lawyers, and penalize those who can't afford them. For this project, I sought to use historic caseload information from the Virginia Courts database to create data-backed recommendations on defense strategies for people contesting charges in Virginia's 134 districts.

## Problem Statement
> 1. Given data collected on hearings heard in Virginia's District Courts in 2017, how well can we predict the outcome of a case?
> 2. If someone is charged with a crime in Virginia, should they contest the charge? And if so, what factors play the most significant role in determining the outcome of their case?
> 3. With the data we have, is there evidence to suggest racial bias in the Virginia Courts system?
> 4. Is it worth it to hire a private lawyer for your defense (and spend hundreds or thousands of dollars), or is a public defender just as effective?

## Contents of This Repo
The repo contains these main files:

- A. Webscraping Notebook
- B. Cleaning / EDA Notebook
- C. Modeling Notebook
- Demo

In addition, I've included presentation slides from the initial in-class version of this project, as well as a notebook outlining the process for creating summary tables I used for the Plotly visualizations. To go straight to the final product, I would recommend the Demo notebook (which has an option to toggle on/off the python code at the bottom of the notebook). 

## Data Sources
Data for this project came from the [Virginia Courts website](http://www.courts.state.va.us/courts/gd/home.html). While I initially wrote and ran a script to scrape records from the website using BeautifulSoup and Selenium, this was a time-consuming process (it took days to scrape one weeks' worth of records.) I ultimately came across software engineer Ben Schoenfeld's [Virginia Court Data site](http://virginiacourtdata.org/) which provides full scraped records from 2010-2017. 

I chose to use Ben's data on just the 2017 Criminal and Traffic Court hearings, which totaled over 2.1 million rows and 42 columns. Features from the original data that I used for my analysis (either as features, to filter the data, or engineer new features):

- *AmendedCharge:* the final charge (if it was changed from the original)
- *CaseType:* type of case (infraction, misdemeanor, felony, etc.)
- *Charge:* the written description of charges brought against the defendant
- *Complainant:* the officer, judge, person or court that brought the charge
- *DefenseAttorney:* the name of the private attorney (or labeled 'public defender' where applicable)
- *FinalDisposition:* the final outcome of the hearing
- *fips:* the fips code associated with the Court district/city
- *Gender:* labels defendants as male or female
- *HearingDate:* the date of the (final) hearing in the case
- *HearingResult:* the result of the hearing (finalized or continuing)
- *HearingPlea:* defendant plea (where applicable - many rows are empty)
- *OffenseDate:* date of the offense
- *person_id:* IDs generated based on names and birthdates that are present in the Virginia Courts database
- *Race:* race/ethnicity (as coded/categorized by the State of Virginia)

There are other features in the data that would be interesting to analyze further (for example: sentencing time, fines, costs, license suspensions) that did not play a role in addressing my problem statement, so they were not used.

## Feature Engineering
To answer the problem statement, I created new features/filters:

- *Outcome_Positive:* If a case was dismissed, not prosecuted, or the defendant was found not guilty, the value was '1' (otherwise, 0)
- *Total_Positive:* If a case either had a 'positive outcome' for the defense or an amended charge, this field was '1' (otherwise, 0)
- *Contested:* If the status of the case was anything other than 'prepaid' or 'guilty in absentia', this field was '1' to represent a defendant who went to court (otherwise, 0)
- *Had_Lawyer:* If the 'DefenseAttorney' field was not missing values, this field was '1'
- *Public_Defender:* If the 'DefenseAttorney' field contained any values referring to a Public Defender, this field was '1'
- *Prior Hearings:* Number of hearings the individual had prior to the current hearing in 2017

The original dataset had 75,000 unique charges (text fields describing the offense) and 5,400+ unique codes associated with those charges. I initially planned to pare down the charges based on their codes, but after investigating some of the more prevalent charges and finding inconsistent/non-existent codes (via the [Virginia Criminal Code](https://law.lis.virginia.gov/vacode) lookup), I decided to cluster the charges using topic modeling through LDA. I was able to categorize the original charges for Infractions, Misdemeanors, Felonies and Civil Violations into 59 topics (Charge Types.)





