
# Website A/B Testing

## Introduction

In this lab, you'll get another chance to practice your skills at conducting a full A/B test analysis. It will also be a chance to practice your data exploration and processing skills! The scenario you'll be investigating is data collected from the homepage of a music app page for audacity.

## Objectives

You will be able to:
* Analyze the data from a website A/B test to draw relevant conclusions
* Explore and analyze web action data

## Exploratory Analysis

Start by loading in the dataset stored in the file 'homepage_actions.csv'. Then conduct an exploratory analysis to get familiar with the data.

> Hints:
    * Start investigating the id column:
        * How many viewers also clicked?
        * Are there any anomalies with the data; did anyone click who didn't view?
        * Is there any overlap between the control and experiment groups? 
            * If so, how do you plan to account for this in your experimental design?


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style('darkgrid')
%matplotlib inline

df = pd.read_csv('homepage_actions.csv')
print(len(df))
df.head()
```

    8188





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>timestamp</th>
      <th>id</th>
      <th>group</th>
      <th>action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-09-24 17:42:27.839496</td>
      <td>804196</td>
      <td>experiment</td>
      <td>view</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-09-24 19:19:03.542569</td>
      <td>434745</td>
      <td>experiment</td>
      <td>view</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-09-24 19:36:00.944135</td>
      <td>507599</td>
      <td>experiment</td>
      <td>view</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-09-24 19:59:02.646620</td>
      <td>671993</td>
      <td>control</td>
      <td>view</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-09-24 20:26:14.466886</td>
      <td>536734</td>
      <td>experiment</td>
      <td>view</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.action.value_counts()
```




    view     6328
    click    1860
    Name: action, dtype: int64




```python
cids = set(df[df.action=='click']['id'].unique())
vids = set(df[df.action=='view']['id'].unique())
print("Number of viewers: {} \tNumber of clickers: {}".format(len(vids), len(cids)))
print("Number of Viewers who didn't click: {}".format(len(vids-cids)))
print("Number of Clickers who didn't view: {}".format(len(cids-vids)))
```

    Number of viewers: 6328 	Number of clickers: 1860
    Number of Viewers who didn't click: 4468
    Number of Clickers who didn't view: 0


> Comment: Everyone who clicked, also viewed the homepage! (Thank goodness!)


```python
eids = set(df[df.group=='experiment']['id'].unique())
cids = set(df[df.group=='control']['id'].unique())
print('Overlap of experiment and control groups: {}'.format(len(eids&cids)))
```

    Overlap of experiment and control groups: 0


> Comment: No overlap between the experiment and control groups.

## Conduct a Statistical Test

Conduct a statistical test to determine whether the experimental homepage was more effective than that of the control group.


```python
df['count'] = 1
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>timestamp</th>
      <th>id</th>
      <th>group</th>
      <th>action</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-09-24 17:42:27.839496</td>
      <td>804196</td>
      <td>experiment</td>
      <td>view</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-09-24 19:19:03.542569</td>
      <td>434745</td>
      <td>experiment</td>
      <td>view</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-09-24 19:36:00.944135</td>
      <td>507599</td>
      <td>experiment</td>
      <td>view</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-09-24 19:59:02.646620</td>
      <td>671993</td>
      <td>control</td>
      <td>view</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-09-24 20:26:14.466886</td>
      <td>536734</td>
      <td>experiment</td>
      <td>view</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Convert clicks into a binary variable on a user-by-user-basis
control = df[df.group=='control'].pivot(index='id', columns='action', values='count')
control = control.fillna(value=0)

experiment = df[df.group=='experiment'].pivot(index='id', columns='action', values='count')
experiment = experiment.fillna(value=0)



print("Sample sizes:\tControl: {}\tExperiment: {}".format(len(control), len(experiment)))
print("Total Clicks:\tControl: {}\tExperiment: {}".format(control.click.sum(), experiment.click.sum()))
print("Average click rate:\tControl: {}\tExperiment: {}".format(control.click.mean(), experiment.click.mean()))
control.head()
```

    Sample sizes:	Control: 3332	Experiment: 2996
    Total Clicks:	Control: 932.0	Experiment: 928.0
    Average click rate:	Control: 0.2797118847539016	Experiment: 0.3097463284379172





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>action</th>
      <th>click</th>
      <th>view</th>
    </tr>
    <tr>
      <th>id</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>182994</th>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>183089</th>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>183248</th>
      <td>1.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>183515</th>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>183524</th>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
import flatiron_stats as fs
```


```python
fs.p_value_welch_ttest(control.click, experiment.click)
```




    0.004466402814337078



## Verifying Results

One sensible formulation of the data to answer the hypothesis test above would be to create a binary variable representing each individual in the experiment and control group. This binary variable would represent whether or not that individual clicked on the homepage; 1 for they did and 0 if they did not. 

The variance for the number of successes in a sample of a binomial variable with n observations is given by:

## $n\bullet p (1-p)$

Given this, perform 3 steps to verify the results of your statistical test:
1. Calculate the expected number of clicks for the experiment group, if it had the same click-through rate as that of the control group. 
2. Calculate the number of standard deviations that the actual number of clicks was from this estimate. 
3. Finally, calculate a p-value using the normal distribution based on this z-score.

### Step 1:
Calculate the expected number of clicks for the experiment group, if it had the same click-through rate as that of the control group. 


```python
control_rate = control.click.mean()
expected_experiment_clicks_under_null = control_rate * len(experiment)
print(expected_experiment_clicks_under_null)
```




    838.0168067226891



### Step 2:
Calculate the number of standard deviations that the actual number of clicks was from this estimate.


```python
n = len(experiment)
p = control_rate
var = n * p * (1-p)
std = np.sqrt(var)
print(std)
```

    24.568547907005815



```python
actual_experiment_clicks = experiment.click.sum()
z_score = (actual_experiment_clicks - expected_experiment_clicks_under_null)/std
print(z_score)
```

    3.6625360854823588


### Step 3: 
Finally, calculate a p-value using the normal distribution based on this z-score.


```python
import scipy.stats as stats
p_val = stats.norm.sf(z_score) #or 1 - stats.norm.cdf(z_score)
print(p_val)
```

    0.00012486528006951198


### Analysis:

Does this result roughly match that of the previous statistical test?

> Comment: Yes, while the p-value is slightly lower, both would lead to confident rejection of the null hypothesis. The experimental page appears to be a more effective design.

## Summary

In this lab, you continued to get more practice designing and conducting AB tests. This required additional work preprocessing and formulating the initial problem in a suitable manner. Additionally, you also saw how to verify results, strengthening your knowledge of binomial variables, and reviewing initial statistical concepts of the central limit theorem, standard deviation, z-scores, and their accompanying p-values.
