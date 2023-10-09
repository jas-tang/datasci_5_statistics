# datasci_5_statistics
This is a repository for Assignment 5 in HHA507. 

# Chi Square Test

First import the appropriate packeges
```
import pandas as pd
from scipy.stats import chi2_contingency as chi2
```

# The failed attempt
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
