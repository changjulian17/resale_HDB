# QUANTITATIVE STRATEGY CASE INTERVIEW

# Introduction

The dataset for this case interview would be the HDB (Housing Development Board) resale property transactions, using open data from data.gov.sg. Answering all questions in a deck of PowerPoint slides.

This presentation assumes that I am an analyst at HDB, and that the series of questions below come from senior management. During your presentation, assume that you are presenting to HDB senior management.
The presentation will last for 1 hour, including time for Q&A.


## Getting to know the dataset
The HDB resale data from 1990 to mid Sep 2021 can be retrieved from data.gov.sg. Data is collected from data.gov.sg with an API seen in code: [`hdb_project`]('hdb_project.csv'). Below is an example of the columns available from the [`HDB_resale.csv`]('HDB_resale.csv') dataset:

|Feature| Description |
|--|--|
| Month | Given in the format of year-month. We may retrieve the year data from this column, which may be useful when analysing the time trend for HDB resale price. |
| Town | Town location should be one of the key factors affecting HDB resale price — we are generally expecting an HDB flat in Orchard has a much higher resale price than Yishun given the same flat type. |
|Flat Type| There are 7 different kinds of flat types: 1 Room, 2 Room, 3 Room, 4 Room, 5 Room, EC and Multi-generation. Among which the 4 Room HDB flats are the most popular ones in Singapore. We may consider using 4 Room data samples to construct the model. |
|Storey Range| This column is given as a string rather than numbers, we may need to do some data munging accordingly if we want to use it to build the model. |
|Flat Model| Similarly, there are plenty of different flat models out there(35 different types). This factor would play an important role in the overall flat price. E.g., the DBSS (Design, Build and Sell Scheme) flats would have a higher resale price considering it allows buyers to design the HDB based on their own style. |
|Remaining Lease| Singapore HDB has a lease of 99 years. This column data has quite some NULL values, and it is calculated based on different years. We may need to adjust this column data accordingly when building the model.|

# Section 1: Data Visualisation

Reference to `HDB_project.ipynb` 

HDB senior management is thinking of creating dashboards to equip potential buyers of HDB resale flats with the necessary information to make an informed decision. 
Created dashboard is at: [Dashboard](public.tableau.com/app/profile/julian.chang/viz/FirstDashboard_16315416189430)

#### Question A: Show an overview of the number of property transactions, median price across the years. Provide both a view at the national level, as well as by HDB towns. Your dashboard should also provide functionality to filter based on Flat Type (e.g. look at only 5 Room flats).

In tab `HDB_Res_nat` , the trend over the years for the national median resale prices is plotted along side the number of transactions. Then in tab `HDB_Res_town` another tab shows the trend for each town. Both allow for a slider to choose the the view with the year slider. For reference, the median price per sqm is also included to normalise over the size of houses for fair comparison. Users can pick or search the flat type, town parameters to observe.


#### Question B: Some buyers would want to get the largest flat possible within a given budget. Create
a dashboard to allow potential buyers to input their budget, and then suggest towns where such flats exist based on historical transactions.

In tab `HDB_Res_Budget` provides a helpful bubble graphic. Based on historical resale prices and recommends the largest apartment in each town. Its is clear from the graphic which town will have the largest apartment within the budget. 

#### Question C: There are buyers who would like to optimise given the proximity of a flat to important locations in the neighbourhood, such as the nearest MRT station. Create a dashboard that allows buyers to: i) input their budget; and ii) optimise flat selection given distance to important locations around the neighbourhood.

Based on the above list of HDB the address field is summarised. Then a list MRT stations and malls in Singapore is taken from Wikipedia.
Using OneMap API the list of longitude and latitude coordinates are collated.
Due to the long wait time, only Toa Payoh neighbourhood is extracted. Future optimisation to extract the geocode can be developed in future to optimise data mining.

# Section 2: Data Modeling

HDB wants to know if a resale flat transaction fits market expectations. Your task is to create statistical models to answer the following questions.

All models are scaled with RobustScaler()

#### Question A*: Predict a resale flat price’s transaction price in 2014. Use the following characteristics: flat type, flat age and town. Propose and implement a minimum of three models, select the best model, and explain the reasons for your choice.

remaining_lease is used instead of flat_age because it has a positive correlation so it is slightly easier to explain.

A regression model is created and trained on a portion of the available data trials performed on OLS, Lasso and Random Forest. The best model chosen based on test Cross-Validation root mean squared error, then the test score is used as reference. Random Forest is chosen with a root mean square error of $18,759. Model used the following parameters: 

- *RandomForestRegressor(max_depth=100, max_features='log2', max_samples=0.5,
                      n_estimators=400, random_state=42, warm_start=True)*

**_Random Forest is chosen as the best model._**
 
_Reasons_

-   bagging aggregates a close estimate of the relationships between coefficients
-   random subset selection. although there are only 3 main features, there is localised relationships for example between towns

_Other Notes_

- OLS is set as the baseline, because it is the most inclusive but is not the most flexible either.  
- remaining_lease and resale_price is scaled to make a normal distribution
- if there is more time polynomial relationships could be added Lasso is used because of regularisation to reduce coefficients although it did not affect performance and all the coefficients were found to be significant even though this method is considered to be aggressive.  
  - alpha is automatically chosen

#### Question B**: A flat was sold in Nov 2017 with the following characteristics. Was this a reasonable price for the transaction? How confident are you in your assessment?

- Flat type: 4 ROOM

- Town: Yishun

- Flat Model: New Generation

- Storey Range: 10 to 12

- Floor Area (sqm): 91

- Lease Commence Date: 1984

- Resale Price: 550,800

##### Initial analysis
If plot the houses transacted for similar

1.  year price
2.  size of home
3.  age of the home
4.  town

The price paid for the house is in excess of the sample expectations. However this is still a small sample and there is a large data set to make a model and see if this transactions follows the expectation of the transacted prices.

_Feature Engineering_
- As discussed during EDA portion there is an outlier, we will impute the correct value with commencement date and 99 year lease.
- Drop the following columns because they are dependent or highly correlated on the remainder of the columns:  
'month','storey_range','block','storey_lower','storey_upper','flat_age','address'
- remaining_lease and resale_price is scaled to make a normal distribution

_model_
Similar model and params from previous question is used. This model uses an ensemble with random feature subset selection and Bagging  models voting, 
Use RF with initial model from Section 2.A but GridSearch is not used because of the length of time to get the result

***The model provided is targeted at ~ 458,000 with a RMSE of 16,975. Meaning that the model expects around a maximum price of around 474,857. so it would be unreasonable to pay 550,800 for a 4 ROOM HDB in Yishun. Provided that it is an MSE and regression was used here and all assumptions hold, there should be a 95% confidence that this prediction is reasonable.***

With regard to feature importance, we see that transact year plays a big role and size of the apartment plays an outsized role in the price. And based on the the initial analysis the proposed resale_price would not be expected based on historical prices

Predictions for lower prices the is a positive bias and the opposite for the higher prices. There may be polynomial terms that is missed in the model which was not added in this case.

#### Question C: Someone mistakenly deleted the column containing data on Flat Type in the database. While backups exist, these data are critical to HDB’s daily operations, and time would be needed to restore these data from the backup. Senior management would like you to create a model to predict flat type given a transaction’s other characteristics. Explain the reasons for choosing this model.

##### Initial Analysis
- there is imbalance in the distribution of flat types. There is high imbalance in the classifications

-   4 Room is the highest at 38%
-   3 room is the next highest at 33%
-   3 to 5 Room take up >90% of classifications

Will run analysis without over or synthetic sampling first. But will take note of accuracy and establish a baseline for assessment. No further manipulation of the model will be done but the consideration of each classification will be considered

 _Feature Engineering_
A subset of features from those available will be considered:
-   flat_model
-   resale_price
-   flat_size_sqm
-   storey_ave
-   town

Other features such as block do not have a clear determining relation to flat type so they are taken out to 
avoid overfitting and computation time.

_model_
Two model approaches are taken:
1. supervised learning model : 	DecisionTreeClassifier(random_state=42)
2. unsupervides learning model: 	KMeans(n_clusters=3, random_state=42)
	- cluster of 4 (less than the 7 flat types) is  chosen because the performance with k = 7 is poor. Due to the small sample of 1,2 Room, Executive and Multi Generation HDB  they should be combined with the remaining categories.

1. Cross Validated accuracy is about 99% which is still greater than the 3 to 5 Room classifications which take up >90% of classifications so the model is effective
2.  As seen there is a good silhouette score at k =3 likely due to 3room grouped with 1,2room and 5room grouped with exec,multi generation. it is possible to perform unsupervised grouping but as the model coerces the divisions the accuracy decreases. As seen in the Silhouette Score over k graph after k = 3

# Section 3: Policy Analysis

HDB is considering and reviewing a slew of policies, and wants to adopt an evidence-based approach to evaluate these policies.

#### Question A*: Yishun has received a negative reputation as “Crazy Town”, and property prices might have been impacted. Are Yishun flats the cheapest in the country?

#### Question B*: Some members of public have been saying that flat sizes have gotten smaller over the years. Is there any truth in this statement?

#### Question C**: The Downtown Line Stage 2 connects the Bukit Panjang heartland to the city. Have prices increased for resale flats in the towns served by this Line? You might want to use a difference in-differences model for this task.

#### Question D***: There have been comments online that people are buying flats in towns further from the city so that the cost savings can be used for a car. Are resale prices in HDB estates in areas further away from the city (i.e. Sengkang and Punggol) impacted by Certificate of Entitlement (COE) prices for cars?

END
