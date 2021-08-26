# JP Data Science Notes

## Modeling

### Error Estimate in Model Selection (Monte Carlo Cross-Validation)
As Sebastian Rauschka points out in his [three-part model selection series](https://sebastianraschka.com/blog/2016/model-evaluation-selection-part1.html), confidence intervals for model performance can be estimated primarily one of three ways.

1. [Normal Approximation Interval](https://sebastianraschka.com/blog/2016/model-evaluation-selection-part1.html#confidence-intervals)
2. [Monte Carlo Cross-Validation](https://sebastianraschka.com/blog/2016/model-evaluation-selection-part2.html#repeated-holdout-validation)
3. [Bootstrap and Empirical Confidence Interval](https://sebastianraschka.com/blog/2016/model-evaluation-selection-part2.html#the-bootstrap-method-and-empirical-confidence-intervals)

For now I'll leave the Normal Approximation Interval approach, which he describes as "rather naive" and something he would "not recommend," up to him.  It is fairly straightforward, using the standard error of the mean to compute confidence intervals so long as the original distribution that our samples are taken from is indeed Gaussian.

The other approach is more "hands on", relying less on distribution assumptions and the tenets of the Central Limit Theorem, and instead is an aggregation of actual model results.  All you do is simply run the model multiple times with different original seeds in the train/test split (thus resulting in different data points being in each split with each run) and average the performance results.  Finding this average and then taking the difference between it and the performance when using the full train set (no test set, max overfit) would tell us the pessimistic bias estimate of the model's generalizability.

The Bootstrap CI method is more involved and described with both equations and some plots in his article, which I won't aim to reproduce here.



### Cross-Validation
The purpose of CV is to make better use of given data to test the predictive reliability of a model.  If data size were not an issue (say one million or more samples/rows, and with fairly balanced classes if applicable), we could do a three-way split of our data: train set, validation set, test set.  We would train the model on the train set, validate it repeatedly on the validation set, and once satisfied with that performance we would lastly test on the test set. But such spoils of data are rare, so we instead have to creatively implement a validation set. Cross-Validation is how we do that.

Briefly, we always split our body of data into *train/test* splits, where we train the model on a larger portion of the data, called the "train" split, and then test our model's predictive ability on the previously-unseen "test" data. Both originate from the original database -- we simply chop it into these train/test sections.  This is standard practice, and is intended to prevent our model from overfitting to the data we train it on, which makes it much less flexible and accurate in predicting on as-yet-unseen future data.  There can be a small problem with this process, however.

We call it *data leakage*, which is when the isolated "test" data still manages to influence the model without actually being in the "train" data (which undermines the entire principle of the train/test split!).  It works like this: once we fit our model to the train data, we test it on the test data and record the results.  Then we tweak our model, test it on the test data again, and record the new results.  We repeat this process many times (usually done functionally in a *grid search*, which simply takes a set of the model's parameters and automatically creates and tests the model iteratively using the various parameters the model accepts, and tells us which single set of parameters gives us the best result), ideally getting incrementally better with each round of tweaks.  Finally, the model hits its maximum potential for the provided data and our tweaks no longer improve it.  Well, without realizing it, we have actually been tuning our model to *fit the test data* the entire time (since each tweak that was kept was one that improved the score on the test data)! Eek! In this way, the test data has *leaked* into our model and influenced how we have tuned it.  In order to counteract this, we can use **cross-validation**.

Cross-validation is used *only on the "train" data*.  Simply, CV partitions the train data in K groups (usually called "folds", commonly 5 or 10), and then using all but one of the folds to train the model, and testing it on the lone holdout fold.  This is done K times, with each fold getting a turn as the test data.  This "musical chairs" testing approach lets each fold *cross-validate* the other folds in the train data, hence the name.  The scores (whatever metric you choose) for this repeated process are averaged and this is reported as the overall score for the model *on the train data*.  In this way, we can continually tweak and test our model on our train data without any fear of leakage from our test data.  


### Multi-colinearity
In Ordinary Least Squares (OLS) regression (the most common example being traditional _linear regression_) our predictive power can be influenced by multi-colinearity. In ensemble tree methods, the predictive power remains largely unaffected due to the random variable selection of each node.   

But the _feature importance_ of __is__ impacted.  This occurs because feature importance is determined by the model withholding a given variable from the set and seeing how negatively the prediction of the model is impacted.  The more negative the impact, the more important the feature is determined to be.  If we have two variables that have largely the same data, withholding one of them will not negatively impact the model very much since that variable's relative is still in the selection pool, giving the model access to that data.   

So, our goal is to (ideally) remove all related features for any type of model.  Doing so is fairly essential to having an accurate OLS regression model, and in determining true feature importance in an ensemble tree model.  The goal of having a perfectly unique dataset in the real world is usually not feasible, but there are methods that can help improve the dataset's variable independence in various ways.  In general, the point of these techniques is to trim any dataset to its core, unique information that allows as good prediction accuracy (with whichever prediction metric you choose) in as lean of a dataset as possible -- to remove superfluous or diluting variables.  We should note here that the model used for feature evaluation and selection doesn't have to be the same model used to make the actual predictions.  It is reasonable, and in some cases smart, to use one model for the actual predictions -- say a neural net or an SVM -- while using a separate model -- such as a random forest, which lends itself nicely to feature selection -- first to evaluate which features you will use in the predicting model.

1. __Aggregation of Dependent Variables:__  
    Combine two related features to engineer a new, single feature and drop the separate parent features.  By forming a single variable that contains information related to multiple variables we retain the important data while removing the confoundingly-related parent variables.  When done, the parent variables need to have a sensible relationship, otherwise we risk _losing_ valuable information as we attempt to combine two unrelated features.  

2. __Recursive Correlation Pruning:__   
    Identify the variables that are most correlated and iteratively remove one of them.  The colinearity limit for when to consider variables sufficiently related to warrant dropping one of them is user determined, but a common approach is to prune any pair that has a (Pearson) R^2 > 0.700.  This is, in effect, saying "any set of variables that share over 70% of the same information can be considered too related and will be trimmed to a single variable."

3. __Recursive Feature Elimination (RFE):__  
    This technique is primarily used in ensemble tree methods.  Run your model and simply remove a certain percentage of the least important features.  For example, after your initial model run the bottom 20% of features would be removed from the dataset and the model would be run again.  Upon completion, the newest bottom 20% of features would be removed, and the entire process repeated until the dataset is ideally "unique".  The stopping point is usually determined by the model's predictive accuracy -- if upon removal of a set of features the model begins to be less predictive, we have discarded unique and vital information that should clearly be left in the model.  Thus successively removing a swath of features can help identify the true importance of the remaining features, but doing so overly zealously will negatively impact the predictions of the model itself.   
    A slight variation on this approach is to iteratively remove a single feature from the dataset, run the model, and repeat for the entire set of features.  This allows us to numerically rank which features have the biggest impact (this is what ensemble tree methods do to determine their "feature importance" values).  SK-Learn has an [RFE package](http://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.RFE.html) which will use any model you specify that can return feature importance or coefficients to determine which features should be recursively eliminated.  You can even return predictions directly from the RFE model using the trimmed feature set.

4. __Variable Decomposition with Principal Component Analysis (PCA):__  
    Also known as _dimension reduction_, this technique seeks to decompose a set of variables to their core components using some higher-order math.  While the math going on under the hood is not something I have studied, so I am not qualified to describe it with any authority, the basic gist of PCA is clear: to take a dataset of higher dimensionality (i.e. many variables, of which numerous likely share some colinearity) and identify the underlying core components to create _new_ variables that are uncorrelated by definition (this is a form of dimensionality reduction).  As a result, the output of PCA means there should be _no_ correlation between variables in the dataset, making it ready for any model.  (Note: there are issues with using _sparse_ data in PCA. If the dataset in question is sparse, PCA may or may not be a viable technique to use).  SK-Learn has a [PCA package](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) available, including references to some of the underlying theory behind the technique.



### Decision Trees (CART)
Decision trees are essentially an application of the semi-well known game, "20 Questions." The board game "Guess Who" is perhaps a better known example.  If we wanted, we could draw out a decision tree of an actual round of Guess Who as a demonstration.  In Guess Who we are trying to find out "who" the opposing player is by asking questions about their character's appearance which iteratively narrow down the pool of possibilities until all but one are eliminated.  Mathematically, the smartest approach is to ask questions which eliminate half the possibilities. If there are 30 possible people for your opponent to be, then the progression would go 30 --> 15 --> 7.5 (8 - can't have half a person) --> 4 --> 2 --> 1.  Thus it would take a maximum of five turns to guess the opponent.  

This is neither the worst case nor best case scenario, technically.  If we had a question that allowed us to eliminate, say, 90% (27 of 30) of the possible people, we could ask it and if we were correct ("Does your person have a big red nose?" and only three people do), then we've eliminated 27 and are left with three.  Our _worst_ case from this point is that it takes three more turns.  Add those to the first turn and we've used only four total now to win.  Hence, five turns is not the best case.  In fact, the best case should be obvious -- one turn.  We start the game and I take a stab in the dark and guess correctly.  One turn, win.  But, how reliable is that?  How many times would we expect that to work? (1/30 flat rate)

Going back to the big red nose question, we see that if we guess correctly with that question we drastically reduce the pool of possibilities from 30 down to three.  But if we miss with it, and they do not have a big red nose, then we can only eliminate the three that do.  We've gone from 30 possible people now to 27.  Not good.

Thus, without specific knowledge relating to the scenario in question (say, my daughter favors girl characters with dark hair...), then the optimal strategy for a generic game of Guess Who is to eliminate half the pool with each question.  

However, the game is designed with this in mind and there are very few questions which can eliminate half of the pool.  There are more than two colors of hair, eyes, shapes of noses, and types of hats, etc.  So each question regarding these items only eliminates by a factor of how many classes are in the given category.  An easy one to divide half the pool, however, is gender.  Boy or girl.  

So, we have one question that can divide half the pool and numerous other ones that can only divide it by a factor less than that.  Which one should we ask first?  Well, it should be clear that asking the gender question at the beginning will have the most impact because eliminating half the pool when there are 30 people removes 15 of them.  Saving it and asking it when there are only six people removes just three...

Decision trees work this way.  They ask questions of, or _split_, the data and try to figure out which questions have the biggest impact and remove the most possibilities from the pool.  For example, if we have healthcare data in which some people were stroke victims, and one of the variables in the dataset is whether or not the person smoked.  The decision tree might find that splitting the data on the "smoker" variable -- in other words, asking "did this person smoke or not?," just like in the Guess Who: Hospital version -- was much more important in predicting whether someone had a stroke than, say, splitting on whether or not they drank orange juice.  

The tree samples which features provide the best split by using a loss/error metric ("Gini coefficient" or "Entropy / information gained" for classifiers and whatever your error metric is for regressors) at each split.  It starts the tree with the _most important_ split and carries on from there, with each successive split being on a feature that is less important.

If we fully fill the tree by letting it grow until every node becomes a leaf (i.e. a terminal node), we will have a tree that fully details the data given to it.  But, this is problematic b/c it means that the splits further down the tree are based on potentially fairly unimportant features.  This "fitting" then becomes "overfitting" and our tree is now _too specific_ in its construction.  It works for this data alone, but not other similar data that isn't exactly the same.  In a word, this tree is inflexible and lacking in general predictiveness.  In data science terms, we'd say it has _high variance_.

This is the problem of using only a single tree and is resolved by using ensemble methods, like Random Forests and Boosted Trees.



### Random Forests
Popular now, but still fairly young in age, Random Forests are a great approach to using decision trees in a more robust manner.  Random Forests address the issues we described above with single decision trees in a few, distinct ways.

1. Use many trees instead of a lone tree.
    1. We will see how this is employed in the RF model and how it impacts prediction positively.
2. Introduce 
