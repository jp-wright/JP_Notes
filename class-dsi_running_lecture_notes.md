# JP DSI LECTURE NOTES:

## <p style="text-align: right;"> Week 1 </p>

### Week 1, Day 1 -- Aug. 29
(Fill in first week of notes)


Homebrew
`brew list` shows all command-line apps installed via homebrew
`brew cask list` shows all GUI apps installed via homebrew.



Plotting in MPL:
plt.style.available #lists all available styles
plt.style.use('style_chosen')

### Week 1, Day 2 -- 8.30.16 (SQL Notes)

##### Entity Relationship Diagram - ERD

+ Primary Key (PK)
  + The main column header, identifies all the data in the column beneath it, in a specific table.   
  + When a PK is referenced in a separate table it is known as a Foreign Key (FK) to the separate table.
    + Note: when a PK is used in another table that has its own PK with the exact same name as the FK, it must be specified explicitly in reference by using the `table-name.FK` method.  
  + The most common relationship between multiple tables (databases?) is a "one-to-many" relationship.  The notion of "many-to-many" is highly complicated and is, for most practical purposes, going to be avoided.


##### Terminal and PSQL

+ In terminal, type `psql` to begin a PSQL session, then can do a command-space search for "postgress", open it, and go from there.  Opening Postgres without having PSQL running will fail to open Postgres.  
+ To exit PSQL use `\q`



##### Aggregate functions

When using an aggregate function, such as `COUNT()` or `AVERAGE()`, you have to include the remaining columns in the `SELECT` statement that aren't part of the aggregate functions in a `GROUP BY` statement.  Period.  

Note you can do an `ORDER BY`, a `GROUP BY`, or any other aggregate wrap-up function on a column you aren't `SELECT`ing for.  I'm not sure why you'd want to do this, but it is valid.

`HAVING` is used basically as a `WHERE` statement for aggregate functions.  For example, you can't do a query with a `WHERE` clause asking for cells that have a `COUNT` of 5, say.  It will crash (unless there is a Primary Key column of the count already listed).  You can't do a `WHERE` on an aggregate calculation made in the `SELECT` statement.  So, use `HAVING` a `COUNT` = 5, instead.


##### Creating SQL databases

[www.sqlfiddle.com](SQL Fiddle) to create databases.  You can also copy and paste entire files, like .csv files, into a database builder and it will create a SQL database from the stuff you pasted.  Neat!  Great for me and NFL spreadsheets...



#### PSQL and Postgres
Use postgres on your laptop.
**Follow instructions in Week1 / SQL / individual.md** and/or **pair.md**

In terminal, type `psql` to start up psql.  You MUST have Postgres running for this to all work (cmd+space search for postgres and start it).  Once in psql on terminal, create database as instructed above, then you'll have the database on your actual machine.


To create a temporary table, do the following:
```sql
SELECT userid, date_7d
INTO TEMPORARY TABLE logins_mob
FROM logins_7d JOIN logins ON logins_7d.userid = logins.userid
WHERE logins.type = "mobile"
```

or what have you. Enter this *into psql terminal*, where it will live as long as the session is open, allowing you to reference it as you need, including creating permanent tables from info taken from the temp tables.

#### psycopg2 -- editing SQL within Python via 'Pipelines'
You will edit stuff all you want in psql terminal and then once ready you will enter it into terminal (or ipython).  You must create a connection using psycopg2, then a cursor for that connection.  You must enclose the SQL query as a cursor.execute() argument, then once done you have to actually commit() the query before the actual database will be modified.  At the end you must close the connection.   If you have ANY ERROR AT ALL in any c.execute() command/query, you MUST use `conn.rollback()` to clear the cursor, as it has been 'tainted' by the error.

See "postgres_script.py" and /sql-python/individual.md for more info.

```python
import psycopg2
from datetime import datetime

conn = psycopg2.connect(dbname='socialmedia', user='postgres', host='/tmp')
c = conn.cursor()

today = '2014-08-14'

timestamp = datetime.strptime(today, '%Y-%m-%d').strftime("%Y%m%d")

c.execute(
    '''CREATE TABLE logins_7d AS
    SELECT userid, COUNT(*) AS cnt, type, timestamp %(timestamp)s AS date_7d
    FROM logins
    WHERE logins.tmstmp > timestamp %(timestamp)s - interval '7 days'
    GROUP BY userid;''', {'timestamp': timestamp}
)

conn.commit()
conn.close()
```


## <p style="text-align: right;"> Week 2 </p>
***

### Week 2, Day 3 -- Sep. 8
Understanding *why* we use certain distributions to model our data is the important key.  It isn't important to memorize every aspect about every dist.  We can look that up.  We need to know have a *feel* for the dists.

If we don't know the parameters of an obersved dist, we can estimate them using various techniques.  The two we are covering are:
+ Model of Moments (MOM)
+ Older, but still effective in general.

+ Maximum Likelihood Estimation (MLE)
+ Ubiquitous method in general data science and statistics for estimating parameters.  See slides from Week 2 for more detail.  In short, it is a cost function and attempts to maximize it for your data points and given parameter.

**Sampling Distribution**  
The distribution of a repeatedly sampled parameter from a parent population (or parent sample).
So, given a parent population N, take a sample size A from the pop and compute the mean of that sample.  Repeat this X number of times, and then plot the distribution of these means.  This is now the *sampling distribution* of the mean for that pop.  Aggregate functions, like the mean, will obey the Central Limit Theorem, more clearly so as X number of repeat samplings increases.  Other summary statistics, like the sample max(), do not obey the CLT -- their sampling distribution is strictly dependent upon the underlying parent population distribution.  This can help identify some aspects of the parent distribution, but makes other inferences about the parent population impossible since those inferences, such as the true mean of the population, depend on the CLT to be correct.


**Hypothesis Testing**  
Basically, "prove it." The null hypo is always the 'standard' or the 'normal' mode, and the alternative hypo is the 'claim' you are making.  Think your coffee makes workers more productive than the 'normal' coffee they're already using? Prove it.  Think your medicine works better than the 'normal' one? Prove it.  Using confidence levels allows us to say how certain we are something is 'proved' if the hypo test says it is.

A quick rule of thumb for determining if the variances of the two samples used in the hypo test is to take the ratio of the larger var to the smaller.  If it is less than 2, probably okay to go ahead and test without variance modification (Welsh t-test assumes diff variances, and can also pool variances).

Note that by the design of hypothesis testing, we are more likely to fail to reject the null than is correct, much like "innocent until proven guilty" in our judicial system -- the onus is upon the alt. hypo to confirm validity ('guilty').


**Five Steps to a Hypothesis Test**
1. State Null and Alt. hypotheses
2. Calculate Test Statistic based on distribution(s) of stat(s).
3. Obtain p-value.
+ Probability of getting your data (or something more extreme) given that the null hypo is true.  Smaller = less likely to get the observed results by mere chance.  
4. Compare p-value to α threshold.
+ If p-val is less than a pre-set threshold, called the α level, then we can say that the observed result is real/true/valid at an α confidence level, such as 95%.  A good way to think of α is as the Type I Error Rate, which is the amount of error (False Positives) you are willing to tolerate.  So, an α = .05 means we would be willing to accept a failure rate of 5%, assuming the hypothesis test passed.
5. State conclusion in terms of the problem.
+ Properly stated as either we "reject" or "fail to reject" the null hypo.  Assuming p-val < α, mathematically speaking, we haven't absolutely 'confirmed' the alt. hypo.  Other explanations for our observed data may still exist, but we can say with α confidence that, whatever the source, the data we observed is not due to random chance.  When using multiple hypo tests, say 20, we should divide the α value by the number of tests performed, and use that as our new α threshold.  So, say α = .05 and we did 20 tests, we would use .05/20 = .01 as our new α value.

α: Type I Error (False Positive)
β: Type II Error (False Negative)
1 - α: Confidence Interval
1 - β: Power  


+ Example: I have three versions of a website. I want to know which version is the best to use in practice.

Version | Users | Purchases | Clicks
--------|-------|-----------|-------
1       | 300   | 20        | 100
2       | 400   | 15        | 100
3       | 350   | 30        | 150

+ null: V1 = V2 = V3
+ alt: at least 1 site is diff
+ 3 tests = 3-choose-2 = 3
+ Modified α = .05 / 3 =
+ We will take the
+ There is a raw approach to this since we do have actual real data:
1. Take a sample from each of the three  versions
2. Bootstrap that sample for each version
3. Subtract the statistic from Version A from Version B (i.e. mean A from bootstrap A from mean B from bootstrap B)
4. Repeat steps 2 & 3 many times
5. Plot the resulting sampling distribution of differences
6. Throw away the top 2.5% and bottom 2.5%
7. We now have the inner 95% range, and can say we expect the true mean for the given version to lie within this range at a 95% confidence level
8. IF this range contains zero (i.e. if the number zero is in our 95% plot range) then we say that there is no observed difference between these two versions.
9. The advantage of this is that it is real data and is not based upon any assumption about the underlying distribution, so no asymptotic assumptions (or equations) need be used.



### Week 2, Day 4 -- Sep. 9: Statistical Power and Bayes
**Two-sided test**: when the hypothesis is *not equal* to something  
**One-sided test**: when the hypo is *greater than* or *less than* something  
**Sensitivity**: true positive rate  
**Specificity**: α rate.  Amount of allowed Type I error we are willing to accept.  
**Power**: the probability that we **correctly reject null** when the null hypo *is* false. (1 - β).  

Symbolically:  
**α**: Type I Error - Reject null when null is True (false positive)  
**β**: Type II Error - fail to reject null when null is false (false negative)  
**(1 - β)**: Power - reject null when null is false  


The calculation of *Power* is not trivial because it is different for every test.
See *power-calculation.ipynb* in `DSI_Camp/Week2/power-bayesian/` for interactive calcs on Power and how diff variables affect the result. (neat: bigger changes are easier to detect, so smaller num of samples are required to observe.)
A common level of Power used is 80%.

For Power in Python:
```python
import scipy.stats as st
st.norm.ppf(alpha)
st.norm.ppf(1 - beta)
```

More:
```python
import scipy.stats as stats
## inverse of the cdf (percent point function)
alpha = 0.05
z = stats.norm.ppf(1-(alpha/2.0))
print("z=%s"%(round(z,2)))

## find a critical value z
phi_z = stats.norm.cdf(z)
print("phi_z=%s"%(phi_z))
```


In general, R handles Power calculations more readily than does Python at present.  As such, the power-calculation.ipynb named above has some examples of how to use R for Power calcs, including the package (pwr) to install in R as well as the Python package (rpy2) needed to access R via Python.  Do not install via pip, install via `[unknown atm]`

1. `brew install r`
  + can do `brew install r --reinstall` if needed to redo.  Installing R can take quite a while.

2. `brew install Caskroom/cask/rstudio`
  + install the GUI component of R (not required but recommended)

3. `pip install rpy2`
  + install the python module to interface with R

4. `sudo R` then `install.packages('pwr')`
  + access R and install the 'pwr' package for Power calcs.




##### Bayes
Priors often are distributions and not merely a single value.
Commonly we evaluate Bayes theorem by P(H|E) proportional to P(E|H) * P(H) after considering the denominator of P(E) as a normalizing constant.

## <p style="text-align: right;"> <span style="color:purple"> Week 3 </span> </p>
***
### Week 3, Day 1 -- Sep. 12: Linear Algebra

`np.reshape` - returns a copy of the vector  
`np.resize` - changes shape in place


### Week 3, Day 2 -- Sep. 13: Linear Regression

In most of our Machine Learning we will focus on Supervised learning.  

+ Supervised
  + Target you want to predict (predict a *y* for a given *x* (usually a large dataframe))
+ Unsupervised
  + No target
+ Semi-Supervised
  + Some targets are labeled, some are unknown.  Not as common.


+ Five Assumptions of Linear Regression:
  1. X and Y are linearly related (Note: this does not mean X var has to be linear -- can be log or exponential, just the relationship has to be linear)
  2. Homoscedasticity of residuals (variance is equal across the range of values)
  3. Normally distributed residuals
  4. No multicolinearity among features (Variance Inflation Factors - **VIF**)
    + VIF > 10: suggests pulling features out of regression
    + VIF > 5: review this term to check for colinearity with another term of similar VIF.
    + Regularization methods are the most common for choosing which terms to keep.
  5. Independence of the observations (For example, independence assumption violated if data is a time series)
    + Dependent variable (y) is continuous, Independent variable (x) can be continuous or discrete
    + Categorical values (names, field, etc.) will be converted to *dummy variables*

Understanding what's going on under the hood of regression is a major factor in standing apart from other candidates as it is a frequent interview topic.  One thing to know: we use the Least Sum of Squares -- i.e. we square our errors and take the root -- because it **has a clean, computable derivative which allows us to use the gradient descent method of finding the smallest error value.**  This means that squaring our residuals in the OLS (LSS) method is computationally viable and has an easy segue to to further methods of error optimization. Also, OLS is built in to just about every piece of statistical software, meaning we don't have to write our own method.  Knowing why we are doing what we are doing is as important as being able to implement the methods themselves in some ways.

Use a canonical formula (such as a log() for Poisson) to link the function of the parameters for a distribution to a linear combination of inputs.

**Dummy Variables**
+ Set first to 0
+ Set last to 0
+ 1, 0, -1

+ Our matrix X must be full rank to be invertible.
  + B = (X' X)^- X'y

Use `pd.get_dummies( , drop_first=True)`   to get dummies and drop the old cols.
If the categories for dummy vars don't necessarily contain all data points (such as not being High / Med / Low) and a data point is NaN, then treat NaN as an additional category.

Use AIC, BIC, Cp, R^2, and p-vals to determine if you should drop data or not.  Of these, R^2 and p-vals are the most spurious and potentially misleading, making relying upon them alone a bad idea.  Cp might have the most predictive power at times, but BIC tends to be the most preferred.


### Week 3, Day 3 -- Sep. 14: Regularization and Bias v. Variance
#### Bias v. Variance
Bias v. Variance can be thought of much like Accuracy v. Reliability in science.

<div align="center">
    <img height="400" src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week3/regularized-regression/images/bias_variance.png">
</div>

<div align="center">
    <img height="400" src="images/visual_bias_variance.png">
</div>



+ Bias (Accuracy):
  + High Bias = **Underfitting** and **Low complexity**
  + Bias can be thought of as how accurate we predict/model the true signal, the true data points.  With increased accuracy we have less bias.  

+ Variance (Reliability)
  + High Variance = **Overfitting** and **High complexity**
  + Variance is how much our model's results will change with different input data.  If we change our input data/signal and our output is consistent, even if it is wrong, it has low variance -- it is reliable (even if reliably wrong).  

We want LOW bias and variance!  But we can't have arbitrarily low measures in both, because they're related.

The big conundrum is balancing Bias v. Variance. If we increase variance by adding model complexity we can *overfit* the model to *that particular data*, then our model will be unreliable with *different data* (from the same population).  We ideally would love very low bias and variance, of course, but outside of instructional simple systems, this is not likely achievable.  As with any system of tradeoffs, there is a sweet spot (i.e. a local minimum, see below) where we have as low of a bias and variance as is reasonable.

<div align="center">
    <img height="400" src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week3/regularized-regression/images/bias_variance_graph.png">
</div>


#### Cross-Validation
Must be done *completely randomly*.  When choosing the training set and the test set, we cannot filter these sets by using any knowledge, such as which features carry the most impact.  It must be done randomly, before the data is examined.  

Four Fundamental Steps:
1. Split your data into training/validation sets.  
    + 70/30, 80/20, or 90/10 splits are commonly used
2. Use the *training set* to train several models of varying complexity.
    + e.g. linear regression (w/ and w/out interaction features), neural nets, decision trees, etc. (we’ll talk about hyperparameter tuning, grid search, and feature engineering later)
3. Evaluate each model using the validation set.
    + calculate R2, MSE, accuracy, recall, etc. or whatever you think is best
4. Keep the model that performs best over the validation set, with the belief it will perform best on the test set.


The K-folds approach is common and results in data and errors to graph.

Using the "Leave-One-Out" (not K-folds) approach is computationally expensive and by design gives a low-variance model, both of which we want to avoid (low variance is good, but not when it is systemically forced).

Always remember that as dimensionality increases (more features), overfitting a model becomes naturally easier.  Have to work against overfitting by trying to select subsets of the features, usually starting with one and gradually adding in other important ones.  The best model is a *lean* model -- as few features as possible that give the desired accuracy and reliability.  Don't add ingredients to the recipe if the recipe already does what you want.

#### Regularization
The fundamental approach of Regularization is to limit the magnitude of the coefficients to limit overfitting.  As we increase the number of features/terms in a regression, the magnitude of each tends to increase (sometimes greatly).  As a result, limiting the magnitude of these terms' coefficients will help to reduce overfitting.

In Regularization models like Ridge (L2) or Lasso (L1), the λ term is the 'Regularization' parameter, and acts as a tuning knob for the coefficients -- how loose will we let them be?
<div align="center">
    <img  src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week3/regularized-regression/images/lasso.png">
</div>
<div align="center">
    <img  src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week3/regularized-regression/images/ridge.png">
</div>

Smaller values (all negative) of λ mean we have a more complex model and let the coefficients have a broader range, larger values (up to zero) damped coefficients more strenuously.


Note Lasso will drop terms (at times arbitrarily) and Ridge will not.  Elastic Net will drop terms, similar to Lasso, but due to its Ridge component it doesn't do so ad hoc.  It's a mix of the best parts of both Lasso and Ridge.  Josh personally would never use Lasso over Elastic Net, his preferred regularization method.



### Week 3, Day 4 -- Sep. 15: Logistic Regression

odds = p^k * (1-p)^n-k

so p = 1/100 and q = 99/100: 1/100 / 99/100 = 1/99
<div align="center">
    <img  src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week3/logistic-regression/images/logistic.png">
</div>

Classification Metrics II:  
+ **Recall** - *True Positive Rate* (AKA "**Sensitivity**")
  + Of those observations that are actually positives, which ones did I label as positive?
  + True Positives / (True Positives + False Negatives)

+ **Precision** - *Positive Predictive Value*
  + Of those observations that I labeled as positive, which ones are actually positive?
  + True Positives / (True Positives + False Positives)

+ **Specificity** - *True Negative Rate*
  + Of those observations that are actually negative, which ones did I label as negative?
  + True Negatives / (True Negatives + False Positives)

+ **Accuracy**
  + How many observations did I label correctly?
  + (True Positives + True Negatives) /  Total number of obs.
  + Highly susceptible to class imbalance - *do not rely on as sole metric!*

+ **False Positive Rate**
  + Of those observations that are actually negatives, which ones did I label as positive?
  + False Positives / (False Positives + True Negatives)

+ **F1 Score** (or just **F Score**)
  + A weighted average of precision and recall.
  + Max score of 1, min of 0.
  + Can be susceptible to class imbalance, as well, I believe (study more).
  + 2 * (precision * recall) / (precision + recall)

+ **Average Precision Score**
  + Compute average precision (AP) from prediction scores.  This score corresponds to the area under the precision-recall curve.
  + This can be similar to F1, I believe, and F1 is certainly the more popular one.  I haven't studied either in detail to see how they differ.

+ **Kappa** (or **Cohen’s Kappa**)
  + Classification accuracy normalized by the imbalance of the classes in the data.

+ **ROC Curves - AUC** (Receiver Operator Characteristic)
  + AUC stands for "area under the curve". Like precision and recall, accuracy is divided into sensitivity and specificity and models can be chosen based on the balance thresholds of these values.  See below for more on ROC.
  + Locate a given threshold of confidence/cutoff on the ROC curve by finding the recall (TP) for that cutoff.


Confusion Matrix:
Traditionally written as:
              Pred. Pos | Pred. Neg    
             |----------|----------|
True Positive|     tp   |   fn     |
             |----------|----------|
True Negative|     fp   |   tn     |
             |----------|----------|


SKLearn flips the matrix.
Can flip it back to be represented as above with:
```python
#Flip the CMAT TT and FF quads to match standard use, with
cmat[0][0], cmat[1][1] = cmat[1][1], cmat[0][0]
cmat = cmat.T
print cmat
```

### Week 3, Day 5 -- Sep. 16: Gradient Descent
All day pair work.


## <p style="text-align: right;"> <span style="color:black"> Week 4 </span> </p>

### Week 4, Day 1 -- Sep. 19: K-Nearest Neighbor (KNN)
Non-parameterized algorithm.  Unlike parameterized regression models, KNN does not assume anything about the data (e.g. no underlying distribution, etc.)
Curse of dimensionality is rife with Non-parameterized models.

There is no actual 'training' for this model.  You simply plot your data and measure the distances bww the points, sort by distance, and classify.

Rule of thumb is sqrt(N) for sample size, but use Power calcs from last week for more precision on power of sample size.

When using categorical features, such as genres of music, must be aware of *exchangeability*, which is the quandary of ordering/assigning genres in a manner that suggests one is more closely related to another (or more preferable).  Ex: order of R&B, Country, Rock, EDM, Folk, Classical would get values of R&B = 1, Country = 2, ..., Classical = 6.  But Why?  And what effect does that have on the resulting clustering?  So, this is a real problem.

KNN is a good technique for data imputation (i.e. replacing missing data)

### Week 4, Day 2 -- Sep. 20: Random Forest
Random Forest is simply Bagging (Bootstrap Aggregating) with one tweak:
the number of predictors chosen to split on at each node of the tree is the square root of the total number of predictors in the dataset.  So, if we have 9 predictors to choose from, each node will randomly choose 3 of them to split on.  This ensures no one predictor dominates the splits.  So to be clear, we Bag the trees then do a Random Forest (n predictors per split = sqrt(p total predictors))

Note if we did not Bag and RF, but still made 100 trees, we would simply make 100 identical trees because they'd all have the same dataset and same predictors, and as such would choose to split on the same predictors each time.  So, Bootstrapping and RF give us a much better statistical look at our data given the same initial dataset.

When testing make sure to specify `random_state = <constant>` so that repeated trials give the same results of splits.

1. From training set, bootstrap -- gives one tree from Bagged data
2. Do first split, use subset of sqrt(predictors) on every split, giving a Random Forest model

For regression we are trying to minimize RSS via RF
Can use `sklearn.utils.resample` to help with bootstrap.


### Week 4, Day 3 -- Sep. 21: SVM
SVMs are really good if the data is truly separable, otherwise logistic regression will outperform it.
Sweet spot of data size is about 8,000-12,000 rows/data points.  Otherwise it is suboptimal.
Never use SVM for regression, even if it is an option in SKLearn.  It is not really made for this.
The most common kernel is Radial Base Functions (RBF)

### Week 4, Day 4 -- Sep. 22: Boosting & Neural Network
After covering multiple algorithms, here is a table reviewing some of their strengths and weaknesses:

<div align="center">
    <img  src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week4/boosting/images/Learning_algorithm_comparison.png" height="500" width="500">
</div>


The deeper the tree in a Decision Tree the less biased it is (it is fitting the training data better) but the higher variance (less applicable to other data).  This is the point of Random Forest, as it lowers variance with multiple random iterations of trees.  Obviously we want low bias and low variance, but the improvement in variance is far higher than the accompanying increase in bias in a Random Forest, so it is a significantly net-positive method.

We will be using Trees (but NOT Random Forest) for our exploration of Boosting.
We can introduce some variation into our trees somewhat akin to RF, but not quite the same (we don't sample with replacement, aka Bootstrap).  By building in this randomness we create what is known as Stochastic Gradient Descent, which has been shown to be more effective than traditional methods, including Random Forest (which has the benefit of simplicity and limited need of tuning) in many cases.


 Random forests and gradient boosting methods are two extremely popular methods in machine learning. Random forests are considered nice due to the low amount of tuning necessary to get ‘good’ results. Alternatively, gradient boosting takes some tuning time, but will often perform better with the right mix of parameters. For gradient boosting, we will frequently use sklearn’s **GridSearchCV** to find the right mix of parameters.


Note: n_jobs=-1 allows the process to be run on multiple cores on your computer. Only parallel algorithms such as RandomForest can do that. Boosting is sequential (the current step depends on the residuals from the previous) and does not have that option. n_jobs=-1 is not a hyperparameter.



### Week 4, Day 5 -- Sep. 23: Group Uber Churn Project
Group project.


## <p style="text-align: right;"> <span style="color:black"> Week 5 </span> </p>

### Week 5, Day 1 -- Sep. 26: Profit Curves and Class Imbalance
If you have a minority class, make sure it’s represented in the same proportion in your y_train and y_test datasets using `stratify`:
`X_train, X_test, y_train, y_test = train_test_split(X, y, stratify=y)`

>This `stratify` parameter makes a split so that the proportion of values in the sample produced will be the same as the proportion of values provided to parameter `stratify`.  
>For example, if variable y is a binary categorical variable with values 0 and 1 and there are 25% of zeros and 75% of ones, `stratify=y` will make sure that your random split has 25% of 0's and 75% of 1's.


Steps to compute Expected Profit:
1. Estimate Error Probabilities
2. Define the cost-benefit matrix
3. Combine probabilities and respective payoffs

**Sampling techniques**:  
*Undersampling* randomly discards majority class observations to balance training sample. Not likely to be useful unless the dataset is very large.
+ PRO: Reduces runtime on very large datasets.
+ CON: Discards potentially important observations.

*Oversampling* replicates observations from minority class to balance training sample. It is often better to use SMOTE.
+ PRO: Doesn’t discard information.
+ CON: Likely to overfit.

*SMOTE* - **S**ynthetic **M**inority **O**versampling **TE**chnique \**
+ Generates new observations from minority class.
+ For each minority class observation and for each feature, randomly generate between it and one of its k-nearest neighbors
+ [Here's a SMOTE module](http://contrib.scikit-learn.org/imbalanced-learn/generated/imblearn.over_sampling.SMOTE.html) for python ([here's its github repo](https://github.com/scikit-learn-contrib/imbalanced-learn)), though I've never used it.  

<br>
Relevant articles to class imbalance and metrics:
[Short guide to classification metrics](http://machinelearningmastery.com/classification-accuracy-is-not-enough-more-performance-measures-you-can-use/)
[Short guide to ROC Curves](http://machinelearningmastery.com/assessing-comparing-classifier-performance-roc-curves-2/)
[General tactics to fight class imbalance](http://machinelearningmastery.com/tactics-to-combat-imbalanced-classes-in-your-machine-learning-dataset/)


#### Week 5, Day 2 -- Sep. 27: Web Scraping & Mongo DB
Adam says Mongo is very good for any situation where you are unsure if you will be adding to or changing the database frequently.  Mongo allows you to simply create new spreadsheets or documents, etc., and simply dump them into the database.  Yes, this creates redundancy, but that is the cost of the vastly increased flexibility of Mongo.  


Using a VPN and tor make tracking and blocking your IP much more difficult for websites.
Gov Data sites will blacklist you if you request too much info. They have FTP sites where you can download huge data dumps.  Many sites will flag you if you ping their servers too frequently or request too much data, messing with their bandwidth.  Be patient, scrape slowly.

`from unidecode import unidecode` can help decode text formatting

##### <p style="text-align: center;"> <span style="color:red"> NEVER PASTE YOUR ACTUAL API KEY INTO ANY OF THESE .md OR .py FILES THAT YOU UPLOAD TO GITHUB (A PUBLIC DOMAIN) BECAUSE PEOPLE SCRAPE FOR YOUR KEYS AND STEAL THEM </span> </p>  

**Steps to using API**
1. Open your bash profile: `atom .bash_profile` (Must be done in bash/terminal)
2. Export your API keys for given API (keywords must be all caps and no space around `=` sign. Very particular.):  
  ```python
  export YELP_KEY=[actual key]
  export YELP_SECRET=[actual key]
  export YELP_TOKEN=[actual key]
  export YELP_TOKEN_SECRET=[actual key]
  ```
3. Save the file
4. Back in terminal, type: `source .bash_profile`
5. Confirm if file was saved, type: `echo $YELP_KEY`

#### Week 5, Day 3 -- Sep. 28: Natural Language Processing (NLP)

+ Text as Data
+ Definitions
+ Text Processing Steps
+ Text Vectorization:
  + Stop WOrds
  + Text Stemming / Lemmatization
  + POS Tagging
  + N-grams
+ Document Simimlarity

**Core Words**
+ Token - simply, a word
+ Document - a string of words
+ Corpus - collection of documents
+ Stop Words - words that don't matter

**Tokenize** - parse data, turning independent words into 'tokens'.
**Stemming** - chops off suffixes etc of words to create groups of the same word (i.e. run and running are now both 'run').  Stemming can be too aggressive, as such we prefer Lemmatization
**Lemmatize** - to intelligently 'stem' words by grouping them by context as well.  For example, "car", "cars", "trucks" etc. might all be grouped as "automobile."  This is the standard approach to modern NLP, but isn't perfect as it can mis-categorize at time.

We do this to prepare the data for the TF-IDF matrix.


Will use a **TF-IDF** (Term Frequency - Inverse Document Frequency) matrix.  Highly weighted words are indicative of words that are unique or important to that document.

See Erich's NLP PDF in Week 5/nlp/ (note that it is a PDF and it won't open in Atom): (/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week5/nlp/nlp_overview_erich.pdf)

Naive Bayes is used almsot exclusively for text analytics.


Mongo and everything in today's assignments were really poor.  Badly written and insufficient to students.  Will note such in my weekly survey.

#### Week 5, Day 4 -- Sep. 29: Time Series

Historically, R is noticeably better than Python for handling time-series.  It has more robust packages and an easier user interaction.  However, Adam says there is a fairly recent package in Python which he has greatly warmed to.  Adam says PyMC is so good that it has him working in Python on a daily basis instead of his previous, more comfortable packages in R (ARIMA, ETS).  He is choosing to optionally introduce it to us, and urges us to become familiar with it.  If he backs it, then I think that says a lot for its viability.  For our future as data scientists, some familiarity with R's time-series capabilities is probably wise.  However, learning the concepts of time-series via PyMC should be considered a valid approach.




Perhaps the most famous time-series graph of the last 50 years is the (in)famous *Hockey Stick* graph, made famous by Sen. Al Gore.


<div align="center">
    <img  src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week5/time-series/images/hockey-stick.jpg" height="420" width="620">
</div>



+ ARIMA - Autoregressive Integreated Moving Average
  + ARIMA models estimated following Box-Jenkins approach
  + p - order (number of time lags) of the autoregressive model
  + d - degree of differencing (number of time data have had past values subtracted)
  + q - order of the moving-average model


**Deterministic Trend** - get the same result each time given the same input
**Stochastic Trend** - not certain to get same result given same inputs, i.e. 'semi-random'

Check "Time Series Analysis in Pandas with statsmodels" paper by Wes McKinney for a good reference (ideally only the firs three pages or so).

Adam has included python code for how to make ACP and PACF plots in his time_series.pdf file.  Check out his 'Resources' slide at the end of his PDF for good refs.




**PyMC and Bayesian Modeling of Time-Series Data**


#### Week 5, Day 5 -- Sep. 30: Regression Case Study

Adam's tips on group projects:
1. Make a Game Plan.  Talk about data, assignments, models, goals.
2. Set a certain amount of time to trying to produce your goals
3. Meet and wrap-up, discuss what worked and didn't and what might need to be cut out.

Files:
1. mylib.py - all the code, plotting, models, etc.
2. run.py - import mylib.py (all of it), and run it effectively and concisely.  Should not take a long time to run. This is the file you spend most of your time on -- this is the file that we want to go through our Game Plan, in order.
3. deliverable.ipynb/.md

Game Plan:
1. EDA
2. Exploration
3. Custom Model
4. Future Importance



### <p style="text-align: right;"> <span style="color:black"> Week 6 </span> </p>

#### Week 6, Day 1 -- Oct. 3: Clustering

**K-Means**  

For K-Means we initialize the dataset with a random assignment to a given cluster.  From here, we iterate over the dataset and continually improve our grouping.  This random initialization *does* impact our final groupings' probabilities.  As a result, we have to run the entire K-Means clustering algorithm multiple times and take the aggregate of the cluster results to find the **global optimum**.  There is an alternative, however, called *K-Means ++*, which does not randomly initialize.  Instead it tries to initialize into groups intelligently, which ideally lets us find our global optimum more quickly, without multiple (or as many) iterations.  SKLearn defaults to using K-Means ++.


How do we calculate the score of a given data point to know which cluster it belongs to?
+ Elbow Method
  + Results basically form a negative log curve, so you get diminishing returns of accuracy with each increase in number of K clusters used.  Find the optimum value of K at which the gain in accuracy begins to level off.  The best value of K is entirely data dependent.  Can be 2, can be 50.  Depends on what data you are examining.  

+ K-Means silhouette coefficient
  + a = mean of intracluster distances from the data point in consideration
  + b = mean of next-closest cluster distances from the data point in consideration
  + S = mean(b - a / (max(a, b)))  [note: mean over all pts in the cluster]
  + Ideally *b* is large (each cluster is far away from one another) and *a* is small (the clusters are tight-knit and for a distinct group) and our range goes from -1 (worst) to 1 (best).  0 means there is full/significant overlap in clusters.

Can "stack" clusters, i.e. cluster a cluster once all the initial clusters are grouped -- but only if you need to further examine/divide a given cluster.

**Practical Considerations for K-Means**
+ Standardize features? Based on a distance metric, so yes.
+ Susceptible to outliers? Euclidean distance, so yes.
+ Susceptible to Curse of Dimensionality? Euclidean distance, so yes.
+ How to handle categoricals? Look into k-medoids, kmodes (https://pypi.python.org/pypi/kmodes/).
+ Fast! (MiniBatchKmeans for large datasets). But finds local minima. (try kmeans++).
+ How many clusters, k?
+ Other clustering algorithms available, e.g. DBScan


**Hierarchical Clustering**  

Avoid making the mistake of assigning importance to horizontal distances in dendrograms.  All information is *vertical* in dendrograms.  This can be visually deceptive as there are horizontal distances.





#### Week 6, Day 2 -- Oct. 4: Dimension Reduction

Out with Alex and Chloe


#### Week 6, Day 3 -- Oct. 5: Topic Modeling

Discussing capstone topics.


#### Week 6, Day 4 -- Oct. 6: Recommender Systems


#### Week 6, Day 5 -- Oct. 7: Recommender Case Study




### <p style="text-align: right;"> <span style="color:black"> Week 7: Break Week </span> </p>

#### Week 7, Day 1 -- Oct. 11: Multi-Arm Bandits  
How do frequentist A/B tests work?
+ Run all experiments and observe the data
+ Significance of the result depends on how many experiments you run (N is a parameter)
+ Does not tell you how likely it is that A is better than B, just that you are confident A is better than B at some significance.  It simply tells you that A is or is not better than B.


Bayes:
Using the Beta dist, update it with the binomial dist as the likelihood factor. Usually initialize with the uniform dist to give equal probability to all outcomes (can initialize with a given prior if so chosen).

If you initialize with a prior that is farther from the 'true' result, it will simply take longer (more iterations) to arrive at the final result.  Descriptive example: You have a strongly weighted prior that global warming is real, it will take much more evidence to prove that it is not real than it would if you had a neutral prior (uniform dist) going in. Of course, the reverse is also true.  However, it is generally considered wisest to initialize with a uniform dist.



**Multi-Armed Bandit**
+ Exploration: Trying out different options to try and determine the reward associated with the given approach (i.e. acquiring more knowledge)  
+ Exploitation: Going with the approach that you believe to have the highest expected payoff (i.e. optimizing decisions based on existing knowledge)

Steps:
+ Start with pure exploration in which groups A and B are assigned equal number of users
+ Once you think you have determined the better option, switch to pure exploitation in which you stop the experiment and send all users to the better performer

Shortcomings:
+ Equal number of observations are routed to A and B for a preset amount of time or iterations
+ Only after that preset amount of time or iterations do we stop and use the better performer
+ Waste time (and money!) on expending resources during continued exploration, such as showing users the version of a site (A v. B) that isn’t performing as well.

MAB is not about always avoiding a sub-optimal strategy. Initially we will willingly choose some sub-optimals strats as we continue to *explore* and find more info about the system.  At some given point, however, we begin to choose the optimal strategy every time in order to minimize *regret* (loss).




### <p style="text-align: right;"> <span style="color:black"> Week 8: Big Data </span> </p>

#### Week 8, Day 1 -- Oct. 17: MapReduce


#### Week 8, Day 2 -- Oct. 18: AWS Setup and LECTURES
Lot of lectures.  

#### Week 8, Day 3 -- Oct. 19: Apache Spark
See "install_spark.md" file for help.
"locahost:8080" in a browser to access UI of Spark

AWS .pem files are region specific (e.g. 'us-west-2') and must match the commands used to open them.


`./spark-ec2 -k jpw-aws -i ~/.ssh/jpw-aws.pem -r us-east-1 -s 6 --copy-aws-credentials --ebs-vol-size=64 launch sparkcluster`


#### Week 8, Day 4 -- Oct. 20: SparkSQL and Spark Machine Learning

check the lecture PDF in the shipdir folder.  Josh included a lot of useful documentation and starter code at the end of it. Also check the ipynb file.

Note there are two Machine Learning modules/docs for Spark.
"mllib" are the old ones, going to be discarded by the community.
"ml" docs are the new ones, the ones everyone is going to use going forward.


#### Week 8, Day 5 -- Oct. 21: Graph Theory


Graph is a synonym for a Network.
Probabilistic Graphical Models is the marriage of Graph Theory with Probability Theory.
A graph is simply a set of nodes connected by lines/edges.

A Markov Chain is a type of graph. It is *directed* because there is a probability of going from Node A to Node B, one-direction.

Several nodes in succession form a *walk*.
Several nodes in succession where no node is crossed twice form a *path*.
A closed path is known as a *cycle*.

When there are multiple attributes on a given edge, it is a *DiGraph* (short for *Directed Simple*) and Adam prefers DiGraphs over MultiGraphs due to flexibility and interpretability.


When plotting graph using networkx, spring_layout() is a pretty common and effective layout to use.

networkX has a big gallery on their site for plotting.

G.subgraph([n1, n2, ...]) is an awesome command that returns subgraphs and allows us to really examine in detail parts of the larger graph.

Sometimes we need to turn a graph into a matrix and analyze / compute it that way.  We can use Python's dictionary data structure to handle this efficiently.



#### Week 9, Day 1 -- Oct. 24: Speedy Python
See "speedy python.pdf" in Week8/high-performance-python

Saving your work as a pickle

```python
import cPickle,os

results = {'a':1,'b':range(10)} results_pickle = 'foo.pickle' tmp = open(results_pickle,'w') cPickle.dump(results,tmp) tmp.close()
tmp = open(results_pickle,'r')
loadedResults = cPickle.load(tmp)
tmp.close()
```


Saving your work in NumPy
```python
import numpy as np a = np.arange(10)

b = np.arange(12)
file_name = 'foo.npz'
args = {'a':a,'b':b}
np.savez_compressed(file_name,**args)
npz = np.load(file_name)
a = npz['a']
b = npz['b']
```
There is also `np.save`, `np.savetxt`, and `np.savez`

`np.savez_compressed(file_name,**args)` is for multiple files, the above three commands are for single files.



An example of using cPickle effectively:
```python
import random
import cPickle as pickle

class MyModel():
    def fit():
        pass
    def predict():
        return random.choice([True, False])

def get_data(datafile):
    ....
    return X, y

if __name__ == '__main__':
    X, y = get_data('data/train.json')
    model = MyModel()
    model.fit(X, y)
    with open('model.pkl', 'w') as f:
        pickle.dump(model, f)
```

Now I can reload the model from the pickle and use it to predict! No need to retrain.

```python
with open('model.pkl') as f:
    model = pickle.load(f)

model.predict(…)
```


Some example code showing various ways to save objects:
```python
import pandas as pd
import numpy as np
import random
import cPickle as pickle


def clean_df(df):
    pass


class MyModel(object):
    def fit(self):
        pass

    def predict(self):
        return random.choice([True, False])


def get_data(datafile):
    ....
    return X, y


if __name__=='__main__':
    # We can save numpy vectors to a compressed dictionary
    a = np.arange(1000)
    b = np.arange(1000)
    file_name = 'numpy_dict.rpz'
    args = {'a': a, 'b': b}
    np.savez_compressed(file_name, **args)

    # And read them in
    npz = np.load(file_name)
    a = npz['a']

    #------------------------

    # Let's look at DataFrames specifically
    df = pd.read_csv('my_awesome_csv.csv')

    # Do your data cleaning which is potentially very time intensive
    clean_df(df)

    # Pickle the resulting DataFrame
    df.to_pickle('my_dataframe.pkl')

    # We can now easily read this back in without having to reprocess the df
    df = pd.read_pickle('my_dataframe.pkl')

    #------------------------

    # Let's look at training a model and then saving the resulting model
    X, y = get_data('data/train.json')
    model = MyModel()
    model.fit(X, y)
    with open('model.pkl', 'w') as f:
        pickle.dump(model, f)

    # Now we can read this model in without having to retrain!
    with open('model.pkl') as f:
        model = pickle.load(f)

    model.predict(...)
```



Use htop to monitor what the processors are doing:
`brew install htop`

Hyperthread - multiple processes on the same core




I will be re-using the Great Circle problem to illustrate several the different methods to speed up code. Basically, we are trying to calculate the shortest distance between two points on a sphere (i.e. the earth).


<div align="center">
    <img height="250" src="/Users/jpw/Dropbox/Work/Galvanize/DSI_Camp/Week8/high-performance-python/img/spherewire.png">
</div>

```python
import math
def great_circle(lon1,lat1,lon2,lat2): radius = 3956 #miles
    x = math.pi/180.0
    a = (90.0-lat1)*(x)
    b = (90.0-lat2)*(x)
    theta = (lon2-lon1)*(x)
    c = math.acos((math.cos(a)*math.cos(b)) + (math.sin(a)*math.sin(b)*math.cos(theta)))
    return radius * c
```


How far is it from Machu Picchu to Pikes Peak?
```python
from GreatCircle import great_circle
print(great_circle(-13,73,-105,38))
```

And what about a million distances?
```python
import numpy as np
n = 1000000
m = np.random.randint(-360,360,n*4).reshape(n,4)
```

Then to loop through the matirx...
```python
for i in range(mat.shape[0]):
    x = great_circle(mat[i,:])
```


What can we do to speed this up?
Can you think of any tools that you might already have?
NumPy is fast when we are in the matrix world
It is generally inefficient to loop. Use NumPy
```python
def great_circle_numpy(mat):
    radius = 3956
    x = math.pi/180.0
    lon1 = mat[:,0]
    lat1 = mat[:,1]
    lon2 = mat[:,2]
    lat2 = mat[:,3]
    a = (90.0-lat1)*(x)
    b = (90.0-lat2)*(x)
    theta = (lon2-lon1)*(x)
    c = np.cosh((np.cos(a)*np.cos(b)) + (np.sin(a)*np.sin(b)*np.cos(theta)))
    return radius*c
```

Run it
```python
from GreatCircle import great_circle_numpy
timeStart = time.time()
c = great_circle_numpy(mat)
runTime = time.time() - timeStart
print time.strftime('%H:%M:%S', time.gmtime(runTime))
```


Subprocess:

```python
import subprocess

timeStart = time.time()
cmd = 'python RunGreatCircle.py'
proc = subprocess.Popen(cmd,shell=True,stderr=subprocess.PIPE,
                        stdout=subprocess.PIPE)
stdOut, stdErr = proc.communicate()
runTime = time.time() - timeStart
print("Python time", time.strftime('%H:%M:%S', time.gmtime(runTime)))
```


CUDA is an API from nVidia, written in C, but is accessible in Python via a C wrapper called PyCUDA.



More Elaborate Pickle:
```python
import random
import cPickle as pickle

class MyModel():
    def fit():
        pass
    def predict():
        return random.choice([True, False])

def get_data(datafile):
    ....
    return X, y

if __name__ == '__main__':
    X, y = get_data('data/data.json')
    model = MyModel()
    model.fit(X, y)
    with open('model.pkl', 'w') as f:
        pickle.dump(model, f)
```

Now I can reload the model from the pickle and use it to predict! No need to retrain :)
```python
    with open('model.pkl') as f:
        model = pickle.load(f)

    model.predict(...)
```




#### Week 9, Day 2 -- Oct. 25: Flask

Use alligator brackets to denote a variable: `<variable>`
There is a limit to how large `GET` requests can be.
There is no size limit to `POST` commands.
`request` is a method you import from Flask
```python
from flask import Flask, request, render_template
```

Specify a port to run on, and then view in browser by typing `localhost:[port]`
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(port = 5000)

```

can also use: `app.run(host='0.0.0.0', port=8080, debug=True)`





#### Week 9, Day 5 -- Oct. 28: Review
Hadoop is made of two parts:
1. HDFS (Hadoop Distributed File System)
2. Map Reduce

Spark can replace the MR part and still embrace a HDFS if needed, though HDFS is not required for Spark
1. RDD (Resilient Distributed Database) -- partitioned across all slaves
  + immutable
  + can be worked with via key-value pairs
  + Can only be interacted upon by two methods:
    + Transformations
    + Actions
  + Spark operates via Lazy Instruction and will queue up all the transformations issued but only perform them once an action command is given.  This prevents it from reading/writing after every transformation which is the biggest time killer in big data (Hadoop does this and this is why it is significantly slower)

    ```python
    import pyspark as ps
    sc = ps.SparkContext()
    ```
    Think of our Spark Context as a master by which we interact with Spark


YARN - Yet Another Resource Negotiator (typically used with Hadoop)
MESOS -







=======
## Random DSI notes that need to be integrated into this file:

#### Testing Files
to run test files use: `nosetests <filename>` in the terminal at correct dir (test file and script should be in same dir)

***
#### Module Imports
When trying to import a module in a different directory than the script you're trying to run, navigate to the directory of the module itself in the terminal and see if there is a '\__init__.py' file.  If there isn't, execute `touch __init__.py` to create it.  Then in your python script you can drill down into the correct directory (only down from the script's directory) in your import statement by doing a dot-command.  Example: your script file is called "py_script" and is in `~/main` dir and your module is called "py_mod" and is in `~/main/data` dir, you would go into the `data` dir in terminal, issue `touch __init__.py` command, and then use the following import command atop your python script: `from data.py_mod import [class]` or whatever you need to import, or `*`.  You can use `..` ahead of the dir to extend the pathname: `df = pd.read_csv('../data/bigfile.csv')`



**Common modules and imports**
```python
from __future__ import division #makes default div a float not int
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pandas.tools.plotting import scatter_matrix
import seaborn as ... #advanced plotting
from statsmodels.regression.linear_model import OLS
import statsmodels.formula.api as sm #module for stats/regressions
from sklearn.cross_validation import train_test_split
from sklearn.utils import resample  #used for bootstrapping
from string import punctuation #to filter punctuation chars!
```


**Progress Bar in Terminal for Slow Funcs**
`conda install progressbar` in terminal
```python
from progressbar import ProgressBar
pbar = ProgressBar()
pbar(slow_func)

```
Read some examples of it, but this is basically it.


***
#### Mongo DB
In a new terminal window, enter `mongod` to start a Mongo server connection.
Then in a new terminal window, enter `mongo [name]` where [name] is the database you'd like to connect to.  Omitting [name] connects to the default database, usually your username.




`n_jobs = -1` (I THINK) for using multi processors to compute in python

after a function you can `return locals()` to return ALL vars in the local scope of the given function to the namespace which called the function (i.e. one level up in scope).
Potentially dangerous but also convenient.



Colormap evaluation module by new MatPlotLib guys from SciPy2015
`pip install viscm`
`python -m viscm view [colormap]`  (e.g. [colormap] = jet)
`python -m viscm edit`  to design your own colormap

`pip install colorspacious` to access this group's colormaps (virids = new default)

VERY COOL!




np.linalg.lstsq(A, b)[0] -- least squares func from numpy



***
#### Graph Lab
Registered email address: jonpaul.wright@gmail.com
Product key: 9FD5-C85F-41E7-951F-1224-84E8-9662-460D

installation and usage:
https://turi.com/download/install-graphlab-create-command-line.html


to activate and use Graph Lab:
`source activate gl-env`

This activates the separate GL environ.  Once the free trial of GL ends, we can remove this environ and it won't affect our normal Anaconda environ. Yay.

to deactivate and stop Graph Lab:
`source deactivate gl-env`



pip install --upgrade --no-cache-dir https://get.graphlab.com/GraphLab-Create/2.1/jonpaul.wright@gmail.com/9FD5-C85F-41E7-951F-1224-84E8-9662-460D/GraphLab-Create-License.tar.gz



If you are getting an error `ImportError: No module named shutil.get_terminal_size` you need to run `pip uninstall backports.shutil_get_terminal_size` and then run `pip install --upgrade backports.shutil_get_terminal_size`




***
#### Class and Instance Methods

Note there is a difference between *class methods* and *instance methods* in a class. It allows you to pass a function/method as simply an object, which it is, to a class.  This requires slightly different assignment syntax within the class, though, as shown below (code from Erich):

```python
class MyClass1(object):
    def __init__(self, bar):
        self.foo = 'updog'
        MyClass1.foobar = bar

class MyClass2(object):
    def __init__(self, bar):
        self.foo = 'updog'
        self.foobar = bar

def bar(self):
    return "What's " + self.foo


if __name__=='__main__':
    # In MyClass1, the .foobar method will be a method similar to how we have
    # seen it before (i.e. it is a method bound to the instance of this class)
    # and we can thus call x.foobar() as though foobar was definied in the class
    x = MyClass1(bar)
    x.foobar()
    # e.g.
    # In [3]: x.foobar
    # Out[3]: <bound method MyClass1.bar of <__main__.MyClass1 object at 0x104370d90>>


    # In MyClass2, .foobar is simply a reference to the bar function and is
    # NOT a bound method.  Because of this we must pass the instance in for
    # this function to work properly.
    y = MyClass2(bar)
    y.foobar(y)

    # e.g.
    # In [39]: y.foobar
    # Out[39]: <function __main__.bar>
```


***
#### String Replacement
Replace items in a string via a ref dict
```python
def replace_all(text, dic):
    for i, j in dic.iteritems():
        text = text.replace(i, j)
    return text
```


replace items in a string via RegEx
```python
import re

rep = {"condition1": "", "condition2": "text"} # define desired replacements here

# use these three lines to do the replacement
rep = dict((re.escape(k), v) for k, v in rep.iteritems())
pattern = re.compile("|".join(rep.keys()))
text = pattern.sub(lambda m: rep[re.escape(m.group(0))], text)
For example:

>>> pattern.sub(lambda m: rep[re.escape(m.group(0))], "(condition1) and --condition2--")
'() and --text--'
```


replace with a temp var:
```python
When you need to swap variables, say x and y, a common pattern is to introduce a temporary variable t to help with the swap:  t = x; x = y; y = t.

The same pattern can also be used with strings:

>>> # swap a with b
>>> 'obama'.replace('a', '%temp%').replace('b', 'a').replace('%temp%', 'b')
'oabmb'
```
***





#### Web Scraping
BeautifulSoup - How to save an image/content to a specific directory
```python
def get_img(html):
    soup = BeautifulSoup(html)
    img_box = []
    imgs = soup.find_all('div', class_= 'pthumb')

    for img in imgs:
        img_box.append(get_domain(BASE_URL) + img.img['src'])

    my_path = '/home/<username>/Desktop'  # use whatever path you like
    for img in img_box:
        urllib.request.urlretrieve(img, os.path.join(my_path, os.path.basename(img)))
```


Save to the same directory the script is run from:
`urllib.urlretrieve(img, os.path.basename(img))`

enforce a pause in running a script!
```python
import time
time.sleep(6)   #Pause for 6 seconds
```


check and make directory via python
```python
def ensure_dir(f):
    d = os.path.dirname(f)
    if not os.path.exists(d):
        os.makedirs(d)
```


open URL in a browser (without Selenium):
```python
import webbrowser

webbrowser.open(url, new=0, autoraise=True)
```
`new=0` : open in current browser window
`new=1` : open in new browser window
`new=2` : open in new tab in current browser window


can also use these to dosomething or such?
`webbrowser.Chrome(self, name='')`
`webbrowser.MacOSX(self, name)`  #Aqua browsers, aka Safari, or sys default





register browser so `webbrowser` knows which browser to open in:
```python
import webbrowser
url = 'http://docs.python.org/'

# MacOS
chrome_path = 'open -a /Applications/Google\ Chrome.app %s'

webbrowser.get(chrome_path).open(url)
```


Use `webbrowser.get()` to specify the browser.
'macosx' : MacOSX('default')
'safari' : MacOSX('safari')
'google-chrome' : Chrome('google-chrome')
'chrome' : Chrome('chrome')



open URL in incognito mode:
NOTE -- Much easier to just use Selenium for Incognito mode, built-in.

```python
import os

SOMETHING LIKE THIS FOR MAC (command line)
os.system("/Applications/Google\ Chrome.app -ArgumentList @( '-incognito', 'http://www.snackdata.com/bigimages/blueberry.gif'")

#Windows
os.system("C:\Program Files (x86)\Google\Chrome\Application\chrome.exe -ArgumentList @( '-incognito', 'www.foo.com'" )

#Mac
Unsure -- add '-incognito' as an argument.
```



***
#### Generators
```python
def toy_func():
    while True:
        do stuff

    yield answer
```

`yield` makes it a generator, and all generators have a built-in `next` function which will return the next value in the iterable/function, piecemeal.  This is computationally beneficial and necessary for big data and fast processing times.

So, define a variable to be the function:

```python
f = toy_func()
f.next
```


***
#### Bash Profile and API Keys

Bash Profile is where we save "environmental variables" to keep secret/secure and hidden from our scripts/the public, but still allow us to use the variables, such as API keys, by accessing them through a .bash_profile profile keyword/variable name.

go to root directory (`cd ~` in terminal), type `atom .bash_profile`
add
```python
export EXAMPLE_KEY=[key]
```

**MUST** source the file after adding to it.
In terminal, `source ~/.bash_profile` from the tab you are going to use.  If you switch to a new tab in terminal, you will have to source in that tab again before it will work.

Check if it is saved by going to root directory and type `echo $EXAMPLE_KEY` where EXAMPLE_KEY is the actual key name, of course.

AWS: use Identity and Access Management pane for all user logins and credentials access.

Use the following in a python script to get the Access Key
```python
import os
os.environ['AWS_ACCESS_KEY_ID']
```





#### LaTeX
Using LaTex:
install Font Awesome for the cool flat icons.
The following will give you different outputs for a .tex file.
`latex [filename.tex]`
`pdflatex [filename.tex]`
`xelatex [filenmae.tex]`  

Use xelatex for most output, I believe.









How to unzip a .zip file in python:
`unzip data/data.zip -d data`
