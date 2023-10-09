# datasci_5_statistics
This is a repository for Assignment 5 in HHA507. 

# Chi Square Test

First import the appropriate packeges
```
import pandas as pd
from scipy.stats import chi2_contingency as chi2
```

## The Failed Attempt
This Chi-Square Test was performed on a dataset with two categorical variables. I initially tried using a dataset that looked at Covid-19 effect on Nursing Homes. 
Using .value_counts(), I found that the two categorical variables I was looking at, 'Submitted Data' and 'Passed Quality Assurance Check' did not have equal numbers. 
This told me that one variable was missing values. I used the fillna function to replace the missing values with a value. 
```
df2['Passed Quality Assurance Check'].fillna("N", inplace=True)
```
df2 being my dataset. 

Afterwhich, I tried making a contingency table 
```
df2['Passed Quality Assurance Check'].value_counts()
```
```
Passed Quality Assurance Check      N        Y
Submitted Data                                
N                               18925        0
Y                               13814  2623100
```

Next, I tried this function
```
chi2, p, _, _ = chi2(contingency_table2)
print(f"Chi2 value: {chi2}")
print(f"P-value: {p}")
```
```
TypeError                                 Traceback (most recent call last)
<ipython-input-177-38f00422182c> in <cell line: 1>()
----> 1 chi2, p, _, _ = chi2(contingency_table2)
      2 print(f"Chi2 value: {chi2}")
      3 print(f"P-value: {p}")

TypeError: 'numpy.float64' object is not callable
```

There were multiple ways I tried manipulating the data, which may have messed up the data.

## Successful attempt
I then tried another dataset. This one looked at hospital enrollments, however when trying to run pd.read_csv, I was seeing a unicode error. This is most likely due to an encoding problem with how the data
was converted into a CSV file. This was solved after I added, encoding ='unicode_escape' to the end of the line. The encoding unicode_escape is a dummy encoding that converts all non-ASCII characters into their \uXXXX representations. 
```
df = pd.read_csv('/content/sample_data/Hospital_Enrollments_September_2023.csv', encoding='unicode_escape')
```
Hypothesis: There is an association between if a hospital has more than one NPI (more than one doctor) and that the provider converted from a rural to a critical access hospital.

Null Hypothesis: There is no association between if if a hospital has more than one NPI and that the provider converted from a rural to a critical access hospital.

I followed similar steps as before.
```
contingency_table = pd.crosstab(df['MULTIPLE NPI FLAG'], df['REH CONVERSION FLAG'])
print(contingency_table)
```
```
REH CONVERSION FLAG     N  Y
MULTIPLE NPI FLAG           
N                    8446  8
Y                     939  0
```
```
chi2, p, _, _ = chi2(contingency_table)
print(f"Chi2 value: {chi2}")
print(f"P-value: {p}")
```
```
Chi2 value: 0.12492900642152108
P-value: 0.7237488760521203
```

The P-value being of 0.7 > 0.05 indicates that there is not enough statistical evidence that the variables are associated. This means that there is no strong evidence to reject the null hypothesis.

The Chi2 value being 0.12 indicates that there is a relatively low level of difference between observed and expected values.

# T-Test

First import the appropriate packages.
```
from scipy.stats import ttest_ind
```
I used the same data as before that looked at Covid 19 at Nursing homes. A T-test looks at a numerical variable that can be divided into twwo groups. 

The hypothesis is that there isn't a difference between Southern and Non-Southern states for confirmed covid cases in nursing homes.

The null hypothesis is that there is difference between Southern and Non-Southern states for confirmed covid cases in nursing homes.

First, I cleaned the columns of the data that I was looking for.
```
df2['Total Resident Confirmed COVID-19 Cases Per 1,000 Residents'] = df2['Total Resident Confirmed COVID-19 Cases Per 1,000 Residents'].dropna()
```
Then, I separated the data into two groups. One that looked at Southern states and the other, non-southern states.
```
df2['is_southern'] = df2['Provider State'].apply(lambda x: 'southern' if x in ['AL',
        'AR', 'DE', 'DC', 'FL', 'GA',
        'KY', 'LA', 'MD', 'MS', 'NC',
        'OKa', 'SC', 'TN', 'TX', 'VA',
        'WV'] else 'non-southern')
```
```
southern_data = df2[df2['is_southern'] == 'southern']['Total Resident Confirmed COVID-19 Cases Per 1,000 Residents']
non_southern_data = df2[df2['is_southern'] == 'non-southern']['Total Resident Confirmed COVID-19 Cases Per 1,000 Residents']
```
Then, I performed the T-test.
```
t_stat, p_val = ttest_ind(non_southern_data, southern_data, equal_var=False, nan_policy='omit')
print(f"T-statistic: {t_stat}")
print(f"P-value: {p_val}")
```
```
T-statistic: -52.560788818904456
P-value: 0.0
```
This low T-stat tells me that the groups are similar and have similar means. The p-value being < 05 tells me that the two groups are statistically significant. This tells me to reject the null hypothesis. There isn't a difference between southern and non-southern states for the prevalance of covid 19 in nursing homes. We can also look at the means to see that this is true.
```
southern_mean = southern_data.mean()
non_southern_mean = non_southern_data.mean()
print(f"Mean prevalence for southern states: {southern_mean}")
print(f"Mean prevalence for non-southern states: {non_southern_mean}")

Mean prevalence for southern states: 862.2756595966712
Mean prevalence for non-southern states: 800.2920560366094
```

# ANOVA
First, import the appropriate packages.
```
import statsmodels.api as sm
from statsmodels.formula.api import ols
```

For the ANOVA test, we choose a dataset that has one numerical variable and one categorical variable with more than two levels or groups. I chose a dataset that looked at the enrollment of a child health 
plus plan. 

Is there a significant difference between the mean values of enrollment for child health plus between Race?

Is there a significant difference between the mean values of enrollment for child health plus between Gender?

Is there a significant difference between the mean values of enrollment for child health plus between any combination of Race and Gender?

The hypothesis is that there is significant difference between race and gender for enrollment of child health plus plans.

The null hypothesis that that there is no significant difference between race and gender for enrollment of child health plus plans.

I looked at if Gender and Race affects the number of enrollments.
```
model = ols('enrollees ~ C(Race) * C(Gender)', data=df3).fit()
model
anova_table = sm.stats.anova_lm(model, typ=2)
print(anova_table)
```
```
                         sum_sq        df            F        PR(>F)
C(Race)            5.920157e+09       5.0  6216.421584  0.000000e+00
C(Gender)          1.367837e+07       1.0    71.814400  2.375483e-17
C(Race):C(Gender)  8.138360e+06       5.0     8.545631  4.200760e-08
Residual           5.302848e+10  278411.0          NaN           NaN
```
The large F-statistic says that the means of each group vary greatly from the overall mean group. At least one of the group means is vastly different from the others.

The p-values being less than 0.05 means we reject the null hypothesis. This means that there are statistically significant differences between the race and gender of child health plus enrollment.
We can see that this is true by looking at the mean rates.
```
df3.groupby('Race')['enrollees'].mean()
```
```
Race
Asian              100.428293
Black               97.979895
Hispanic           190.645307
Native American      4.315089
Other              387.262887
White              393.154588
Name: enrollees, dtype: float64
```
```
df3.groupby('Gender')['enrollees'].mean()
```
```
Gender
Female    223.220469
Male      235.878485
Name: enrollees, dtype: float64
```

# Regression Analysis
First, import the appropriate packages.
```
import statsmodels.api as sm
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
```
For a regression analysis, we choose a dataset with one numerical independent variable, and one numberical dependent variable. I chose a dataset that looked at Covid tests and rates in the NY State. 

The hypothesis is that there is a relationship between the number of covid tests and the number of positive cases. The more tests you conduct, the higher number of cases.

The null hypothesis is that there is no relationship between the number covid tests and the number of positive cases.

Looking at my dataset, I wanted to slim it down.
```
df4.columns
```
```
Index(['Test Date', 'County', 'New Positives',
       'Cumulative Number of Positives', 'Total Number of Tests Performed',
       'Cumulative Number of Tests Performed', 'Test % Positive', 'Geography'],
      dtype='object')
```
Using index number, I created a smaller dataframe.
```
df5 = df4.iloc[:, [3,4,6]].copy()
```
Then I proceeded to create variables for both. 
```
X = df5['Total Number of Tests Performed']
y = df5['Cumulative Number of Positives']
```
```
X = sm.add_constant(X)
model = LinearRegression().fit(X, y)
slope = model.coef_[0]
intercept = model.intercept_
r_squared = model.score(X, y)
print(f"Slope (Coefficient for Covid): {slope:.2f}")
print(f"Intercept: {intercept:.2f}")
print(f"R-squared value: {r_squared:.2f}")
```
As a result of performing the regression analysis, we saw this result.
```
Slope (Coefficient for Covid): 0.00
Intercept: 75508.94
R-squared value: 0.25
```
We can also make a summary model.
```
model2 = sm.OLS(y, X).fit()
```
This being the result.
```
 OLS Regression Results                                  
==========================================================================================
Dep. Variable:     Cumulative Number of Positives   R-squared:                       0.254
Model:                                        OLS   Adj. R-squared:                  0.254
Method:                             Least Squares   F-statistic:                 3.174e+04
Date:                            Sun, 08 Oct 2023   Prob (F-statistic):               0.00
Time:                                    21:06:48   Log-Likelihood:            -1.3545e+06
No. Observations:                           93294   AIC:                         2.709e+06
Df Residuals:                               93292   BIC:                         2.709e+06
Df Model:                                       1                                         
Covariance Type:                        nonrobust                                         
===================================================================================================
                                      coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------------------
const                            7.551e+04   1648.089     45.816      0.000    7.23e+04    7.87e+04
Total Number of Tests Performed    16.5639      0.093    178.145      0.000      16.382      16.746
==============================================================================
Omnibus:                   130171.097   Durbin-Watson:                   1.907
Prob(Omnibus):                  0.000   Jarque-Bera (JB):         35735138.991
Skew:                           8.305   Prob(JB):                         0.00
Kurtosis:                      97.430   Cond. No.                     1.83e+04
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 1.83e+04. This might indicate that there are
strong multicollinearity or other numerical problems.
```

In addition, we can create a scatter plot.
```
plt.scatter(df5['Total Number of Tests Performed'], df5['Cumulative Number of Positives'], label='Data Points')
plt.plot(df5['Total Number of Tests Performed'], model.predict(X), color='blue', label='Regression Line')
plt.xlabel('Total Number of Tests Performed')
plt.ylabel('Cumulative Number of Positives')
plt.title('Relationship between Number of Tests Performed and Number of Positives')
plt.legend()
plt.show()
```
![](https://github.com/jas-tang/datasci_5_statistics/blob/main/images/scatterplot5.JPG)

The coefficient being 0 indicates that there is no linear relationship between the number of COVID tests conducted and the number of positive cases.

The intercept of 75508.94 suggests that, when there are no COVID tests, the estimated number of positive cases is approximately 75,508.94.

An R-squared value of 0.25 means that approximately 25% of the variability. This means that the model only explains a small portion of the variation.

These results suggest that either this type of test does not fit my data, or that the higher the number of tests does not lead to a change in the number of positive cases.

