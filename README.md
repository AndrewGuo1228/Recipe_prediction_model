The exploratory data analysis on this dataset can be found [here](https://andrewguo1228.github.io/Recipe-ratings-research-project/)

---
Yanhao Guo
# Framing the Problem

### Prediction Problem

In this project, the focus would be to predict **how much calories are there in the recipes**.

The type of the prediction problem is **regression**.

### Response Variable
 
The response variable I chose is the **calorie content of each recipe.** I selected this variable because of its vital importance in nutritional analysis and diet management. Monitoring calorie intake is essential for maintaining a balanced and healthy diet. By predicting the caloric value of various recipes, I aim to provide crucial information to help individuals manage their calorie consumption effectively.

### Evaluation Metric

R-squared (R2)

Reasoning:

1. This metric indicates the proportion of variation in the caloric content of recipes that the attributes in our predictive model can explain.

2. It offers a significant indicator of the effectiveness of our model in fitting the data and in elucidating the variation observed in the calorie figures of the recipes.

3. This approach is highly compatible for regression tasks.

### Justification of information known 

When training my model to predict calories, I have leveraged all the details included in the recipes, except for the ratings, as they are assigned only after the recipes are published. Therefore, the features I use in my model are solely based on the information available within the recipes. This includes attributes like ["total fat", "carbohydrates", "sugar", "n_ingredients", "steps", "tags"].

### Data Cleaning

I have done some basic data cleaning when doing EDA, which are included [in this website](https://xic051.github.io/DSC/report.html). 
The followings are some additional data cleaning that did for the purpose of the prediction models of this project. 

#### 1. Some unnecessary columns in the data frame dropped

Columns being dropped: `["nutrition", "contributor_id", "user_id", "recipe_id", "submitted", "rating", "average rating", "review", "date"]`

#### 2. Some rows for some columns dropped

- calories 

The original `merged_df` contains many receipes that have unreasonable calories, such as 45609.0. I set calories with less than 3000 as a resonable threshold. Therefore, I only want to focus on recipes with 3000 or less calories and filter out the rows with extreme calories values.

- total fat, sugar, sodium, protein, saturated fat, carrbohydrates 

Since the unit for these numiercal variables are **PDV**, which is **Percentage of Daily Value**, it is impossible for these columns to have a value that is greater than 100. So we filter out rows with a value that is greater than or equal to 100 among these columns in `merged_df`. 

---

# Baseline Model 

## Description of Our Model

In this model, I use three features - "total fat (PDV)", "published_year", and "tags" - to predict the calorie content of recipes. After processing these features, a detailed description of which is provided below, I constructed a pipeline incorporating a linear regression model. I then evaluated the model's performance using the R-squared metric on both the training and testing data.

### Feature Transforming and Encoding:

#### 1. `"n_ingredients"`

It is a **discrete numerical variable**.

I used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

#### 2. `"tags"`

It is a **nominal categorical variable** in form of a list of strings.  

I first used a **FunctionTransformer()** by using a custom function **transform_oven()** defined to tranform the `tags` column in the original data frame to a column of boolean series (True or False) checking whether the tag **"main-dish"** is included in the tags list of that recipe or not. The `tags` column now indicates whether each recipe is a main-dish or not. This transformation is based on the belief that main dishes typically have higher calorie contents compared to other types of dishes, such as appetizers or side dishes, as they often include protein-rich ingredients, larger portions, and more complex preparations.

After the custom transformation, I used **OneHotEncoder(handle_unknown='ignore')** to convert the transformed categorical variable (True or False) into a one-hot encoded representation. 

#### 3. `"carbohydrates"`

It is a **continuous numerical variable**.

I used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

## Performance of Our Baseline Model

R-squared on train data: 0.5689925700915398
R-squared on test data: 0.5716317971225593

Based on the performance, firstly, I can see that R-squared values for both train and test data are relatively close, indicating that our baseline model is not overfitting the training data and is performing consistently on unseen data.

Secondly, although the R-squared value isn't very close to 1, it is still a decent value, indicating that baseline model is in the right direction for prediction and can perform better after improvement in the final model.  

---

# Final Model

## Features Changes

### 1. add a `oven` feature from transformation of column `steps`

The variable in question is a nominal categorical variable, presented as a list of strings.

I initially applied a FunctionTransformer() with my custom function transform_oven(). This function is designed to transform the steps column in the original data frame into a column of boolean values (True or False), determining whether "oven" is mentioned in the steps. Consequently, the steps column now signifies whether each recipe includes a step that uses an oven.

I chose this transformation because I believe that cooking methods involving an oven, like baking or roasting, might contribute to higher calorie content. This is often due to the use of additional fats, oils, or ingredients high in calories. By incorporating this feature, my model can better account for the potential increase in calories associated with oven cooking, enhancing its accuracy in predicting the calorie content of recipes that involve oven-based cooking methods.

Following this custom transformation, I utilized OneHotEncoder(handle_unknown='ignore') to transform the boolean categorical variable (True or False) into a one-hot encoded format.


### 2. add `total fat` 

It is a **continuous numerical variable**.

**I choose to add it because** I believe that fat content is a significant contributor to the overall calorie content of a recipe. Fat is known to have a higher calorie density compared to other nutrients. By incorporating this feature, the model can take into account the amount of fat present in a recipe and its potential effect on calorie estimation. 

After plotting the distribution of `total fat`, I see that the distribution is **right-skewed**, so I decided to use **QuantileTransformer(n_quantiles=100)** to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/fat.html" width=800 height=600 frameBorder=0></iframe>


### 3. add `sugar`

It is a **continuous numerical variable**.

**I choose to add it because we believe** that sugar is another ingredient that significantly contributes to the calorie content of recipes. It is a carbohydrate and provides a substantial amount of calories per gram. By adding it, the model can include the impact of sugar on the overall calories when making predition. Recipes with higher sugar levels are likely to have higher calorie content. 

After plotting the distribution of `sugar`, we see that the distribution is **right-skewed**, so I decided to use **QuantileTransformer(n_quantiles=100)** to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/sugar.html" width=800 height=600 frameBorder=0></iframe>

### 4. Change (not adding new feature) how to process feature `carbohydrates` from StandardScaler() to QuantileTransformer(n_quantiles=100)

After plotting the distribution of `carbohydrates`, I see that the distribution is **right-skewed**, as the original StandardScaler() transformation assumes a normal distribution of the data, I decided to use **QuantileTransformer(n_quantiles=100)** instead which helps to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/carbohydrates.html" width=800 height=600 frameBorder=0></iframe>

**Note**

For the original feature `n_ingredients`, I continue to use **StandardScaler()** as the distribution of`n_ingredients` is a normal distribution. 

<iframe src="assets/n_ingredients.html" width=800 height=600 frameBorder=0></iframe>

Also, I keep the transformation and encoding of columns `tags` to check for "main-dish" tag from the baseline model the same in our final model. 

## Modeling Algorithm

I decided to use the **RandomForestRegressor** as my final model, an ensemble method based on decision trees. The best-performing hyperparameters I found were **'max_depth' set to 10** and **'n_estimators' set to 400**, as determined by GridSearchCV.

For selecting the optimal hyperparameters, I conducted a grid search coupled with 5-fold cross-validation. The 'max_depth' hyperparameter range was set from 1 to 14, and the 'n_estimators' range was from 100 to 400, increasing by 100 at each step. I trained and evaluated the model on each combination of hyperparameters, choosing the best one based on the mean cross-validated score.

The overall model is structured with a preprocessing pipeline, which includes feature transformations for the tags and steps columns. These columns are processed using OneHotEncoder after applying custom functions for data transformation. For the numerical features - **total fat**, **carbohydrates**, and **sugar** - I use a QuantileTransformer to address the right-skewed distribution. Additionally, the 'n_ingredients' column is standardized using the StandardScaler.


## Performance of the final model




| Hyperparameters                                              |    Score |
|:-------------------------------------------------------------|---------:|
| {'regressor__max_depth': 1, 'regressor__n_estimators': 100}  | 0.452017 |
| {'regressor__max_depth': 1, 'regressor__n_estimators': 200}  | 0.451658 |
| {'regressor__max_depth': 1, 'regressor__n_estimators': 300}  | 0.451493 |
| {'regressor__max_depth': 1, 'regressor__n_estimators': 400}  | 0.451357 |
| {'regressor__max_depth': 2, 'regressor__n_estimators': 100}  | 0.679458 |
| {'regressor__max_depth': 2, 'regressor__n_estimators': 200}  | 0.679707 |
| {'regressor__max_depth': 2, 'regressor__n_estimators': 300}  | 0.680026 |
| {'regressor__max_depth': 2, 'regressor__n_estimators': 400}  | 0.679295 |
| {'regressor__max_depth': 3, 'regressor__n_estimators': 100}  | 0.817722 |
| {'regressor__max_depth': 3, 'regressor__n_estimators': 200}  | 0.817311 |
| {'regressor__max_depth': 3, 'regressor__n_estimators': 300}  | 0.817955 |
| {'regressor__max_depth': 3, 'regressor__n_estimators': 400}  | 0.818082 |
| {'regressor__max_depth': 4, 'regressor__n_estimators': 100}  | 0.878611 |
| {'regressor__max_depth': 4, 'regressor__n_estimators': 200}  | 0.87835  |
| {'regressor__max_depth': 4, 'regressor__n_estimators': 300}  | 0.878665 |
| {'regressor__max_depth': 4, 'regressor__n_estimators': 400}  | 0.878772 |
| {'regressor__max_depth': 5, 'regressor__n_estimators': 100}  | 0.90676  |
| {'regressor__max_depth': 5, 'regressor__n_estimators': 200}  | 0.907008 |
| {'regressor__max_depth': 5, 'regressor__n_estimators': 300}  | 0.906999 |
| {'regressor__max_depth': 5, 'regressor__n_estimators': 400}  | 0.907078 |
| {'regressor__max_depth': 6, 'regressor__n_estimators': 100}  | 0.923404 |
| {'regressor__max_depth': 6, 'regressor__n_estimators': 200}  | 0.923654 |
| {'regressor__max_depth': 6, 'regressor__n_estimators': 300}  | 0.923694 |
| {'regressor__max_depth': 6, 'regressor__n_estimators': 400}  | 0.9237   |
| {'regressor__max_depth': 7, 'regressor__n_estimators': 100}  | 0.931153 |
| {'regressor__max_depth': 7, 'regressor__n_estimators': 200}  | 0.931392 |
| {'regressor__max_depth': 7, 'regressor__n_estimators': 300}  | 0.931374 |
| {'regressor__max_depth': 7, 'regressor__n_estimators': 400}  | 0.931367 |
| {'regressor__max_depth': 8, 'regressor__n_estimators': 100}  | 0.935308 |
| {'regressor__max_depth': 8, 'regressor__n_estimators': 200}  | 0.935338 |
| {'regressor__max_depth': 8, 'regressor__n_estimators': 300}  | 0.935403 |
| {'regressor__max_depth': 8, 'regressor__n_estimators': 400}  | 0.935379 |
| {'regressor__max_depth': 9, 'regressor__n_estimators': 100}  | 0.936714 |
| {'regressor__max_depth': 9, 'regressor__n_estimators': 200}  | 0.936888 |
| {'regressor__max_depth': 9, 'regressor__n_estimators': 300}  | 0.936831 |
| {'regressor__max_depth': 9, 'regressor__n_estimators': 400}  | 0.936887 |
| {'regressor__max_depth': 10, 'regressor__n_estimators': 100} | 0.937035 |
| {'regressor__max_depth': 10, 'regressor__n_estimators': 200} | 0.937031 |
| {'regressor__max_depth': 10, 'regressor__n_estimators': 300} | 0.937078 |
| {'regressor__max_depth': 10, 'regressor__n_estimators': 400} | 0.93709  |
| {'regressor__max_depth': 11, 'regressor__n_estimators': 100} | 0.936207 |
| {'regressor__max_depth': 11, 'regressor__n_estimators': 200} | 0.936263 |
| {'regressor__max_depth': 11, 'regressor__n_estimators': 300} | 0.936351 |
| {'regressor__max_depth': 11, 'regressor__n_estimators': 400} | 0.936343 |
| {'regressor__max_depth': 12, 'regressor__n_estimators': 100} | 0.935161 |
| {'regressor__max_depth': 12, 'regressor__n_estimators': 200} | 0.935173 |
| {'regressor__max_depth': 12, 'regressor__n_estimators': 300} | 0.935427 |
| {'regressor__max_depth': 12, 'regressor__n_estimators': 400} | 0.935363 |
| {'regressor__max_depth': 13, 'regressor__n_estimators': 100} | 0.933754 |
| {'regressor__max_depth': 13, 'regressor__n_estimators': 200} | 0.934111 |
| {'regressor__max_depth': 13, 'regressor__n_estimators': 300} | 0.934086 |
| {'regressor__max_depth': 13, 'regressor__n_estimators': 400} | 0.934168 |
| {'regressor__max_depth': 14, 'regressor__n_estimators': 100} | 0.932521 |
| {'regressor__max_depth': 14, 'regressor__n_estimators': 200} | 0.932913 |
| {'regressor__max_depth': 14, 'regressor__n_estimators': 300} | 0.933112 |
| {'regressor__max_depth': 14, 'regressor__n_estimators': 400} | 0.933133 |






Compared to the baseline model, the final model incorporates additional features, improves the preprocessing process by handling skewed data, and utilizes a more advanced algorithm to search for the best combinations of hyperparameters. The performance of the final model is improved in terms of R-squarted value from around **0.5689925700915398** for training data and **0.5716317971225593** for testing data, to  **0.9486009222505515** for training data and **0.9466226769946644** for testing data. 

---

# Fairness Analysis 

**Group X**: Recipes categorized as 'main-dish'

**Group Y**: Recipes not categorized as 'main-dish'

**Evaluation Metric**: R-squared 

**Null Hypothesis (H0)**: The model is fair with respect to the precision of predicting calories of main-dish and non-main-dish recipes.  The regressor's performance (i.e. R-squared values) for main-dish recipes and non-main-dish recipes are the same, and any differences are due to random chance.

**Alternative Hypothesis (Ha)**: The model is unfair with respect to the precision of predicting calories of main-dish and non-main-dish recipes. The regressor's performance (i.e. R-squared values) for non-main dish is lower than its precision for main dish.

**Test Statistic**: The difference in R-squared values between the two groups (recipes for main-dish minus recipes for non main-dish).

**Significance Level**: 0.05 (representing a 5% risk of concluding that a difference exists when there is no actual difference)

**Observed difference**: -0.017222756653036186

**P-value**: 1.00

**Conclusion**: 

With a p-value of 1.00, so it fails to reject the null hypothesis. Therefore, based on this specific test and evaluation metric (R-squared), there is insufficient evidence to conclude that the model's performance differs significantly between 'main-dish' and 'non-main dish' recipes. This indicates that, according to this specific test and evaluation metric, the model does not seem to display a significant bias towards either 'main-dish' or 'non-main dish' recipes. However, I cannot make a conclusion that our model is fair in other dimensions, and additional analyses and evaluations may be necessary to comprehensively assess fairness beyond this particular metric.

**Visualization**:

<iframe src="assets/fairness.html" width=800 height=600 frameBorder=0></iframe>

```python

```
