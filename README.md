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

- [A. Webscraping Notebook](https://github.com/katerdowdy/virginia_courts/blob/master/A_WebScraping.ipynb)
- [B. Cleaning / EDA Notebook](https://github.com/katerdowdy/virginia_courts/blob/master/B_Cleaning_EDA.ipynb)
- [C. Modeling Notebook](https://github.com/katerdowdy/virginia_courts/blob/master/C_Modeling.ipynb)
- [Demo](https://github.com/katerdowdy/virginia_courts/blob/master/Demo.ipynb)

In addition, I've included [presentation slides](https://github.com/katerdowdy/virginia_courts/blob/master/I%20Fought%20the%20Law_%20Contesting%20Charges%20in%20Virginia%E2%80%99s%20District%20Courts.pdf) from the initial in-class version of this project. To go straight to the final product, I would recommend downloading the [Demo notebook](https://github.com/katerdowdy/virginia_courts/blob/master/Demo.ipynb) (which has an option to toggle on/off the python code at the bottom of the notebook). 

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

## Modeling
Initially, I explored creating statewide classification models using Logistic Regression, Decision Tree, and Random Forest using these features:

- Court
- Gender
- Race
- ChargeType
- HadLawyer
- PublicDefender

My accuracy scores for these models:

- Logistic Regression - Train: 64.8%, Test: 64.8%
- Decision Tree - Train: 66%, Test: 64.8%
- Random Forest - Train: 66.5%, Test: 64.8%

Because there was no significant difference in test scores for the models and the Logistic Regression was the least overfit, I moved forward with the Logistic Regression for further analyses. Confusion matrix metrics for the Logistic Regression:

- True Negatives: 39462
- False Positives: 59458
- False Negatives: 26102
- True Positives: 117951
- Accuracy: 0.6478621081354718
- Misclassification Rate: 0.35213789186452815
- Sensitivity/Recall (True Positive Rate): 0.8188028017465794
- Specificity (True Negative Rate): 0.39892842701172665
- False Positive Rate: 0.41275086252976334
- Precision: 0.6648535305424189

For the purposes of making recommendations to people who are facing charges, there are two main issues with these models I would need to change. First, demographic information shouldn't be a predictor of outcome, so race and gender shouldn't be used as features. Second, I would want to adjust the threshold of predicting 'positive' vs. 'negative' (default is 50%) to minimize the number of false positives and provide a more conservative estimate of outcomes.

I decided that replicating the model at the county-level (and for specific charges) would be a more realistic application for a real-world recommendation system (where people know where and for what they've been charged), and may also eliminate some multicollinearity that's likely present in the court variables and charge variables (ex. number of prior hearings may be  correlated with the seriousness of the offense, if they all pertain to the same charge.) I demo county-level and charge-specific predictive models in the Demo notebook, using these features:

- HadLawyer
- PublicDefender
- PriorHearings
- DefenseAttorney
- Complainant
- HearingPlea

There are some challenges with this approach: a) some counties with a small number of hearings will end up with a larger number of features than rows, if all features after becoming categorical variables are used, and b) it adds another layer to determining the threshold for adjusting the threshold for predicting 'positive' vs. 'negative' to optimize for precision because the classification metrics will be different for each model. However, I did find it interesting that county/charge-level models did in some cases perform significantly better (and may benefit from the inclusion of the lawyer/complainant features.) 

## ANOVA Tests
Even though I didn't use demographic information in the new county-level predictive models, race did play a significant role in my initial exploratory models. I used ANOVA tests to see if there was a statistically significant difference in outcomes based on race (holding the county and charge equal). I did the same for cases where defendants defended themselves, hired private lawyers, or had public defenders. I found that statistical significance varied on a county/charge level - examples can be found in the [Modeling notebook](https://github.com/katerdowdy/virginia_courts/blob/master/C_Modeling.ipynb) and [Demo notebook](https://github.com/katerdowdy/virginia_courts/blob/master/Demo.ipynb). 

## Conclusion & Future Steps

Answering the questions from the problem statement:

> 1. Given data collected on hearings heard in Virginia's District Courts in 2017, how well can we predict the outcome of a case?

It looks like trying to build a model that predicts outcomes across Virginia and all types of infractions is not going to paint the most accurate picture of a defendant's odds. The best state-wide model I built had 65% accuracy on train/test data. On the county-level models, some models scored over 75% on accuracy (more than 20% above baseline) which is interesting. I would say that, depending on the county and the charge, the information on historic caseloads provides some useful information for predicting outcomes. 

> 2. If someone is charged with a crime in Virginia, should they contest the charge? And if so, what factors play the most significant role in determining the outcome of their case?

Yes, for some charges, but it depends on the charge and the court. Infractions and Civil Violations can't be defended by public defenders, so anyone who goes to court for these must defend themselves or hire a lawyer. Some misdemanors require a court hearing while others don't. There seem to be clear differences in how easy some charges are to contest, as well as disparities in how charges are handled across the 134 districts.

> 3. With the data we have, is there evidence to suggest racial bias in the Virginia Courts system?

Yes, the system does seem to hold some racial bias. With a predictive model making recommendations, I wouldn't include race/ethnicity (or gender) in the inputs because it shouldn't be a determinant of outcome, and the adverse effects of discouraging people from contesting (potentially unfair) charges because of historical bias would only help perpetuate that unfair imbalance in the system. The ANOVA tests show that the influence of race on case outcomes differs by county and charge.

> 4. Is it worth it to hire a private lawyer for your defense (and spend hundreds or thousands of dollars), or is a public defender just as effective?

With regards to whether or not it is better to hire a private lawyer or public defender, it also depends on where in Virginia you are and the charge. Fairfax Virginia appears to have slightly better outcomes with their public defenders than Newport News, for example.

There are limitations with the data I used for this project; creating a more accurate model would likely require building a more complex model. There are things that happen in the courtroom that likely (and should) have a significant impact on the outcome of a case, like the disposition of the defendant and the verbal or written case they make on their behalf. To build a highly accurate model without accounting for those human factors would paint a cynical view of the justice system.

With more time, there's certainly room to fine-tune the ChargeType features more and explore the data further, as well as look more closely at how the variables interact in the county-level models for comparison. I think it would be great to make this information and visual representations public (with the caveat that it's not real legal advice) because even the raw data alone would likely be able to help people make decisions about their legal battles, and also illustrate the criminal justice system in a new light for residents of Virginia (and who knows, maybe prosecutors and judges too.)
