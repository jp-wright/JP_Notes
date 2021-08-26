# JP SKLearn Notes


## confusion_matrix
sklearn confusion_matrix is arranged opposite of standard cmats.  The arrangement I always learned is:

Key:
+ TP = True Positive
+ TN = True Negative
+ FP = False Positive
+ FN = False Negative

##### Common way:
ConMat           | (Pred) True  | (Pred) False |
-----------------|--------------|--------------|
(Actual) True    |   TP         |   FN         |
(Actual) False   |   FP         |   TN         |

<BR>

##### SKlearn confusion_matrix way:
ConMat           | (Pred) False | (Pred) True  |
-----------------|--------------|--------------|
(Actual) False   |   TN         |   FT         |
(Actual) True    |   FN         |   TP         |

So either adjust your matrix in your head when reading the confusion_matrix results, or you can simply flip it with an index assignment:
```python
# To get the common confusion_matrix alignment, we flip the diagonal and then transpose to flip the off-diagonal quadrants.
# Use the following dummy info to test which quadrants are which.
y_true = [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1]
y_pred = [1, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1]
cmat = confusion_matrix(y_true, y_pred)
cmat[0][0], cmat[1][1] = cmat[1][1], cmat[0][0]
cmat = cmat.T
```
<BR>

Here is a nice graphical representation of all classes and results using Seaborn and MPL from Jake van der Plaas, along with some additional console output by me (my console output is currently only meant for binary classification):

```python
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.metrics import confusion_matrix

mat = confusion_matrix(y_test, y_pred)
# Flip the entire matrix to match standard use, with:
cmat[0][0], cmat[1][1] = cmat[1][1], cmat[0][0]
cmat = cmat.T
print (cmat, "\nActual True: {at} - Pred True: {pt} \
\nActual False: {af} - Pred False: {pf} \
\nAccuracy: {ac} \
\nPrecision: {pr} \
\nRecall: {re}".format(at=cmat.sum(1)[0], \
af=cmat.sum(1)[1], \
pt=cmat.sum(0)[0], \
pf=cmat.sum(0)[1], \
ac=round(cmat.diagonal().sum() / cmat.sum() * 100, 3), \
pr=round(cmat[0,0] / cmat.sum(0)[0] * 100, 3), \
re=round(cmat[0,0] / cmat.sum(1)[0] * 100, 3)))

# y_name = sorted(y_test.unique())  # use when y_test is Series
y_name = sorted(set(y_test), reverse=True)
ax = sns.heatmap(cmat, square=True, annot=True, fmt='d', cbar=False, xticklabels=y_name, yticklabels=y_name, robust=True)

# Be mindful of which Axis is True or Predicted.  If you flip the CMAT as I've done, the following is corrected. If you only flip it once, then these would be reversed.
plt.xlabel('Predicted Label')
plt.ylabel('True Label')
plt.xticks(rotation='vertical')
plt.yticks(rotation='horizontal')
f = plt.gcf()
f.subplots_adjust(bottom=0.16, left=0.12, right=.99, top=0.95)
plt.show()
```

<BR><BR>

## GridSearchCV
Here's the [documentation](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) on GridSearchCV, and here's the [user guide](http://scikit-learn.org/stable/modules/grid_search.html#grid-search) with some more thorough explanations.

##### Important Note About `n_jobs` And Memory Explosion
https://randomforests.wordpress.com/2014/02/02/basics-of-k-fold-cross-validation-and-gridsearchcv-in-scikit-learn/
I did finally figure out issues with `GridSearchCV` from SKLearn (without punching a hole through my monitor, so, double win!):
1. When using this module, you can __not__ set `n_jobs` > 1 _inside the model constructor_ (e.g. `rf = RandomForestRegressor(... n_jobs=1)`) if you are using `n_jobs` > 1 inside the `GridSearchCV` as well.  This is because when we call `GridSearchCV(rf, params, cv=5)`, the model instance (e.g. `rf`) gets spawned once for every fold (`cv`).  If you have multiple cores assigned to the model instance, you have then spawned multiple counts of each instance using multiple cores.  I have only four cores on my laptop, so it could only take two folds of two cores to cap out on my possible cores limit. If you monitor this activity with `htop` in a separate terminal window, you see that one process (i.e. one use of `GridSearchCV` will use all cores, which is inefficient and not the desired outcome of parallelization).  

2. The correct approach is to use `n_jobs=1` inside the model, and `n_jobs=[num cores]` (where num cores is specified by you) inside `GridSearchCV`.  Doing this, we see in `htop` that the number of cores assigned to `GridSearchCV` each spawn their own process, correctly, and utilize a separate core for their computation.  This is the desired outcome of parallelization.

3. While the two points above matter, the single biggest finding is that there is a vicious memory leak in the newly released (as of SK-Learn v. 0.18) `'mae'` option for the `criterion` argument inside the regression model constructor.  Until v 0.18, the only choice was `criterion='mse'`.  `'mae'` is likely a bit better suited to my data, however, so I wanted to use it as the splitting criterion for my `RandomForestRegressor` model.  Bad move.  After days of experiments and hassle, I figured out that `'mae'` was taking around 120x (!) longer than the traditional `'mse'` was for every fit in the grid search.  

    Wondering what was going on, I ran a search that alternated between the two and watched in `htop`. Sure enough the `'mae'` runs started gradually shooting the memory consumption up, and would not relinquish it until an `'mse'` run came up.  This explained the slow times and the infuriating memory crashes of the previous week -- in `'mae'` searches of any measure (n combinations > 40) the memory kept being consumed with each `cv` fitting, and _never given back_.  8 GB, 9 GB, 10 GB, and so on until it crashed my entire computer or OS X issued a `Killed: 9` termination command to the process.

    A very important point to note: you can't use `criterion='mae'` inside the model itself, but you __can__ still use `'mae'` as your _scoring_ metric inside the `GridSearchCV` function itself.  That's fine.  The memory leak only exists in the use of `'mae'` as a way to measure splitting efficacy in the regression version of `RandomForestRegressor`.  This is what I did -- still used `'mae'` as my scoring metric since I feel it is what best applies to my data in regressing on the spread of games.


##### Important Note About Scoring Metrics:
From the User Guide:
>By default, parameter search uses the score function of the estimator to evaluate a parameter setting. These are the sklearn.metrics.accuracy_score for classification and sklearn.metrics.r2_score for regression.  

This means that the default choice of the grid search is going to be the default "accuracy" score for classification or "R2" score for regression.  These are both very poor choices unless the data happens to be perfectly balanced and, ideally, linearly independent.  In reality, neither of these two things tend to be true.  As such, we can specify a different scoring metric for the grid search to use (estimator-dependent, of course).  This is such an important concept that SK-Learn has some nice [examples and an explanation](http://scikit-learn.org/stable/modules/model_evaluation.html#scoring-parameter) to help you along.

Here is a table of the possible `scoring` parameter options, taken from the user guide:
>For the most common use cases, you can designate a scorer object with the scoring parameter; the table below shows all possible values. All scorer objects follow the convention that higher return values are better than lower return values. Thus metrics which measure the distance between the model and the data, like metrics.mean_squared_error, are available as neg_mean_squared_error which return the negated value of the metric.

Scoring	                 | Function	                   | Comment
-------------------------|-----------------------------|-----------------
**Classification**       |                             |
`‘accuracy’`	         | metrics.accuracy_score      |	 
`‘average_precision’`	 | metrics.average_precision_score |  
`‘f1’`	                 | metrics.f1_score	           | for binary targets
`‘f1_micro’`	         | metrics.f1_score	           | micro-averaged
`‘f1_macro’`	         | metrics.f1_score	           | macro-averaged
`‘f1_weighted’`	         | metrics.f1_score	           | weighted average
`‘f1_samples’`	         | metrics.f1_score	       | by multilabel sample
`‘neg_log_loss’`	     | metrics.log_loss	       | requires `predict_proba` support
`‘precision’` etc.	     | metrics.precision_score | suffixes apply as with `‘f1’`
`‘recall’` etc.	         | metrics.recall_score	   | suffixes apply as with `‘f1’`
`‘roc_auc’`	             | metrics.roc_auc_score	   |
**Clustering**           |                             |
`‘adjusted_rand_score’`	 | metrics.adjusted_rand_score |  
**Regression**           |                             |
`‘neg_mean_absolute_error'`| metrics.mean_absolute_error |	 
`‘neg_mean_squared_error'`| metrics.mean_squared_error |	 
`‘neg_median_absolute_error’`| metrics.median_absolute_error |
`‘r2’`	                 | metrics.r2_score	           |

Read further in the explanations post for ways to design your own custom scoring metrics if needed.  

Please note that not all scorers are able to handle multi-class targets in classification models.
Scorer               | Binary | Multi-Class
---------------------|--------|------------
`‘accuracy’`	     | Yes    |	Yes
`‘average_precision’`| Yes    | No ("multi-label" only)
`‘f1’`	             | Yes	  | No
`‘f1_micro’`	     | Yes	  | Yes, when `average='micro'`
`‘f1_macro’`	     | Yes	  | yes, when `average='macro'`
`‘f1_weighted’`	     | Yes	  | Yes, when `average='weighted'` ()
`‘f1_samples’`	     | Yes	  | Yes, when `average='samples'`
`‘neg_log_loss’`	 | Yes	  | Yes
`‘precision’` etc.	 | Yes    | Yes. Must set `average=` to one of `[None, ‘micro’, ‘macro’, ‘samples’, ‘weighted’]`
`‘recall’` etc.	     | Yes	  | Yes. Must set `average=` to one of `[None, ‘micro’, ‘macro’, ‘samples’, ‘weighted’]`
`‘roc_auc’`	         | Yes	  | No


##### `average` Argument for Multi-Class:
`average` : string, [`None`, `‘binary’` (default), `‘micro’`, `‘macro’`, `‘samples’`, `‘weighted’`]  
    This parameter is required for multi-class/multi-label targets. If `None`, the scores for each class are returned. Otherwise, this determines the type of averaging performed on the data:  
+ `'binary'`:  
    Only report results for the class specified by `pos_label`. This is applicable only if targets (`y_{true,pred}`) are binary.  
+ `'micro'`:  
    Calculate metrics globally by counting the total true positives, false negatives and false positives.  
+ `'macro'`:  
    Calculate metrics for each label, and find their unweighted mean. This does not take label imbalance into account.  
+ `'weighted'`:  
    Calculate metrics for each label, and find their average, weighted by support (the number of true instances for each label). This alters `‘macro’` to account for label imbalance; it can result in an F-score that is not between precision and recall.  
+ `'samples'`:  
    Calculate metrics for each instance, and find their average (only meaningful for multi-label classification where this differs from accuracy_score).  

##### Note On Choosing Between mean_squared_error (MSE) and mean_absolute_error (MAE)
From [this post](https://medium.com/human-in-a-machine-world/mae-and-rmse-which-metric-is-better-e60ac3bde13d) on the error metrics:
>__RMSE__ has the benefit of penalizing large errors more so can be more appropriate in some cases, for example, if being off by 10 is more than twice as bad as being off by 5. But if being off by 10 is just twice as bad as being off by 5, then __MAE__ is more appropriate.  
>From an interpretation standpoint, __MAE__ is clearly the winner. __RMSE__ does not describe average error alone and has other implications that are more difficult to tease out and understand.  

##### Important Note About the _ERROR_ Scorers inside `GridSearchCV` or `cross_val_score`:  
The SKLearn scorer API is written to *maximize* scores (i.e. select scores that are as large as possible).  This is problematic when we are using one of the three *error* metrics (`‘neg_mean_absolute_error’`, `‘neg_mean_squared_error’`, `‘neg_median_absolute_error’`) because they function by *minimizing* scores.  This means that under the standard approach by the SKLearn API, the worst of these scores (the highest score -- the one with the *most* error) would be considered the best and would be chosen by `GridSearchCV`. Obviously this isn't what we want from our parameterization.  

So, SKLearn addressed this by simply negating the error scorers which means that the bigger the raw error score the more negative it becomes (*3.5* --> *-3.5* and *14.0* --> *-14.0*), and since the scorer API always chooses the maximum score, this means it will now correctly choose the 'smaller' score (one with less error). The source of confusion and purpose of this note is that the error is then *reported as a negative*.  This was very confusing to me initially (and many others as evidenced by [this post](https://github.com/scikit-learn/scikit-learn/issues/2439)) because I didn't understand how we could have a negative value for a squared product....   

The official solution as stated in that post, basically, is to simply multiply the results by -1, to flip the sign.  Easy enough, but warrants mentioning here since `GridSearchCV` and `cross_val_score` are the two places it is commonly encountered.



<br>

#### Basic setup
Here is a basic example using `GridSearchCV` with a Random Forest:  
```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.cross_validation import train_test_split

# X and y are assumed to exist ...
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0, train_size=.8)

rf = RandomForestClassifier()
parameters = {'n_estimators': [50, 100], 'max_features': ['sqrt'], 'max_depth': [3, 5], 'max_leaf_nodes': [3, 5], 'oob_score': [True], 'random_state': [462], 'n_jobs': [3]}
clf = GridSearchCV(rf, parameters, scoring='recall', cv=5, verbose=1)
clf.fit(X_train, y_train)
```

This will take from a couple to many minutes to process.  It should be noted that GridSearchCV can also be used to test multiple *estimators* (i.e. algorithms), as well.  So we could test XGBoost v. Random Forest, for example, in order to find not only the best parameters for a given model, but also find the best model as well.  The `verbose` argument gets progressively more detailed with higher integer values, with `verbose=1` being the best option.

Once done, we have access to the results in various ways:

#### Attributes
Attribute       | Description
----------------|------------
`cv_results_`   | **[dict]** - A dict with keys as column headers and values as columns, that can be imported into a pandas DataFrame. (the key `params` is used to store a list of parameter settings dict for all the parameter candidates.)
`best_estimator_` | **[estimator]** - Estimator that was chosen by the search, i.e. estimator which gave highest score (or smallest loss if specified) on the left out data. Not available if `refit=False`.
`best_score_`   | **[float]** - Score of best_estimator on the left out data
`best_params_`  | **[dict]** - Parameter setting that gave the best results on the hold out data.
`best_index_`   | **[int]** - The index (of the `cv_results_` arrays) which corresponds to the best candidate parameter setting. The dict at `search.cv_results_['params'][search.best_index_]` gives the parameter setting for the best model, that gives the highest mean score (`search.best_score_`).
`scorer_`       | **[function]** - Scorer function used on the held out data to choose the best parameters for the model.
`n_splits_`     | **[int]** - The number of cross-validation splits (folds/iterations)

<BR>

#### Methods
    Method                              |   Description
----------------------------------------|-------------------------
`decision_function`(\*args, \*\*kwargs)	| Call `decision_function` on the estimator with the best found parameters. Estimator itself must support `decision_function` (Random Forest, for example, does not).
`fit`(X, y=None, groups=None)	        | Run `fit` with all sets of parameters.  `y`: Target relative to `X` for classification or regression; `None` for unsupervised learning. `groups`: Group labels for the samples used while splitting the dataset into train/test set.
`get_params`(deep=True)	                | Get parameters for this estimator.
`inverse_transform`(\*args, \*\*kwargs)	| Call `inverse_transform` on the estimator with the best found params.
`predict`(\*args, \*\*kwargs)	            | Call `predict` on the estimator with the best found parameters.
`predict_log_proba`(\*args, \*\*kwargs)	| Call `predict_log_proba` on the estimator with the best found parameters.
`predict_proba`(\*args, \*\*kwargs)	    | Call `predict_proba` on the estimator with the best found parameters.
`score`(X, y=None])	                    | Returns the score on the given data, if the estimator has been refit. Note this uses the score defined by `scoring` where provided, and the `best_estimator_.score` method otherwise. `y`: Target relative to `X` for classification or regression; `None` for unsupervised learning.
`set_params`(\*\*params)	            | Set the parameters of this estimator. The method works on simple estimators as well as on nested objects (such as pipelines). The latter have parameters of the form <component>\_\_<parameter> so that it’s possible to update each component of a nested object.
`transform`(\*args, \*\*kwargs)	        | Call `transform` on the estimator with the best found parameters.


## Class Imbalance
While not part of SK-Learn, there are [modules available](http://contrib.scikit-learn.org/imbalanced-learn/generated/imblearn.over_sampling.SMOTE.html) for SMOTE.




## Cross-Validation
Here is a [good set of examples](http://scikit-learn.org/stable/modules/cross_validation.html#cross-validation) of using CV in SK-Learn.

+ We can use CV to return the score, as well as the confidence interval (!) for a given metric for any set of train data using `cross_val_score`:

    ```python
    >>> from sklearn.model_selection import cross_val_score
    >>> clf = svm.SVC(kernel='linear', C=1)
    # Note the use of specific scoring metric in CV.
    >>> scores = cross_val_score(clf, iris.data, iris.target, cv=5, scoring='f1_macro')
    >>> scores                                              
    array([ 0.96...,  1.  ...,  0.96...,  0.96...,  1.        ])

    >>> print("Accuracy: %0.2f (+/- %0.2f)" % (scores.mean(), scores.std() * 2))
    Accuracy: 0.98 (+/- 0.03)
    ```

+ We can also use `cross_val_predict` (much like the standard `[model].predict` method) to return predictions for test target data so that we can then directly compare our test predictions to our actual test targets (without having to re-run the model using whatever parameters we fed the CV). (Note: the result of this computation may be slightly different from those obtained using cross_val_score as the elements are grouped in different ways.):

    ```python
    >>> from sklearn.model_selection import cross_val_predict
    >>> predicted = cross_val_predict(clf, iris.data, iris.target, cv=10)
    >>> metrics.accuracy_score(iris.target, predicted)
    0.966...
    ```



#### pre_dispatch
From [this post](http://stackoverflow.com/questions/32673579/scikit-learn-general-question-about-parallel-computing) on Stackoverflow
>Suppose you are doing KNN and have to choose between k=[1,2,3,4,5, ... 1000]. If you set n_jobs=2, GridSearchCV will first create 1000 jobs, each with one choice of your k, also making 1000 copies of your data (possibly blowing up your memory if your data is big), then sending those 1000 jobs to 2 CPUs (most jobs will be pending of course). GridSearchCV doesn't just spawn 2 jobs for 2 CPUs because the process of spawning jobs on-demand is expensive. It directly spawns equal amount of jobs as parameter combinations you have (1000 in this case). In this sense, the wording n_jobs might be misleading. Now, using pre_dispatch you can set how many pre-dispatched jobs you want to spawn.
