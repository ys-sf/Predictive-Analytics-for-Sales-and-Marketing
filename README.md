# Predictive analysis for sales and marketing

## Project title: A complete analysis of the chronological series for the dynamic sales forecast

### Presentation of the project
- Sales forecasting represents a major challenge for companies, due to factors such as lack of historical data, the unpredictability of external variables such as weather, natural phenomena, government regulations and, above all, the dynamic nature of the market.
-This project aims to implement various sales forecasting techniques at Olist, a Brazilian E-commerce start-up.
- The project includes two notepads:
1. "Data_Cleaning" serves as an initial notepad, loading the raw data, performing cleaning procedures and merging all tables in the main dataset.
2. "Time_series" is the following notepad, where various modeling techniques for the forecast of the chronological series have been explored.

### Data description
The data set obtained from [Kaggle] presents a public collection of Brazilian e-commerce commands carried out on Olist Store.
- Covering 100,000 orders between 2016 and 2018, these transactions were made in different marketplaces in Brazil.
- The data set offers a complete overview of each order, including aspects such as order status, price, payment and transport performance, customer location, product characteristics and customer reviews.
- In addition, a geolocation data set connecting Brazilian postal codes to the corresponding latitude and longitude coordinates was shared.

### PIPELINE DU PROJET
1. Collecte et prétraitement des données
2. Prétraitement des données de séries chronologiques
3. Analyse des séries chronologiques à l'aide du modèle SARIMA
4. Modélisation des séries chronologiques à l'aide de Facebook Prophet
5. Problèmes liés aux données échantillonnées toutes les heures
6. Conclusion

### 1. Data collection and pre -treatment
- I started by loading all the data games individually and I carried out the following tasks:
- Understand the data dictionary to obtain an overview of digital and categorical columns.
- Correct the data format.
- Clean the data by deleting the redundant columns, imputing zero values ​​and deleting the lines and columns in double.

#### Quick observations
#### 1. Customer data game:
- The customer dataset contains information on the geolocation of customers.
- We have a total of 99,441 customer identifiers, which constitutes the primary key of this table.These customer identifiers are created when a user makes a purchase.These are in fact transaction identifiers.
- We have a total of 96,096 unique customer identifiers.This indicates that we have around 96.6% of new customers.Only 3.4% of them made repeated purchases on the Olist platform.This is due to the fact that Olist was founded in 2015 and started to sell online in 2016. The data we downloaded from Kaggel date from 2016 to 2018, a period at which the company was relatively recent.So we only have new customers.
- This data set has "four" object type columns and "one" digital type column.
- There are no duplicates between lines or columns.
- There is no zero value.


#### 3. Sellers data set:
- The sellers' data set contains information on the location of the seller.- We have a total of 3095 unique seller identifiers, which is the primary key to this data set.
- This data set has three object type columns and a digital type column.
- There is no duplicate between lines and columns.
- There is no zero value.

#### 4. Payment data set:
- The payment data set contains information on customer payment method for each order.
- We have a total of 99,441 customer identifiers, which corresponds to the total number of order identifiers, but we have payment information for 99,440 orders.
- This data set has three object type columns and two digital type columns.
- There is no duplicate between lines and columns.
- There is no zero value.
- Order_id is the foreign key to this table.We will keep this side table because we are not interested in this table for our current workfield.

#### 5. Control articles data set:
- The data articles data set contains information on order items.It indicates the number of items of each order, the shipping limit and the amount of shipping costs.
- We have a total of 98,666 control identifiers, which is less than 99,441.
- This data set has four object type columns and three digital type columns.
- The “Shipping Deadline” column is in date/hour format, so we must convert it to correct format.
- There are no duplicates between lines or columns.
- There are no zero values.

#### 6. Command data set
- The command data set contains command information.Each order contains a customer identifier, its status, the marketing of the purchase and the actual and estimated delivery information.
- We have a total of 99,441 unique orders, which is the primary key of this table.
- This data set has eight object type columns.
- ** There are five columns of date and time values, but they are recorded in object format.We must convert them to the date and time format. **
`` `
#converting all lines containing date and time data in date-hour format.Orders ['Horodetage_Achat_Commande'] = PD.TO_DATETIVE (commands ['Horodatage_Achat_command'])))
Orders ['approval_command'] = pd.To_datetime (commands ['approval_command'])
Orders ['Date_livreation_transporter'] = PD.TO_DATETIME (Orders ['Date_livreation_transporter'])
Orders ['Date_Livreation_Client'] = PD.TO_DateTime (Orders ['Date_livreation_client'])
Orders ['Date_Livreation_estimation_Commande'] = PD.TO_DateTime (commands ['Date_livreation_estimation_command'])
`` `
- There are no duplicates between lines or columns.- There are zero values ​​in the fields order_approved_at, order_delivered_carrier_date and order_delivered_customer_date.
We have zero values ​​in three columns: order_approved_at, order_delivered_carrier_date and order_delivered_customer_date.Does this have a link with the status of the order?
`` `
#PourCentage of orders according to different statutes
order_st_per = round (orders ['order_status']. Value_counts (normalize = true)*100, 2)

#Graphic representation of orders to view their statutes.Plt.Figure ()
graph = plt.bar (order_st_per.index, order_st_per.values)
PLT.TTLE ("percentage compared to the total of orders")
PLT.xticks (rotation = 45)
For P in the graph:
height = p.get_height ()
Plt.annotate ("{}%". Format (height), (p.get_x () + p.get_width ()/2, height + .05), ha = "center", va = "bottom", fontsize = 9)
PLT.Show ()
`` `

We have a total of eight order statutes.Our data indicates that 97 % of orders have been delivered.Only 0.63 % were canceled.Different command statuses make it possible to determine at which stage is the order.

We want to determine whether there is a link between missing values ​​and the status of the order.We will filter the missing lines and check the status of the command.

The customer paid the order → Order created
The seller approved the command → approved order
The seller prepares the order → control of the order
The seller charged the command → Billed command
The seller has sent the order and sent it to the logistics partner → shipped order
The logistics partner delivered the product to the end customer → command delivered

** We may not need the lines whose order status is unavailable, invoiced, under treatment, created, approved or shipped.We will therefore delete them. ** Logically, these lines do not contain the correct values ​​corresponding to the status.If we impute the missing values, we should also modify the status.These lines are very few compared to the total and their deletion should not have much effect.

** It is necessary to charge the missing values ​​for the command statutes delivered and canceled **.
##### How to attribute the missing values?- We can find the difference in days between the known values, for example order_purchase_timestamp, and the columns containing the nan.
- We can visualize the distribution of the days obtained in the previous step by drawing the mustache box.
- We can then decide if we want to charge the missing values ​​with the average, the median or the mode of the difference of days.
- Once this step has been taken, we can determine the missing value by adding the difference in days to the date of purchase of the order.

##### Observations:
- We note the presence of many aberrant values ​​in these graphs.
- The number of days between the date of purchase and the date of the carrier is a negative observation.If we assume that the data is saved on the Olist server, where the time zone factor has been deleted, this observation is incorrect and can be deleted.
- by examining the mustaches boxes, we see many aberrant values;The imputation by the median is therefore the best approach.
We will simply delete these columns, because we do not want to treat them in our current perimeter.If we were to impute, we would calculate the median number of days for each of the diff_approved, diff_logist and diff_del values, then add it to order_purchase_timestamp to obtain order_approved_at, order_delivered_carrier_date and order_datered_custom_date respectively.

#### 7. Product data set:
- The product data set contains information on product categories and their attributes.
- We have a total of 32,951 product identifiers, which is the primary key of this table.There are a total of 73 product categories.
- This data set has two object type columns and seven digital type columns.
- There are no duplicates between lines or columns.
- There are few zero values ​​and we must attribute these values.

##### How to attribute these missing values?
-We found that the majority of missing lines contain categorical data, that is to say… name of the product category and its digital values, description, length of the name and quantity of photos.
- We can identify the lines corresponding exactly to the columns (weight, length, height, width) and inform the zero value with the name of the corresponding category.
- We can inform the other values ​​(product length of the product, product name and quantity of photos) with the average, the median or the mode of this known category.
- If there are several correspondence for the product category, we can filter the most corresponding category.
- In the absence of correspondence, we can create a separate "other" category and inform the rest with the average, the median or the mode (determined after the creation of a boxplot).
- We will separate the lines from the data dataframa with the missing values ​​in a separate dataframa (missing) and create another dataframe (all_values) without zero value.
- We will look for the element element correspondence by the missing values ​​with All_Values.- We only miss two lines for the weight, height, length and width of the product.We will use the average to fill these values.
#### 8. Notice data set:
- The opinion data set contains information on the opinions left by customers.It includes the note, the commentary, the date of creation and the timeing for submission of the opinion.
**- We have a total of 99,224 review identifiers, 98,410 of which are unique identifiers.This means that 814 opinions were submitted again.These are the ones that must be treated.
- We have 98,673 unique control identifiers and 98,410 unique notice identifiers.This means that there are 263 reviews with the same order identifier.These opinions may concern different products ordered under the same identifier. **
- This data set has "six" object type columns and "one" digital type column.- There are no duplicates between lines or columns.
- There are 145,903 zero values.

#### 9. Product_eng data set:
- I merged the product category with the English names of the products and abandoned the original name of the category with names in Portuguese.

#### Junction of all tables
- We will join the tables in order to obtain a main table to resolve the business problem of sales prediction.
- From the "commands" data set, we will first join the "articles_commandes" data set, then the "product" data set.
- Since we have already cleaned the "command" data set and saved it in a CSV file, we will load it and start from there.
- So I merged the "Orders" and "Articles_commandes" tables, the "Products" table with "Command_Comp", the "sellers" data with "command_con", the folished "customer" data and the "opinion" data set.

The Kaggel website has taught us that the total value of the order is calculated from quantity and price.Since the price indicates the unit price, the total value of the order is equal to the quantity x the price.
- Creation of the “Amount_total” column.`` Final_Cleaned ['Total_amount'] = Final_Cleaned ['Qty']*Final_Cleaned ['Price'] `` `` `

#### Extraction of holiday data
- Since we will predict the amount of sales, we must collect information on the holidays to help our model understand their impact.We will create a simple scape to collect information on Brazilian national holidays from this website (https://www.officeholidays.com/countries/brazil/).
- We can add 2017 and 2018 to the URL above to obtain the pages containing the information on the holidays of the years 2017 and 2018.
`` `
# For the web scraping (the requests package allows you to send HTTP queries via Python)
import relates
from BS4 Import Beautifulsoup

# To perform Regex operations
import re

# To add deadlines to avoid queries spam
import time
#define Empty dictionary to save content
content = {}

#Scaping information on holidays for pages 2017 and 2018
For I in [2017, 2018]:
URL = 'https://www.officeholidays.com/countries/brazil/'
URL = URL+STR (I)
Response = Requests.get (URL)
Soup = Beautifulsoup (Response.Content)
Content [i] = Soup.find_all ('Time')

#Extract of information on holidays from the extracted data
Empty #List
Holidays = []
For Key in Content:
dict_size = len (content [key])
dict_val = content [key]
For J in Range (0, dict_size):
Holidays.Append (dict_val [J] .attrs ['Datetime'])

#Creation of a dataframe for public holidays
Holidays_df = pd.dataframe (index = [Holidays], data = np.ones (len (Holidays)), columns = ['is_holiday'])))
Holidays_df.head ()
`` `
- This dataframe has only one column: "is_holiday", which means that it is a public holiday.The index corresponds to the dates of public holidays.
- These dates concern the years 2017 and 2018. The index is not continuous;These are only the dates of public holidays.We have recorded the data as well in order to be able to use it for chronological series.
#### Final data
- A total of 96,000 unique orders.
- The Olist platform has 96.79% of new customers and 3.21% made repeated purchases.
- A total of 32,000 different products belonging to 74 categories were sold.
- Global turnover in August 2018 amounts to 14.9 million Brazilian reals.
- The 2017 Black Friday recorded a sales record of 184,000 Brazilian reals.
- The five main categories are:


- Les commandes mensuelles et le chiffre d'affaires ont enregistré une croissance.

### 2. PRESTRATORY OF DATA OF CHRONOLOGICAL SERIES

### Data dictionary
We have a total of 110,013 command lines with 28 characteristics.I specify all the general details of the data extracted during cleaning and sorting of data.Each line of the table indicates an order with the product category purchased, the quantity of items purchased, the unit price of the product, as well as information on the purchase delay, delivery methods, evaluation note and customer and seller information.

- ** order_id **: Specifies the single command.We have 95,832 unique orders.Over 110,000 lines, an order_id can reappear in the dataframe, but it will have another product category and the number of items purchased in this category.
- ** Customer_id **: Specifies the customer identifier of the order.A customer identifier is associated with each order.There are a total of 95,832 unique customer identifiers.
- ** Order_purchase_timestamp **: The horoditing of the command.It includes the date and time.
- ** order_estimated_delivery_date **: estimated delivery date at the time of purchase.
- ** Qty **: Number of items purchased in a product category.
- ** Product_id **: Indicates the real product in a product category.We have 32,072 unique products in 74 product categories.
- ** SELLER_ID **: We have 2,965 unique sellers.
- ** Shipping_limit_date **: This date informs the seller of the shipping limit so that he can ship the order as soon as possible.
- ** Price **: unit price of each product.
- ** FREIGHT_VALUE **: transport costs calculated according to the weight and dimensions of the product.This value is valid for a single article.If there are three articles, the total cost of transport will be equal to three times the value of the freight.
- ** Product_name_lenght **: Number of characters extracted from the product name.
- ** Product_description_lenght **: Number of characters extracted from the description of the product.
- ** Product_photos_qty **: Number of photos of published products.
- ** Product_Weight_G **: Product weight in grams.
- ** Product_length_cm **: Length of the product in centimeters.
- ** Product_height_cm **: Product height in centimeters.
- ** Product_width_cm **: Product width in centimeters.
- ** Product_category_name_english **: English names in product categories.
- ** seller_city **: City of the seller.
- ** seller_state **: state of the seller.
- ** SELLER_LAT **: Latitude of the seller.
- ** seller_lng **: longitude of the seller.
- ** Customer_unique_id **: 92,755 unique customers, or 96.79% of the total of the database customers.Only 3.21% of customers made repeated purchases.This may be due to the fact that the data we have is that of the creation of olist, when it had just started its activity.Therefore, we have all new customers in the database.
- ** Customer_city **: City of the customer.
- ** Customer_state **: Customer condition.
- ** Customer_lat **: Latitude of the customer.
- ** Customer_lng **: Longitude of the customer.
- ** review_score **: Customer reviews are between 1 and 5.


** `Target variable` **: ** Amount_total **: We calculated this value after having multiplied ** Qty ** by ** Price **.This is the actual amount of sales, important for the company.We predict this amount to prepare for the future.

`Note ': We did not take into account the transport costs in the calculation of the" amount_total ", because we found that in its infancy, Olist subcontracted the logistics to a third party.We therefore want to provide a sales overview only on the products sold on the Olist platform.

We also found that Olist had acquired Pax, his logistics partner, later in 2020. Consult [here] (https://www.bloomberglinea.com/english/olist-abecomes-brazils-newest-unicorn-raises-186m/) for more details.

#### Data processing for time series
We found that the format of the “order_purchase_timestamp” column was incorrect.We will start by converting this column into a date-hour format and trying to extract certain characteristics for the analysis.

We can extract information from year, date, month, day of the week and day from the dates.
`` `
#Conversion of date columns in object format in date-hour format
Master ['Purchase_year'] = pd.to_datetime (master ['order_purchase_timestamp']).
Master ['Purchase_month'] = Pd.To_datetime (Master ['Order_Purchase_timestamp']).
Master ['Purchase_mmyyyy'] = pd.to_datetime (master ['order_purchase_timestamp']). Dt.strftime ('%m-%y')
MASTE R ['Purchase_week'] = PD.TO_DateTime (Master ['Order_purchase_timestamp']). Dt.isocalendar (). Week
Master ['Purchase_Dayofweek'] = pd.to_datetime (master ['order_purchase_timestamp']). Dt.weekday
Master ['Purchase_Dayofmonth'] = pd.to_datetime (Master ['Order_purchase_timestamp']). DT.DAY
`` `

We will aggregate the Total_amount by date in order to obtain a chronological series, that is to say a dataframe with the Total_amount column organized by date.We will define dates as index.

### Data exploratory analysis
#### 1. Thermal card:
- Identify the digital characteristics strongly correlated in total_amount.This is simply a general overview of identifying the characteristics likely to have an impact on sales and their correlation.! [Image]
** Observations **:
- We note that the total amount is strongly correlated at the price.This is obvious, because we know that the total amount has been calculated from the price.
- The “purchase week” and “month of purchase” values ​​are strongly correlated.
- The values ​​"weight of the product" and "freight" are positively correlated, because freight is calculated according to the weight of the product, as specified by data editors on Kaggle.
- We do not see any other characteristic with a strong correlation with the total amount.

#### 2. Histogram:
- to view the distribution of the total amount.! 
** Observations **:
- There is a peak at zero, because we have no observation for most days in 2016.
- If we do not know that, our global distribution is normal, with some aberrant values ​​on the right side.These aberrant observations concern the peak sales period.



** Observations: **
- The categories health_beauty, watches_cadeaux, lit_able_de_bain, accessories_informatics and leisure_sports are the most profitable product categories.
- The categories Games_PC, CD_DVD_MUSIQUES, MODE_VESTIONS_ENFANTS are the least profitable product categories.

#### Decomposition of chronological series
** We will decompose the chronological series by additive decomposition in order to observe the underlying trend, seasonality and residues **.

Additive decomposition: $ trend $ + $ seasonality $ + $ residual $

`` `
# Decompose the chronological series
Decomposition = tsa.seasonal_decompose (Daily_Data, Model = 'Additive')
# Save a copy in a new database
Daily_df = Daily_data.copy ()
# Add decomposition data
Daily_df ['trend'] = decomposition.trend
Daily_df ['seizurel'] = decomposition.seasonal
Daily_df ['residual'] = decomposition.resid
`` `
`` `
# Represent the actual and decomposed components of the chronological series
passes = ["amount_total", "trend", "seasonal", "residual"]

Fig = make_subploots (rows = 4, passes = 1, subplot_titles = passes)

For I, collar in enumerer (passes):
Fig.add_Trace (
Go.Scatter (x = Daily_df.index, y = Daily_df [col]),
Row = I+1,
col = 1
))

fig.update_layout (height = 1200, width = 1200, showlegend = false)
# fig.show ()
Fig.show ("SVG")
`` `


** Observations **:
- There is a slight upward trend.The trend reaches a peak on November 26, 2017, due to Black Friday sales of November 24, 2017. It then decreases, then goes up.Although this Black Friday is an aberrant value, we must take it into account in our calculations, because it is an important factor.
- There is a weekly seasonality: the value reaches a peak per week, then decreases.
- There is no clear trend in the residue.He captured the peaks of November 24, 2017 and September 29, 2017.

#### Modeling preparation
1. Training and distribution test
2. Definition of functions for the layout of predictions and forecasts
3. Definition of evaluation functions
We will define functions to calculate the average cumulative weighted error (MAPE) and the average quadratic error (RMSE).If we have there as real value and predictions as a predicted value for n observations, then:

MAPE (average error in absolute percentage): this is a simple average of absolute percentage errors.It is calculated by:

$$ \ FRAC {1} {n} \ sum_ {i = 1}^{n} {|\ FRAC {y_ {Actual_i} - Predictions_ {i}} {y_ {Actual_i} |} \ Times {100} $$$

RMSE (Root Mean Squared Error): this is the square root of the square of the square of the difference between the initial values ​​and predicted in the data set.

$$ \ SQRT {\ FRAC {1} {n} \ sum_ {i = 1}^{n} {(y_ {news_i} - predictions_ {i})}^2}}} $$

### 3. Modeling
### 10. Samari
We will start with the Samari model to take into account the seasonality of our model.Samari means "seasonal self -regressive integrated mobile average", which explicitly supports the univariated chronological series with a seasonal component.Before moving to modeling, we must understand the orders to choose for self -regressive and mobile mobile averages.We will draw the ACF and PACF curves to determine it.

ACF: autocorrelation function, describing the correlation between the original series and the offbeat series.
PACF: partial correlation function, identical to ACF, but deleting all the intermediate effects of shorter offsets, leaving visible only the direct effect.

#### Curves ACF and PACF curves
`` `
DEF PLOT_ACF_PACF (DF, ACF_LAGS: INT, PACF_LAGS: INT) -> NONE:
"" "
This function draws the offender and partial autocorrelation offsets.
---
Arguments:
DF (PD.DataFrame): DataFrame contains the number of orders and dates.
ACF_LAGS (int): number of ACF offsets
PACF_LAGS (int): number of PACF discreets
Refers: none
"" "

# Figure
Fig, (ax1, ax2) = PLT.Subploots (1, 2, Figsize = (16.9), facecolor = 'W')

# ACF and PACF
Plot_acf (DF, AX = AX1, LAGS = ACF_LAGS)
Plot_pacf (df, ax = ax2, lags = pacf_lags, method = 'ywm')

# Labels
ax1.set_title (f "autocorrelation {df.name}", fontsize = 15, pad = 10)
ax1.set_ylabel ("amount of sales", fontsize = 12)
ax1.set_xlabel ("Deadline (days)", fontsize = 12)

ax2.set_title (f "partial autocorrelation {df.name}", fontsize = 15, pad = 10)
ax2.set_ylabel ("amount of sales", fontsize = 12)
ax2.set_xlabel ("Deadline (days)", fontsize = 12)

# Legend and grid
ax1.grid (linestyle = ":", color = 'gray')
ax2.grid (linestyle = ":", color = 'gray')

PLT.Show ()
`` `


**Observation** :
ACF curve:
- It shows many significant discrepancies.*In the ACF curve, no discrepancy becomes zero.*** This means that our data is not stationary, as we have explained using statistical tests and observation of the mobile average and the standard deviation. **
- It will be difficult to determine the order of mobile averages and mobile averages;We will have to differentiate them in order to identify significant discrepancies.
- We note that the gap reaches a peak every 7 days.This is the seasonality of the model.

PACF curve:
- The PACF model has some significant discrepancies, but the curve does not decrease much and has very few oscillations.It is therefore difficult to assert or determine if the mobile averages can be used on this model.

We will try to draw the ACF and PACF curves by double differentiation, that is to say by differentiating the daily difference with seasonal difference data.

#### application of the sama model
The sama model is specified:

$$ samari (p, d, q) \ Times (p, d, q) _s $$

Or :
- The trend elements are:
- P: autoregressive order
- D: order of difference
- Q: Mobile average order
- seasonal elements are:
- P: Seasonal self -regressive order.
- D: seasonal difference order.D = 1 would calculate a first -rate seasonal difference.
- Q: Seasonal mobile average order.Q = 1 would use first -rate errors in the model.
- S: unique seasonal period

### Theoretical estimates:
- ** S **: Our PACF graphic has a peak that reappears every 7 days.We can therefore define the seasonal period at ** s = 7 **.This is also confirmed by our seasonal component after additive decomposition.
- ** P **: We have observed a certain variation in the ACF graphic and found significant discrepancies of 1, 2 and 3 compared to the PACF graphic.Let's start with ** p = 1 ** and observe its operation.
- ** d **: We have observed a trend in our series, so we can delete it by differentiation, therefore ** D = 1 **.
- ** Q **: According to our ACF correlations, we can define ** Q = 1 **, because it is the most significant gap.
- ** P **: ** P = 0 **, because we use the ACF graphic to find a significant seasonal gap.
- ** d **: Since we deal with seasonality and we have to differentiate the series, ** D = 1 **
- ** Q **: The seasonal mobile average will be defined on ** Q = 1 **, because we have found only one significant seasonal gap in the ACF graphic.
Let's go:

$$ samari (1, 1, 1) \ Times (0, 1, 1) _ {7} $$

### bas Basic sama
`` `
# Definition of hyperparameters
p, d, q = 1, 1, 1
P, d, q = 0, 1, 1
s = 7

# Sarima adjustment
samari_model = sarimax (train_df ['Total_amount'], order = (p, D, q), Seasonal_order = (P, D, Q, S))))
samari_model_fit = samari_model.fit (dispos = 0)
print (samari_model_fit.summary ())
`` `

#### Observations:
- ** The graph of standardized residues **: residues appear as a white noise.They resemble the residue of the decomposed time series.
- ** The normal Q-Q graphic **: shows that the ordered distribution of residues follows the linear trend of samples from a standard normal distribution with N (0, 1).As indicated above, there are some aberrant values.
- ** The histogram and the estimated density graph **: the KDE distribution follows the right n (0, 1), but with notable differences.As indicated above, our distribution has thicker tails.
- ** The correlogram graph **: shows that the residues of the temporal series are weakly correlated with their delayed versions.This means that there are no more models to extract in residues.

Let's test the model on our training set:

###Ove predictions and assessment of the sama model
** Prediction with Samari **
`` `
# Definition of the prediction period
pred_start_date = test_df.index [0]
pred_end_date = test_df.index [-1]

samari_predications = samari_model_fit.predict (start = pred_start_date, end = pred_end_date)
samari_residuals = test_df ['Total_amount'] - samari_pring
`` `

** Sarima assessment **
`` `
# Recovery of evaluation data
samari_root_mean_squared_error = rmse_metrics (test_df ['Total_amount'], samari_predictions)
samari_mape_error = mape_metrics (test_df ['Total_amount'], samari_predications)

print (medium quadratic f'erreur | rmse: {sarima_root_mean_squared_error} ')
print (absolute f'erreur average in percentage | mape: {samari_mape_error} ')
`` `
Average quadratic error |RMSE: 13,810.6
Absolute Absolute Error in percentage |MAPE: 68.99
We obtain a Mape of 69.99% and an RMSE of 13,810.6.

#### Sarima forecasts
We will try to plan sales for the next 180 days.The 121 days being known thanks to our test data, we will try to see what our model provides for the next 60 days.`` `
# Forecast window
days = 180

samari_forecast = samari_model_fit.forecast (days)
samari_forecast_series = pd.series (samari_forecast, index = samari_forecast.index)

# Negative orders being impossible, we can cut them.
samari_forecast_series [samari_forecast_series <0] = 0
`` `

** Tracking forecasts using the reference value Samari **
`` `
Plot_Forecast (train_df ['Total_amount'], test_df ['Total_amount'], samari_forecast_series)
`` `

Observations:
- The model fairly well predicts global daily trends.
- He fails to effectively detect the variations between weeks and months.
- It follows a positive trend and does not capture peaks and hollows.
- We will have to refine it more and add a functionality "holidays" so that it can extract information from it.
- Although this model does not have a great long -term predictive power, it can serve as a solid basis for our next models.

### 4. Modeling of chronological series with FB Prophet
FB Prophet is a python forecast package developed by the Facebook data research team.Its objective is to provide professional users with a powerful and easy -to -use tool to provide commercial results without the need to be an expert in chronological series analysis.We will apply this model and observe its performance.

** Preparation of data for FB Prophet **
FB Prophet needs data in a specific format to be able to process them.The entrance to Prophet is always a dataframe comprising two columns: DS and Y.The DS column (Datestamp) must respect the format expected by Pandas, ideally AAAA-MM-JJ for a date or AAAA-MM-JJ HH: MM: SS for a time time for.The column must be digital and represents the measure, in our case: Total_amount.
```
#Preparation of data for fbprophet

prophet_df = dfex ['Total_amount']. Reset_index ()
prophet_df.rename (columns = {"index": "ds", "total_amount": "y"}, inplace = true)

#En using our original trains_DF and test_DF, we will convert them into Trains and Test Prophet.prophet_train = train_df ["total_amount"]. Reset_index ()
prophet_train.rename (columns = {"order_purchase_timestamp": "ds", "total_amount": "y"}, inplace = true)
Prophet_test = test_df ["Total_amount"]. Reset_index ()
prophet_test.rename (columns = {"order_purchase_timestamp": "ds", "total_amount": "y"}, inplace = true)
`` `

** Application of a reference Prophet FB **
Since we have observed a positive trend and seasonality for our data, we will define Growth = 'Linear' and let the model Determine the appropriate seasonality by defining Yearly_SeAonality, Daily_Seasonality and Weekly_Seasonality = True.`` `
#Instancing the model
fb_baseline = prophet (growth = 'linear',
Yearly_seasonality = True,
Daily_seasonality = True,
Weekly_seasonality = True)
fb_baseline.fit (prophet_train)
`` `

** predictions using the basic prophet ** model
`` `
#Create predictions with dataaframe
future_base = fb_baseline.make_future_dataframe (periods = len (test_df), freq = "d")
#Create a forecast
forecast_base = fb_baseline.Predict (future_base)
Forecast_base [['ds', 'yhat', 'yhat_lower', 'yhat_upper']]. Tail ()
`` `

** Trace and assessment of the basic model **
`` `
#Assessment on the SET test
fb_baseline_mape = mape_metrics (prophet_test ['y'], forecast_base [-121:]. Reset_index () ['Yhat'])
fb_baseline_rmse = rmse_metrics (prophet_test ['y'], forecast_base [-121:]. Reset_index () ['yhat'])

print (medium quadratic f'erreur | rmse: {fb_baseline_rmse} ')
print (absolute f'erreur average in percentage | mape: {fb_baseline_mape} ')
`` `
Average quadratic error |RMSE: 14,904.05
Absolute Absolute Error in percentage |MAPE: 75.28

** Tracking forecasts with FB Basic Prophet **
`` `
from FBPROPHET.PLOT IMPORT PLOT_PLOTLY

Fig = plot_plotly (fb_baseline, forecast_base)
fig.update_layout (
Title = "amount of daily sales",
xaxis_title = "date",
yaxis_title = "amount of turnover"
))
# fig.show ()
Fig.show ("SVG")
`` `



Observations:
- Although the Prophet has not yet provided us with a reliable MAPE or RMSE from the graph, we can see that he is able to capture seasonality, the trend, certain peaks and hollow.
- It would be interesting to deepen the question by adjusting hyperparammeters and including the impact of holidays.

### 5. Problems with sampled data all timetables
In order to increase the number of data points, I tried to re-exchange the data set every hour and I found that I obtained a lot of zero values ​​at certain hours of the day, because no order had been placed at that time.I tried to apply the Saima model, but it gave negative predictions and a downward trend.After reading and consultation, I found that we had to carry out transformations or apply different approaches to process this data.Therefore, I limited myself to daily data.

### 6. Conclusion
#### Summary :

|Model |MAPE |
|------------------------------ |: --------: |
|Sarima (1.1,1) (0.1,1) (7) |68.99 |
|Basic prophet |71.78 |
|Basic prophet with holidays |77.88 |