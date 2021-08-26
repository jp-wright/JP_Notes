# Probability and Stats Notes

##### This needs to be fleshed out with the lecture notes and assignments from DSI, to start.  8.19.17

## Hypothesis Testing
We can test two (or more) groups (samples, populations, sample v. population) to see how likely they are from the same "source" or not.  Basically, how different are they?  Are they different enough to _mathematically_ be considered different?  If so, _how_ different does that have to be?  What are the odds of such a difference (if it even exists) being observed?  These are the questions we can answer with a hypothesis test.  There are actually various types of hypothesis tests with different "rules" based on how many groups we are comparing and what differences we are looking for, but the gist is the same for all: are these groups different?  By how much?  And what is the probability of seeing such a difference?

In common terms, we're basically saying "prove it." The _null hypothesis_ is always the 'standard' or the 'normal' mode, and the _alternative hypothesis_ is the 'claim' you are making.  Think your coffee makes workers more productive than the 'normal' coffee they're already using? Prove it.  Think your medicine works better than the 'normal' one? Prove it.  Using _confidence levels_ allows us to say how certain we are something is 'proved' if the hypothesis test says it is.

Note that by the design of hypothesis testing, we are more likely to fail to reject the null hypothesis than is correct, much like "innocent until proven guilty" in our judicial system -- the onus is upon the alternative hypothesis to confirm validity ('guilty').  This is a good thing.

#### Common Types of Hypothesis Tests
+ Z-tests
+ t-tests
+ A/B
+ Multi-Armed Bandit
+ ANOVA

The Z-test and t-test are really just two versions of the same test.  They're frequently confused, so let's go over exactly when to use each.  The other three will be covered after.

<BR>

#### Z-test vs. t-test
[When to use a Z-test vs. a t-test](http://www.statisticshowto.com/when-to-use-a-t-score-vs-z-score/)?  They are actually similar at their core and only two main factors distinguish them (hence why they're commonly confused).  Both assume an underlying normal distribution for the groups being tested.  However, if the distribution is not normal, the tests can still be done if the sample size is large (though the interpretability of their results will vary/diminish).

##### Z-tests
1. Sample size should be > 30
2. Population's standard deviation (σ) is known.

##### t-tests
1. Just a modified Z-test
2. Used primarily when sample size is small (<= 30).
3. Used when population's standard deviation (σ) is unknown.


The sole purpose of both of these tests is to tell you how many standard deviations (sigma) from the mean value your observed measurement is. There are two slightly different ways to do this and that's why we have two tests.

The simpler case is when we have a sufficient sample size (over 30) and all the information we need up front, namely the population mean and population standard deviation.  In this case we simply look up a value for how many standard deviations the sample's mean that we're testing is away from the population mean -- how "far out" the score we're testing is.  This number (distance) is called the __Z score__, and is commonly in the ±0-3 range.  It comes directly from the __Normal distribution__ and is looked up on the [Z table](http://www.statisticshowto.com/tables/z-table/) (aka a Normal Distribution table).  In this sense, we can think of it as "Z = Normal Distribution" That's it.  

However, when we don't know the standard deviation of the population, we can estimate it by using the standard deviation of the sample.  Obviously this is not likely to be exactly correct, but still allows us the ability to do the test.  The tool that allows us to make this estimate is called the __t distribution__.  Using this population standard deviation estimate we can then compute how far from the population mean our observed measurement is (e.g. the sample mean), just as we did for the Z-test.  In this case since we used the t distribution to make our estimate of the population standard deviation, we call this number (distance) the __t score__.  This is what defines the test as being a __t-test__ instead of a __Z-test__.  That's it.  Easy.

There's one catch -- once the sample size gets over 30, the values returned by the t distribution start to match those from the Normal distribution.  As a result, once the sample size is over 30 we can just use the normal distribution, and hence just σ instead of sample population in our equation (basically, the Z-test).  Nice.

> “When a sample has more than 30 observations, the normal distribution can be used in place of the t distribution. ~ Applied Statistics for Public and Nonprofit Administration (Meier et.al) p. 191.”

One thing to note is that, in the real world we often do not have the "true" population standard deviation or mean (which we can also estimate from a sample).  For example, how on earth could we measure every single person in the USA in some metric?  By the time we got finished, some of the people in our database would move or even die.  So, because of this the __t-test__ is used more commonly.  If all information is known, the __Z-test__ would naturally be preferred because we are using the actual Normal Distribution since there is less unknown information.

<BR>

###### Glossary
Symbol | Definition
-------|-----------
__n__  | sample size  
__µ__  | population mean  
__σ__  | population standard deviation (sigma)  
__X__  | sample mean  
__s__  | sample standard deviation  
__Z__  | Z score  
__t__  | t score  

<BR>

##### Z-Score and t-Score Calculations
__Z score__:
+ (X - μ) / [σ / √(n)]

__t score__ (n <= 30):  
+ (X – μ) / [s / √(n)]

__t score__ (n > 30):  
+ (X – μ) / [σ / √(n)] = Z score!

<BR>

__Note:__   
The two calculations have the same numerator: the difference between the observed sample mean and the population mean.  The difference between the formulae comes in the denominator where we have to use an estimate for the population standard deviation in the t-test that comes from our sample standard deviation.

<BR>

##### Types of t-tests
There are two primary types of t-tests.
+ Student's t-test
    + Needs equal variances between the two groups being tested (unless sample sizes are equal).
    + Rule of thumb: if ratio of larger variance to smaller variance is 2x or less, consider equal.
    + Biased if sample sizes and variances are unequal (too liberal).
+ Welsh's t-test
    + Insensitive to variance differences between groups.
    + Less prone to bias.

The most prevalent t-test is the __Student's t-test__.  (This shouldn't be the case anymore -- more on this below).  Originally created by [a chemist at the Guinness factory in Dublin](https://en.wikipedia.org/wiki/Student%27s_t-test#History).  It's used to test one or two groups.

##### Examples of t-tests (Wiki)
> There are various t-tests and two most commonly applied tests are the _one-sample_ and _paired-sample_ t-tests. _One-sample_ t-tests are used to compare a sample mean with the known population mean. _Two-sample_ t-tests, on the other hand, are used to compare either independent samples or dependent samples against each other.

+ A one-sample location test of whether the mean of a population has a value specified in a null hypothesis.
    + Is the sample's mean we're measuring in the range we expect it to be?

+ A two-sample location test of the null hypothesis such that the means of two populations are equal.
    + Are the two groups' means distinct from each other?
    + Student's t-tests
        + Assumes equal variances in the two groups being tested.
    + Welch's t-test
        + Accounts for different / uncertain variances in the two groups being tested.  Often referred to as "__unpaired__" or "__independent samples__" t-tests.

+ A test of the null hypothesis that the difference between two responses measured on the same statistical unit has a mean value of zero.  
    + Often referred to as the "__paired__" or "__repeated measures__" t-test.
    + For example, suppose we measure the size of a cancer patient's tumor before and after a treatment. If the treatment is effective, we expect the tumor size for many of the patients to be smaller following the treatment.

+ A test of whether the slope of a regression line differs significantly from 0.

<BR>

Note here that "location" refers to the location of the statistic (e.g. mean) on a distribution plot; is it located close or far from where we expect it to be?  

The biggest thing to take away is that a Student's t-test assumes the variances of the two groups/populations in the test are equal.  A quick rule of thumb for this assumption is to take the ratio of the larger variance to the smaller.  _If it is less than 2x, it is probably okay to go ahead and test with Student's t-test_.  If it 2x or more larger, we should use the Welsh's t-test, which assumes differing variances, and can also pool variances.  Technically speaking, the Student's t-test is robust against differing variances if the sample sizes of the two groups being tested are the same.  Welsh's t-test works regardless of sample sizes.

In fact, it [can be convincingly argued](http://daniellakens.blogspot.com/2015/01/always-use-welchs-t-test-instead-of.html) that we should just use Welsh's t-test by default, because if the variances are the same we will get the same result as we would with the Student's t-test (so, no penalty for using Welsh's), and if they're different we will get the correct answer whereas a Student's t-test will be incorrect.


#### Actually Using the Tests
1. test for normality -- probability plot
    ```python
    import numpy as np
    from matplotlib import pylab as plt
    import scipy.stats as stats

    measurements = np.random.normal(loc = 20, scale = 5, size=100)   
    stats.probplot(measurements, dist="norm", plot=plt)
    plt.show()
    ```


    plot many on subplot:
    ```python
    def qq_plot(data=[], dist='norm'):
        fig = plt.figure(figsize=(10,8))

        for feat in range(len(data)):
            plots = math.ceil(len(data)/2)
            ax = fig.add_subplot(plots, 2, feat+1)
            stats.probplot(data[feat].values, dist=dist, plot=ax)
            ax.set_title("QQ Plot for {}".format(data[feat].name))

        fig.tight_layout()
        plt.show()
    ```






    if not normal can still do tests...


2. do tests

...comparing t/z score v. cutoff point α, p-value...


[Great examples](http://www.kean.edu/~fosborne/bstat/07b2means.html)

...
An F-test is used to compare 2 populations’ variances. The samples can be any size. It is the basis of ANOVA.



> “When your sample size is less than 30 and the population standard deviation is known, which do u use for cases where your population is normally distributed and where it is not?”
If you have a small sample size that you know is normally distributed, use the z-score.
>
>“When the sample size is greater than 30 and population standard deviation is unknown, which do you use for cases where your population is normally distributed and where it is not?”
If you don’t know the population SD, there’s no way of knowing if your data is normal. So, use the t-score.
>
>“When your sample size is less than 30 and the population standard deviation is unknown, which do u use for cases where your population is normally distributed and where it is not?”
If you know it’s normal, use the z. Although if you don’t know the population SD, how can you know if the distribution is normal?? I’d use the t for this scenario.
>
>“When your sample size is greater than 30 and the population standard deviation is known, which do u use for cases where your population is normally distributed and where it is not?”
>If your data is normal, use the z. If not, figure out what distribution it is and use the appropriate table (i.e. t, chi square etc.). If you have no idea what the distribution is, use the t but be cautious when interpreting your results.






<BR><BR><BR><BR><BR><BR>







## DSI Lecture Notes
Understanding *why* we use certain distributions to model our data is the important key.  It isn't important to memorize every aspect about every dist.  We can look that up.  We need to know have a *feel* for the dists.

If we don't know the parameters of an observed dist, we can estimate them using various techniques.  The two we are covering are:
+ Model of Moments (MOM)
    + Older, but still effective in general.

+ Maximum Likelihood Estimation (MLE)
    + Ubiquitous method in general data science and statistics for estimating parameters.  See slides from Week 2 for more detail.  In short, it is a cost function and attempts to maximize it for your data points and given parameter.

##### Sampling Distribution
The distribution of a repeatedly sampled parameter from a parent population (or parent sample).
1. Given a parent population _N_, take a sample size _A_ from the pop and compute the mean of that sample.  
2. Repeat this _X_ number of times, and then plot the distribution of these means.  This is now the _sampling distribution_ of the mean for that pop.  
3. Aggregate functions, like the mean, will obey the Central Limit Theorem, more clearly as _X_ number of repeat samplings increases.  
4. Other summary statistics, like the sample max(), do not obey the CLT -- their sampling distribution is strictly dependent upon the underlying parent population distribution.  This can help identify some aspects of the parent distribution (i.e. the non-CLT statistics can show the 'shape' of the underlying distribution), but makes other inferences about the parent population impossible since those inferences, such as the true mean of the population, depend on the CLT to be correct.

<BR>

##### Hypothesis Testing
Basically, "prove it." The null hypo is always the 'standard' or the 'normal' mode, and the alternative hypo is the 'claim' you are making.  Think your coffee makes workers more productive than the 'normal' coffee they're already using? Prove it.  Think your medicine works better than the 'normal' one? Prove it.  Using confidence levels allows us to say how certain we are something is 'proved' if the hypo test says it is.

A quick rule of thumb for determining if the variances of the two samples used in the hypo test are equal is to take the ratio of the larger var to the smaller.  If it is less than 2, probably okay to go ahead and test without variance modification (Welsh t-test assumes diff variances, and can also pool variances).

Note that by the design of hypothesis testing, we are more likely to fail to reject the null than is correct, much like "innocent until proven guilty" in our judicial system -- the onus is upon the alt. hypo to confirm validity ('guilty').


##### Five Steps to a Hypothesis Test
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



#### Statistical Power and Bayes
+ Two-sided test
    + when the hypothesis is *not equal* to something  
+ One-sided test
    + when the hypo is *greater than* or *less than* something  
+ Sensitivity
    + true positive rate  
+ Specificity
    + α rate.  Amount of allowed Type I error we are willing to accept.  
+ Power
    + the probability that we **correctly reject null** when the null hypo *is* false. (1 - β).  

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





## Bayes
Priors often are distributions and not merely a single value.
Commonly we evaluate Bayes theorem by P(H|E) proportional to P(E|H) * P(H) after considering the denominator of P(E) as a normalizing constant.













To test for normality in Scipy:
1. use Shapiro-Wilk test (think the null is not normal)
2. use a Q-Q plot, if linear -> assume normality

In a hypothesis test, if the stdev used is not from the sample, the test is a Z-test and not a t-test.
