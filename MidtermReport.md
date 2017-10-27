# ORIE 4741 Midterm Project Report

## Authors

- Reuben Rappaport (rbr76)
- Celine Brass (cjb57)
- Bridget Cheng (bjc267)

## Introduction
The city of Chicago has witnessed a stunning 9,589 crimes reported thus far in the month of September. This is only the tip of the iceberg when you consider the many crimes which go unreported to begin with. The Chicago police force is tasked with securing the safety of its citizens and yet, of the many crimes reported, only a small fraction will eventually result in an arrest. In examining the efficacy of the police, a natural question to ask is which factors determine whether or not a crime will be solved.

This sort of analysis could lead to insights into deficiencies in policing - perhaps certain areas of the city are systematically under policed or certain types of crime are systematically under investigated. Analyzing the nature and features of crimes committed in the past to identify these biases will allow us to improve the dispatchment of our police force and procedures for handling specific types of crime in the future. Specifically we aim to answer the following question: if a crime of a given type occurs at a given time and location in the city of Chicago, will the police eventually arrest someone for it.


## Feature Analysis

The following features are included in the original data set :

* **ID** : unique identifier for each record
* **Case Number** : The Chicago Police Depart RD (Records Division) Number
* **Date** : the date the incident ~supposedly~ occurred
* **Block** : a partially redacted address where the incident occurred, placing it on the same block as the incident
* **IUCR** : Illinois Uniform Crime Reporting Code. This describes both the primary type of the incident and secondary description. See the full list of IUCR codes here
* **Primary Type** : the primary description of the IUCR code
* **Description** : the secondary description of the IUCR code
* **Location Description** : description of the location where the incident occurred. Ex : STREET, APARTMENT, etc
* **Arrest** : Whether or not an arrest was made. This is the output of our logistic regression model.
* **Domestic** : indicates whether or not the incident was domestic-related
* **Beat** : indicates the beat where the incident occurred. A beat is an area of Chicago assigned to a single patrolling police car. See a full map of the beats of Chicago here
* **District** : indicates the police district where the incident occurred. See a full map of the police districts of Chicago here
* **Ward** : indicates the city council district where the incident occurred. See a full map of the wards of Chicago here
* **Community Area** : indicates in which of Chicago’s 77 community areas the incident occurred. See a full map of the community area of Chicago here
* **FBI Code** : indicates the crime classification as outlined by the FBI’s National Incident-Based Reporting System
* **X Coordinate** : the x coordinate of the location (partially redacted) of the incident
* **Y Coordinate** : the y coordinate of the location (partially redacted) of the incident
* **Year** : the year the incident occurred
* **Updated on** : the date and time the record was last updated in the database
* **Latitude** : the latitude of the partially redacted location of the incident
* **Longitude** : the longitude of the partially redacted location of the incident
* **Location** : the partially redacted location of the incident in a format suitable for mapping

Obviously, we do not want to include all features in our final model. Thus, to get a sense of what features actually affect the arret rate, we made some initial plots as shown below. Below each plot is a brief summary of what it tells us about the relation ship between data.

# Initial Plots #

![Primary Types Over Time](ArrestPredictionGraphs/PrimaryTypesOverTime.png)

![Primary Type By Arrest Rate](ArrestPredictionGraphs/PrimaryTypeByArrestRate.png)

![Hour of Day By Total Arrest Count](ArrestPredictionGraphs/HourByTotalArrests.png)

![Day Of Week By Total Arrest](ArrestPredictionGraphs/DayOfWeekByTotalArrests.png)

![Day Of Month By Total Arrests](ArrestPredictionGraphs/DayOfMonthByTotalArrests.png)

![Month By Total Arrest](ArrestPredictionGraphs/MonthByTotalArrest.png)

![Police District By Average Arrest Rates](ArrestPredictionGraphs/DistrictByAverageArrestRate.png)

![Ward By Total Arrest](ArrestPredictionGraphs/WardByTotalArrest.png)


## Model Selection
A problem with many predictive models is that they apply an absolute prediction to each sample. However it is not the case that a crime of a given type committed at a given time and place will deterministically result in an arrest or not. It is perfectly possible for nearly identical crimes to have different outcomes. Based on this structure we wanted to predict the _probability_ that a crime with certain features would lead to an arrest. A natural model to perform this translation between binary outcomes and statistical probabilities is the Logistic Regression so that is what we chose.

For a Logistic Regression all features must be independent binary numeric variables. This influenced our choices during the data cleaning and feature engineering stage of model construction.

## Feature Engineering and Data Cleaning
Using the initial data visualizations, we were able to identify which features we believed would be informative to our model. Of the original 22 features present, we chose to completely discard the following features from the dataset: ID, Case Number, IUCR, Block, Beat, Ward, FBI Code, Updated On, X_Coordinate, Y_Coordinate, Longitude, Latitude, and Location.

The ID, Case Number, and Updated On features provide no descriptive information about the crime, so they were dropped. Because the IUCR is a numeric encoding of both the Primary Type and Description features, we decided to remove the single IUCR feature in favor of the two distinct categorical features. Since Block, X Coordinate, Y Coordinate, Longitude, Latitude, and Location values are extremely specific location-based values, (there were upwards of 849,571 unique values for Location), it is unlikely that a model would be able to draw useful information from these features. We predicted that because of the variable nature of crime and policing, the beats of Chicago would have shifted over the years. Because of this instability, we felt it would be inappropriate to base our model on the Beat feature. The Ward feature neither reflects the social and demographic features nor the policing activity of an area, so we chose to drop this as well.

This left us with the following features: Primary Type, Description, Date, Location Description, Arrest, Domestic, Community Area, and District. Both Community Area and District encode geographic location, so to maintain independence of features in our preliminary model, we chose to work only with District and drop Community Area. We will explore the effects of using Community Area in future models, as described in the Going Forward section.

Of these remaining features, Arrest and Domestic were already binary valued, and did not require any feature engineering. The boolean true and false values of the Arrest and Domestic features were simply transformed to be binary 0 and 1 values.

To address the Date feature, the first step we took was to segment the string value into two new features, Hour and Month. We chose to disregard the Day of Month and Day of Week features initially used in the visualizations, as we saw the distribution of arrests was almost uniform over days of the month and days of the week. Furthermore, because representing hour and month as integers does not represent the cyclic nature of these values, we cleverly applied a sine and cosine transformation function to the values, so that the closeness of values like 0 and 23 in the hours of a day is captured despite a large numerical difference. This introduced four new features, Sin Hour, Cos Hour, Sin Month, and Cos Month. We removed the original Date feature as a final step.

Primary Type, Description, Location Description, and District were categorical features and needed processing to be represented in the dataset as quantitative binary values. To do this, we utilized one-hot encodings for each value that the categorical feature took on. This process, known as creating dummy variables, indicates the presence or absence of a categorical effect. For example, for each of the possibly Primary Type values, “Robbery”, “Prostitution”, “Ritualism”, etc., a new feature “Primary_Type_Robbery”, “Primary_Type_Prostitution”, “Primary_Type_Ritualism”, etc. was introduced with a value of 0 or 1 depending on the value of the original Primary Type feature. After the introduction of these dummy variable features, we removed the original Primary Type, Description, Location Description, and District features as a final step.
To limit the size of our data, we decided to use only reports that have taken place in the last 5 years--this left only crimes reported between the years of 2013-present day. Furthermore, we noticed from our visualizations that crimes of particular Primary Type values had arrest rates of nearly 1--we immediately attributed this to the fact that certain crimes would only be reported if an arrest were definitely going to be made, for example, liquor law violations or public indecency violations. We manually inspected the various Primary Type values, and ultimately chose to remove crimes with Primary Type of Gambling, Liquor Law Violation, Prostitution, Narcotics, and Public Indecency.
After this feature engineering and data processing, we were left with 9,324,840 data samples and 469 features.

## Preliminary Results
After cleaning up the dataset and running all the feature transformations we split the data 80:20 and trained a logistic regression model. Our preliminary results were quite encouraging!
```
              precision     recall    f1-score       support
          0        0.89       0.98        0.93        154420
          1        0.80       0.41        0.54         32077
avg / total        0.87       0.88        0.86        186497
```
Overall the model was quite accurate with a high average score. Its precision and recall were excellent on negative results. However due to the dearth of positive results in the sample (there are almost five times as many negative results as positive ones!) the classifier has extremely poor recall on positive examples. This is something of a fundamental structural problem in the dataset since the majority of crime reports do not end with an arrest. We’ll need to take steps to address it going forward.

## Going Forward
Although the model we trained did very well on negative examples, it had poor recall on positive examples due to their serious underrepresentation in the dataset. Our next step is going to be to look into more sophisticated sampling techniques we could use to counteract this. We’re thinking of possibly using stratified sampling to place an outsized emphasis on positive samples relative to their actual incidence.
