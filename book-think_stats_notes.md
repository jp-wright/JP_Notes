# Think Stats by Allen Downey
Think Stats is an 'interactive' book, if you will, by virtue of using Jupyter Notebooks.  It also includes a lot of code modules that are used throughout the book which Downey wrote just for this purpose.  So, using them is advisable when going through the book.  In my opinion, they are written at a high enough level to actually use anytime in dealing with distributions and stats -- just `import thinkstats2 as ts2` in the correct directory (or simply put the module somewhere and add to PYTHONPATH).  

These notes will skip over much of the implementation of the ts2 module.  These notes are geared for statistical concepts, regardless of module used.  For more hands-on examples of how to use the excellent stats modules provided with this book, please _see the PDF_.  That said, some will be included as well as general Python implementations when appropriate.

I think, looking to the future, perhaps the single biggest thing to be gained from _Think Stats_ is studying how Downey created his own statistics library for this book.  Learning how he built the dependencies for separate classes and the concise and effective structures of each function will provide a lasting blueprint for how to be a better programmer.

Initially I planned to make concise personal notes on the book, but as I began reading it I realized that Downey had done such a fantastic job of providing only essential information with great examples that I didn't have much room for improvement.  So, by and large what follows below are whole cloth excerpts with a bit thrown in here and there by me.  My longer personal notes on some of these topics are in the `jp_prob_stats_notes.md` file.


## Empirical Distributions
### 1. Histograms
Counts up items in a list / series / sample and tells you how many times they appear, or their _frequency_.  Helps reveal the distribution of the data.  

In python, one easy way to think about histograms is as a dictionary with occurrences / sample possibilities as the _keys_ and their counts as _values_.

```python
# randomly selection IQs from 10,000 people.  Make into 20 bins.
hist, edges = np.histogram(np.random.normal(100, 15, 10000), bins=np.arange(35, 65, 5))

# create dict with IQ scores as keys and frequency/count as values
histo_dict = {k: v for k, v in zip(edges.astype(int), hist)}

# Output
{35: 0,
 40: 1,
 45: 1,
 50: 6,
 55: 24,
 60: 61,
 65: 115,
 70: 264,
 75: 427,
 80: 678,
 85: 897,
 90: 1234,
 95: 1321,
 100: 1292,
 105: 1146,
 110: 898,
 115: 678,
 120: 489,
 125: 259,
 130: 127,
 135: 46,
 140: 25,
 145: 9,
 150: 2,
 155: 0}
```

These are the raw counts for how many people out of 1,000 scored a given result on an IQ exam.  Considering the data was made by sampling randomly from a normal distribution, we expect the data to be normally distributed (especially at a sample size = 1,000).  This is nice as a cursory inspection, but it can be misleading if we compare two histograms of distinctly differently-sized populations.  Below we will see how to normalize the distribution so that we can compare _probabilities_ instead of _frequencies_, thus obviating sample size issues.

Note this is technically a _binned_ distribution because there are some scores not reported, such as 102, which someone surely did score.  Instead it represents the range between the two numbers, inclusive of the lower value and up to but not including the upper value (e.g. 35-39, 40-44, etc.).


#### Effect size
An effect size is a summary statistic intended to describe (wait for it) the size of an effect. For example, to describe the difference between two groups, one obvious choice is the difference in the means.  

Mean pregnancy length for first babies is 38.601; for other babies it is 38.523. The difference is 0.078 weeks, which works out to 13 hours. As a fraction of the typical pregnancy length, this difference is about 0.2%.  

If we assume this estimate is accurate, such a difference would have no practical consequences. In fact, without observing a large number of pregnancies, it is unlikely that anyone would notice this difference at all.

Another way to convey the size of the effect is to compare the difference between groups to the variability within groups. Cohen’s _d_ is a statistic intended to do that; it is defined

\[
d = \frac{x_1 − x_2}{s}  
\]

where _x<sub>1</sub>_ and _x<sub>2</sub>_ are the means of the groups and _s_ is the “pooled standard deviation.”

(non LaTeX: _d = (x<sub>1</sub> - x<sub>2</sub>) / s_)

Here’s the Python code that computes Cohen’s _d_:

```python
def CohenEffectSize(group1, group2):
    diff = group1.mean() - group2.mean()
    var1 = group1.var()
    var2 = group2.var()
    n1, n2 = len(group1), len(group2)
    pooled_var = (n1 * var1 + n2 * var2) / (n1 + n2)
    d = diff / math.sqrt(pooled_var)
    return d
```

In this example, the difference in means is 0.029 standard deviations, which is small. To put that in perspective, the difference in height between men and women is about 1.7 standard deviations (see https://en.wikipedia.org/wiki/Effect_size).




<BR><BR>

### 2. Probability Mass Function - PMF
#### PMFs
Another way to represent a distribution is a probability mass function (PMF), which maps from each value to its _probability_. A probability is a frequency expressed as a fraction of the sample size, $n$. To get from frequencies to probabilities, we divide through by $n$, which is called _normalization_.


Using the `histo_dict` from above, we can convert the frequencies of the histogram data to the probabilities of a PMF by dividing the raw count value by the total number of observations.  In this case, the total number of samples/observations was 1,000, which means the probability is found simply by moving the decimal four spots over.  However, that is usually not the case, so we will do the math quickly in a list and dict comp.

```python
probs = [val / sum(histo_dict.values()) for val in histo_dict.values()]
pmf_dict = {k: p for k, p in zip(histo_dict.keys(), probs)}

# output
{35: 0.0,
 40: 0.0001,
 45: 0.0001,
 50: 0.0006,
 55: 0.0024,
 60: 0.0061,
 65: 0.0115,
 70: 0.0264,
 75: 0.0427,
 80: 0.0678,
 85: 0.0897,
 90: 0.1234,
 95: 0.1321,
 100: 0.1292,
 105: 0.1146,
 110: 0.0898,
 115: 0.0678,
 120: 0.0489,
 125: 0.0259,
 130: 0.0127,
 135: 0.0046,
 140: 0.0025,
 145: 0.0009,
 150: 0.0002,
 155: 0.0}
```

Again, this PMF has been binned because the histogram data it was built from was binned.  This is a common occurrence in using PMFs, but there is an 'art' to it as using too many bins doesn't 'capture' the data's trends well but too few bins loses information.  One way around this is to use a CMF, discussed below.


Or we can also use the nice built in methods and objects from the `thinkstats2` module that accompanies this book.  Given a Hist object from `thinkstats2`, we can make a dictionary that maps from each value to its probability:
```python
n = hist.Total()
d = {}
for x, freq in hist.Items():
d[x] = freq / n
```

Or we can use the `Pmf` class provided by thinkstats2. Like `Hist`, the `Pmf` constructor can take a list, pandas Series, dictionary, `Hist`, or another `Pmf` object. Here’s an example with a simple list:

```bash
>>> import thinkstats2
>>> pmf = thinkstats2.Pmf([1, 2, 2, 3, 5])
>>> pmf

Pmf({1: 0.2, 2: 0.4, 3: 0.2, 5: 0.2})
```

The `Pmf` is normalized so total probability is 1.
`Pmf` and `Hist` objects are similar in many ways; in fact, they inherit many of their methods from a common parent class. For example, the methods Values and Items work the same way for both. The biggest difference is that _a `Hist` maps from values to integer counters; a `Pmf` maps from values to floating-point probabilities._

Note that modifying a probability within a PMF will result in a cumulative probability of the PMF not adding up to 1 (which makes the conclusions drawn from a PMF useless).  In order to fix this we must re-normalize (divide through by the total probability to make it 1 again) the PMF before we can use it again.  Intuitively, if we drop the probability for one item, the probabilities for the remaining items in the PMF will increase and vice versa.

If you are given a PMF, you can still compute the mean, but the process is slightly different:
$$
\overline{x} = \sum_{i} p_i \times x_i
$$

where the $x_i$ are the unique values in the PMF and $p_i = PMF(x_i)$. Similarly, you can compute variance like this:
$$
S^2 = \sum_{i} p_i (x_i − \overline{x})^2
$$

The reason a PMF is valuable is that it allows to use cut through differing sample sizes and compare simple probabilities.  They also allow the flexibility to bias or unbias a distribution.  Below is an example of taking in a PMF of marathon runners' speeds and choosing a given speed that someone might run at and seeing how the other runners appear to someone running that speed (e.g. the distribution will shift based on how fast you run since you will pass/be passed at different rates).

```python
def ObservedPmf(pmf, speed, label=None):
    """Returns a new Pmf representing speeds observed at a given speed.

    The chance of observing a runner is proportional to the difference
    in speed.

    Args:
        pmf: distribution of actual speeds
        speed: speed of the observing runner
        label: string label for the new dist

    Returns:
        Pmf object
    """
    new = pmf.Copy(label=label)
    for val in new.Values():
        diff = abs(val - speed)
        new[val] *= diff
    new.Normalize()
    return new
```


#### Chapter Glossary
+ __Probability mass function (PMF)__:  
    A representation of a distribution as a function that maps from values to probabilities.

+ __Probability__:  
    A frequency expressed as a fraction of the sample size.

+ __Normalization__:  
    The process of dividing a frequency by a sample size to get a probability.


<BR><BR>

### 3. Cumulative Distribution Function - CDF
#### Percentiles
If you have taken a standardized test, you probably got your results in the form of a raw score and a _percentile rank_. In this context, the percentile rank is the fraction of people who scored lower than you (or the same). So if you are “in the 90th percentile,” you did as well as or better than 90% of the people who took the exam.

Here’s how you could compute the percentile rank of a value, `your_score`, relative to the values in the sequence scores:

```python
def PercentileRank(scores, your_score):
    count = 0
    for score in scores:
        if score <= your_score:
            count += 1

    percentile_rank = 100.0 * count / len(scores)
    return percentile_rank
```
As an example, if the scores in the sequence were 55, 66, 77, 88 and 99, and you got the 88, then your percentile rank would be 100 * 4 / 5 which is 80.

If you are given a value, it is easy to find its _percentile rank_; going the other way is slightly harder. If you are given a percentile rank and you want to find the corresponding value, one option is to sort the values and search for the one you want:

```python
def Percentile(scores, percentile_rank):
    scores.sort()
    for score in scores:
        if PercentileRank(scores, score) >= percentile_rank:
            return score
```

The result of this calculation is a _percentile_. For example, the 50th percentile is the value with percentile rank = 50. In the distribution of exam scores above, the 50th percentile is 77.  This implementation of `Percentile` is not efficient. A better approach is to use the percentile rank to compute the index of the corresponding percentile:

```python
def Percentile2(scores, percentile_rank):
    scores.sort()
    index = percentile_rank * (len(scores)-1) // 100
    return scores[index]
```

The difference between “percentile” and “percentile rank” can be confusing, and people do not always use the terms precisely. To summarize, `PercentileRank` takes a value and computes its percentile rank in a set of values; `Percentile` takes a percentile rank and computes the corresponding value.


#### CDF
The CDF is the function that maps from a _value_ to its _percentile rank_.
The CDF is a function of _x_, where _x_ is any value that might appear in the distribution. To evaluate _CDF(x)_ for a particular value of _x_, we compute the fraction of values in the distribution less than or equal to _x_.

Here’s what that looks like as a function that takes a sequence, `sample`, and a value, `x`:

```python
def EvalCdf(sample, x):
    count = 0.0
    for value in sample:
        if value <= x:
            count += 1
    prob = count / len(sample)
    return prob
```

This function is almost identical to `PercentileRank`, except that the result is a probability in the range 0–1 rather than a percentile rank in the range 0–100.


As an example, suppose we collect a sample with the values `[1, 2, 2, 3, 5]`. Here are some values from its CDF:  

__CDF(0) = 0__  
CDF(1) = 0.2  
CDF(2) = 0.6  
CDF(3) = 0.8  
__CDF(4) = 0.8__  
CDF(5) = 1

Note the two bold items, 0 and 4.  We can evaluate the CDF for any value of _x_, not just values that appear in the sample. If _x_ is less than the smallest value in the sample, _CDF(x)_ is 0.  If it's greater than the largest value, _CDF(x)_ is 1.  Since the increments are discrete, a CDF is what's known as a _step function_.


The CDF is approximately a straight diagonal line, which means that the distribution is uniform. That outcome might be non-obvious, but it is a consequence of the way the CDF is defined. What this figure shows is that 10% of the sample is below the 10th percentile, 20% is below the 20th percentile, and so on, exactly as we should expect.

The biggest advantage offered by CDFs and percentile ranks is their comparative transferability.  We can convert the value or result in one sample or field to a percentile rank and then find the same rank in a different group (that is applicable) for a cross-group comparison.  For example, we can convert the raw IQ score of an eight-year-old child to the corresponding percentile rank amongst his age group and then find the value that equals the same percentile rank for adults.  This allows us to determine how well a child did compared to an adult, etc.

#### Chapter Glossary
+ __percentile rank__:  
    The percentage of values in a distribution that are less than or equal to a given value.

+ __percentile__:  
    The value associated with a given percentile rank.

+ __cumulative distribution function (CDF)__:  
    A function that maps from values to their cumulative probabilities. $CDF(x)$ is the fraction of the sample less than or equal to $x$.

+ __inverse CDF__:  
    A function that maps from a cumulative probability, $p$, to the corresponding value.

+ __median__:  
    The 50th percentile, often used as a measure of central tendency.

+ __interquartile range__:  
    The difference between the 75th and 25th percentiles, used as a measure of spread.

+ __quantile__:  
    A sequence of values that correspond to equally spaced percentile ranks; for example, the quartiles of a distribution are the 25th, 50th and 75th percentiles.

+ __replacement__:  
    A property of a sampling process. “With replacement” means that the same value can be chosen more than once; “without replacement” means that once a value is chosen, it is removed from the population.

<BR><BR>

## Modeling Distributions
The distributions we have used so far are called _empirical distributions_ because they are based on empirical observations, which are necessarily finite samples.

The alternative is an _analytic distribution_, which is characterized by a CDF that is a mathematical function. Analytic distributions can be used to model empirical distributions. In this context, a model is a simplification that leaves out unneeded details. This chapter presents common analytic distributions and uses them to model data from a variety of sources.  There are many other distributions not covered here, such as the _binomial_, _beta_, _gamma_, _Poisson_, _Bernoulli_, _chi_, and _Weibull_.

### 1. Exponential Distribution
\[
CDF(x) = 1 − e^{−λx}
\]

The parameter, _λ_, determines the shape of the distribution. Below we see what this CDF looks like with _λ_ = 0.5, 1, and 2.

<img src="images/exponential_cdf.png" width="400" height="400">

In the real world, exponential distributions come up when we look at a series of events and measure the times between events, called _inter-arrival times_. If the events are equally likely to occur at any time, the distribution of inter-arrival times tends to look like an exponential distribution.

One real-world example is the time between births.  Looking at the this inter-arrival time for 44 births from the database, we can look at the CDF and its cousin the _complementary CDF (CCDF)_ to see if the data is from an exponential distribution.  A CCDF is simply _1 - CDF(x)_.  If the sampling distribution is from an exponential distribution, the CCDF will be a straight line _if plotted on a log-y scale._

This works because if you plot a CCDF of a dataset that you think is from an exponential distribution, we expect something along the lines of:

$$
y \approx e^{-\lambda x}
$$

Taking the log of both sides:
$$
\log y \approx -\lambda x
$$

So on a log-y scale the CCDF is a straight line with slope _−λ_

<img src="images/complementary_cdf.png" width="800" height="400">

CDF on the left, CCDF with log-y scale on the right.  It is not exactly straight, which indicates that the exponential distribution is not a perfect model for this data. Most likely the underlying assumption —- that a birth is equally likely at any time of day —- is not exactly true. Nevertheless, it might be reasonable to model this dataset with an exponential distribution. With that simplification, we can summarize the distribution with a single parameter.

The parameter, _λ_, can be interpreted as a rate; that is, the number of events that occur, on average, in a unit of time. In this example, 44 babies are born in 24 hours, so the rate is _λ = 0.0306_ births per minute. The mean of an exponential distribution is _1/λ_, so the mean time between births is 32.7 minutes.

<BR><BR>

### 2. Normal Distribution
The normal distribution, also called _Gaussian_, is commonly used because it describes many phenomena, at least approximately. It turns out that there is a good reason for its ubiquity, which we will get to later.

The normal distribution is characterized by two parameters: the mean, $μ$, and standard deviation $σ$. The normal distribution with _μ = 0_ and _σ = 1_ is called the _standard normal distribution_. Its CDF is defined by an integral that does not have a closed form solution, but there are algorithms that evaluate it efficiently. One of them is provided by SciPy: `scipy.stats.norm` is an object that represents a normal distribution; it provides a method, `cdf`, that evaluates the standard normal CDF.


<img src="images/gaussian_cdf.png" width="400" height="400">

##### Checking for Normality
Matching the CDF of a normal distribution with best-fit parameters to a CDF of real data, such as birth weights or inter-arrival times, is one way to determine if a normal distribution would serve as a good model for the data.  Another way is to use a _probability plot_.  

#### Probability Plot
Here's the "easy" way to make a probability plot.
1. Sort the values in the sample.  
2. From a standard normal distribution (_μ = 0_ and _σ = 1_), generate a random sample with the same size as the sample, and sort it.  
3. Plot the sorted values from the sample versus the random values.

If the distribution of the sample is approximately normal, the result is a straight line with intercept mu and slope sigma. `thinkstats2` provides `NormalProbability`, which takes a sample and returns two NumPy arrays:

`xs, ys = thinkstats2.NormalProbability(sample)`  
+ `ys` contains the sorted values from sample.  
+ `xs` contains the random values from the standard normal distribution.


Now let’s try it with real data. Here’s code to generate a normal probability plot for the birth weight data from the previous section. It plots a gray line that represents the model and a blue line that represents the data.

```python
def MakeNormalPlot(weights):
    mean = weights.mean()
    std = weights.std()
    xs = [-4, 4]
    fxs, fys = thinkstats2.FitLine(xs, inter=mean, slope=std)
    thinkplot.Plot(fxs, fys, color='gray', label='model')
    xs, ys = thinkstats2.NormalProbability(weights)
    thinkplot.Plot(xs, ys, label='birth weights')
```

+ `weights` is a pandas Series of birth weights; mean and std are the mean and standard deviation.

+ `FitLine` takes a sequence of `xs`, an intercept, and a slope; it returns `xs` and `ys` that represent a line with the given parameters, evaluated at the values in `xs`.

+ `NormalProbability` returns `xs` and `ys` that contain values from the standard normal distribution and values from weights. If the distribution of weights is normal, the data should match the model.

<img src="images/normal_prob_plot_birth.png" width="400" height="400">

This figure shows the results for all live births, and also for full term births (pregnancy length greater than 36 weeks). Both curves match the model near the mean and deviate in the tails. The heaviest babies are heavier than what the model expects, and the lightest babies are lighter.

Notice the excellence of fit.  We have seen here that "checking for normality" isn't too difficult and that depending on our goal overall, we may not need our dataset to be perfectly modeled by a distribution.  If the outliers / extreme values are unimportant in the case above, the normal distribution _does_ do a good job of modeling the birth weights from about -1.5 σ to +2 σ.  If that's the range we're interested in, then great.  

When we select only full term births, we remove some of the lightest weights, which reduces the discrepancy in the lower tail of the distribution.  This plot suggests that the normal model describes the distribution well within a few standard deviations from the mean, but not in the tails. Whether it is good enough for practical purposes depends on the purposes.


The conclusions drawn from a normal distribution model are valid _only if the dataset is normally distributed_.  This is common sense, but bears repeating.  For the example above, if we made statistical inferences about the larger newborn population outside of those σ we would be unable to have much confidence in them.  Within those ranges of σ, however, we could feel assured that our conclusions were accurate.

<BR><BR>

### 3. Lognormal Distribution
If the logarithms of a set of values have a normal distribution, the values have a _lognormal distribution_. The CDF of the lognormal distribution is the same as the CDF of the normal distribution, with _log x_ substituted for _x_.

_CDF<sub>lognormal</sub>(x) = CDF<sub>normal</sub>(log x)_


The parameters of the lognormal distribution are usually denoted _μ_ and _σ_. But remember that these parameters are not the mean and standard deviation; the mean of a lognormal distribution is _exp(μ + σ<sup>2</sup>/2)_ and the [standard deviation is ugly](http://wikipedia.org/wiki/Log-normal_distribution).

If a sample is approximately lognormal and you plot its CDF on a _log-x_ scale, it will have the characteristic shape of a normal distribution. To test how well the sample fits a lognormal model, you can make a normal probability plot using the log of the values in the sample.

<img src="images/log_linear_cdf_weight.png" width="700" height="350">  

This CDF shows the distribution of adult weights on a linear scale with a normal model (left) and the same distribution on a log scale with a lognormal model (right). The lognormal model is a better fit, but this representation of the data does not make the difference particularly dramatic.  (A close observer will see the grey line of real data skewing a bit at the extremes on the normal model (left)).


<img src="images/log_linear_prob_plot_weight.png" width="700" height="350">

This normal probability plot shows adult weights, _w_, and their logarithms, _log<sub>10</sub> w_.  Now we can see clearly that the data deviate at the extremes using the normal distribution as the analytical model while the lognormal distribution provides a good fit.  Human weight would then be described as having a lognormal distribution to it.

<BR><BR>

### 4. Pareto Distribution
The [Pareto distribution](http://wikipedia.org/wiki/Pareto_distribution) is named after the economist Vilfredo Pareto, who used it to describe the distribution of wealth. Since then, it has been used to describe phenomena in the natural and social sciences including sizes of cities and towns, sand particles and meteorites, forest fires and earthquakes.

The CDF of the Pareto distribution is:

\[
CDF(x) = 1 − \left(\frac{x}{x_m}\right)^{−α}
\]

+ _x<sub>m</sub>_ and _α_ determine the location and shape of the distribution.  
+ _x<sub>m</sub>_ is the minimum possible value.

<img src="images/pareto_cdf.png" width="400" height="400">

This shows CDFs of Pareto distributions with _x<sub>m</sub> = 0.5_ and different values of _α_.

There is a simple visual test that indicates whether an empirical distribution fits a Pareto distribution: _on a log-log scale, the CCDF looks like a straight line_. Let’s see why that works.

If you plot the CCDF of a sample from a Pareto distribution on a linear scale, you expect to see a function like:

\[
y ≈ \left(\frac{x}{x_m}\right)^{−α}
\]

Taking the log of both sides yields:

\[
\log y ≈ −α(\log x − \log x_m)
\]

So if you plot _log y_ versus _log x_, it should look like a straight line with slope
_−α_ and intercept _α * log x<sub>m</sub>_.

As an example, let’s look at the sizes of cities and towns. The U.S. Census Bureau publishes the population of every incorporated city and town in the United States.

<img src="images/ccdf_log_log_populations.png" width="400" height="400">

This shows the CCDF of populations on a _log-log scale_. The largest 1% of cities and towns, below 10<sup>−2</sup>, fall along a straight line. So we could conclude, as some researchers have, that the tail of this distribution fits a Pareto model.  On the other hand, a lognormal distribution also models the data well.

<img src="images/cdf_log-x_norm_prob_plot_log_data_populations.png" width="700" height="350">

This plot shows the CDF of populations and a lognormal model (left), and a normal probability plot (right). Both plots show good agreement between the data and the model.  Neither model is perfect. The Pareto model only applies to the largest 1% of cities, but it is a better fit for that part of the distribution. The lognormal model is a better fit for the other 99%. Which model is appropriate depends on which part of the distribution is relevant.


### Generating Random Numbers
Analytic CDFs can be used to generate random numbers with a given distribution function, _p = CDF(x)_. If there is an efficient way to compute the inverse CDF, we can generate random values with the appropriate distribution by choosing _p_ from a uniform distribution between 0 and 1, then choosing _x = ICDF(p)_.

For example, the CDF of the exponential distribution is  
_p = 1 − e<sup>−λx</sup>_  

Solving for _x_ yields:

_x = −log(1 − p)/λ_

So in Python we can write  
```python
def expovariate(lam):
    p = random.random()
    x = -math.log(1-p) / lam
    return x
```
`expovariate` takes lam and returns a random value chosen from the exponential distribution with parameter `lam`.

Two notes about this implementation: I called the parameter `lam` because `lambda` is a Python keyword. Also, since _log 0_ is undefined, we have to be a little careful. The implementation of `random.random` can return 0 but not 1, so _1 − p_ can be 1 but not 0, so _log(1-p)_ is always defined.


#### Chapter Glossary
+ __empirical distribution__:  
    The distribution of values in a sample.

+ __analytic distribution__:  
    A distribution whose CDF is an analytic function.

+ __model__:  
    A useful simplification. Analytic distributions are often good models of more complex empirical distributions.

+ __interarrival time__:  
    The elapsed time between two events.

+ __complementary CDF__:  
    A function that maps from a value, _x_, to the fraction of values that exceed _x_, which is _1 − CDF(x)_.

+ __standard normal distribution__:  
    The normal distribution with mean 0 and standard deviation 1.

+ __normal probability plot__:  
    A plot of the values in a sample versus random values from a standard normal distribution.

<BR><BR>


### Probability Density Functions - PDF
The __derivative__ of a CDF is called a _probability density function_, or PDF.

For example, the PDF of an exponential distribution is
\[
PDF_{expo}(x) = λe^{−λx}
\]

The PDF of a normal distribution is
\[
\text{PDF}_{normal}(x) = \frac{1}{σ\sqrt{2π}} \text{exp} \left[−\frac{1}{2} \left( \frac{x−μ}{σ} \right)^2 \right]
\]
Evaluating a PDF for a particular value of _x_ is __usually not useful__. The result
    is not a probability; it is a probability _density_.  This can be difficult and unintuitive to interpret, much like _variance_ can be due to having squared units.

In physics, density is mass per unit of volume; in order to get a mass, you have to multiply density by volume or, if the density is not constant, you have to integrate over volume.

Similarly, __probability density__ measures probability per unit of _x_. In order to get a probability mass, you have to integrate over _x_.

The following example creates a NormalPdf from `thinkstats2` with the mean and variance of adult female heights, in cm. Then it computes the density of the distribution at a location one standard deviation from the mean.

```python
>>> mean, var = 163, 52.8
>>> std = math.sqrt(var)
>>> pdf = thinkstats2.NormalPdf(mean, std)
>>> pdf.Density(mean + std)
0.0333001
```

The result is about 0.03, in units of _probability mass per cm_. Again, a probability density doesn’t mean much by itself. Intuitively speaking, how are we to interpret probability mass per cm? That is not a natural unit of measure. But if we plot the Pdf, we can see the shape of the distribution:

<img src="images/pdf_kde_female_height.png" width="400" height="400">

This shows the normal density function and a KDE based on a sam- ple of 500 random heights. The estimate is a good match for the original distribution.  `thinkplot.Pdf` plots the Pdf as a smooth function, as contrasted with `thinkplot.Pmf`, which renders a Pmf as a step function. Here we see the result, as well as a PDF estimated from a sample, which we’ll compute in the next section.

#### Kernel Density Estimate - KDE
[Kernel density estimation](http://en.wikipedia.org/wiki/Kernel_density_estimation) (KDE) is an algorithm that takes a sample and finds an appropriately smooth PDF that fits the data.
`scipy` provides an implementation of KDE and `thinkstats2` provides a class called `EstimatedPdf` that uses it:

```python
class EstimatedPdf(Pdf):
    def __init__(self, sample):
        self.kde = scipy.stats.gaussian_kde(sample)
    def Density(self, xs):
        return self.kde.evaluate(xs)
```

`__init__` takes a sample and computes a kernel density estimate. The result is a `gaussian_kde` object that provides an evaluate method.

Density takes a value or sequence, calls `gaussian_kde.evaluate`, and returns the resulting density. The word “Gaussian” appears in the name be- cause it uses a filter based on a Gaussian distribution to smooth the KDE.

##### Why Use a KDE?
Estimating a density function with KDE is useful for several purposes:
+ __Visualization__:  
    During the exploration phase of a project, CDFs are usually the best visualization of a distribution. After you look at a CDF, you can decide whether an estimated PDF is an appropriate model of the distribution. If so, it can be a better choice for presenting the distribution to an audience that is unfamiliar with CDFs.

+ __Interpolation__:  
    An estimated PDF is a way to get from a sample to a model of the population. If you have reason to believe that the population distribution is smooth, you can use KDE to interpolate the density for values that don’t appear in the sample.

+ __Simulation__:  
    Simulations are often based on the distribution of a sample. If the sample size is small, it might be appropriate to smooth the sample distribution using KDE, which allows the simulation to explore more possible outcomes, rather than replicating the observed data.

<BR>

### How Distributions Relate to Each Other
At this point we have seen PMFs, CDFs and PDFs; let’s take a minute to review.  
We started with PMFs, which represent the probabilities for a discrete set of values. To get from a PMF to a CDF, you add up the probability masses to get cumulative probabilities. To get from a CDF back to a PMF, you compute differences in cumulative probabilities. We’ll see the implementation of these operations in the next few sections.

A PDF is the derivative of a continuous CDF; or, equivalently, a CDF is the integral of a PDF. Remember that a PDF maps from values to probability densities; to get a probability, you have to integrate.

To get from a discrete to a continuous distribution, you can perform various kinds of smoothing. One form of smoothing is to assume that the data come from an analytic continuous distribution (like exponential or normal) and to estimate the parameters of that distribution. Another option is kernel density estimation.

The opposite of smoothing is __discretizing__, or __quantizing__. If you evaluate a PDF at discrete points, you can generate a PMF that is an approximation of the PDF. You can get a better approximation using numerical integration.

To distinguish between continuous and discrete CDFs, it might be better for a discrete CDF to be a “cumulative mass function,” (CMF) but as far as I can tell no one uses that term.

Overall, we can see that __once we have any given distribution we can convert it to another distribution by performing some basic mathematical operations on it__ (in a nice Python function).  This is a fantastic fact that means the bigger issue is knowing what you want to present or model as opposed to struggling for the ability to do so.

<img src="images/distribution_relationships.png">


<BR><BR><BR>

## Python Implementation of Distributions in `thinkstats2`
At this point you should know how to use the basic types provided by `thinkstats2`: `Hist`, `Pmf`, `Cdf`, and `Pdf`. The next few sections provide details about how they are implemented. This material might help you use these classes more effectively, but it is not strictly necessary.

#### `Hist` Implementation
`Hist` and `Pmf` inherit from a parent class called `_DictWrapper`. The leading underscore indicates that this class is “internal;” that is, it should not be used by code in other modules. The name indicates what it is: a dictionary wrapper. Its primary attribute is `d`, the dictionary that maps from values to their frequencies.

The values can be any hashable type. The frequencies should be integers, but can be any numeric type.

`_DictWrapper` contains methods appropriate for both `Hist` and `Pmf`, including `__init__`, `Values`, `Items` and `Render`. It also provides modifier methods `Set`, `Incr`, `Mult`, and `Remove`. These methods are all implemented with dictionary operations. For example:

```python
# class _DictWrapper
    def Incr(self, x, term=1):
        self.d[x] = self.d.get(x, 0) + term
    def Mult(self, x, factor):
        self.d[x] = self.d.get(x, 0) * factor
    def Remove(self, x):
        del self.d[x]
```

`Hist` also provides `Freq`, which looks up the frequency of a given value. Because `Hist` operators and methods are based on dictionaries, these methods are constant time operations; that is, their run time does not increase as the `Hist` gets bigger.

#### `Pmf` Implementation
`Pmf` and `Hist` are almost the same thing, except that a `Pmf` maps values to floating-point probabilities, rather than integer frequencies. If the sum of the probabilities is 1, the `Pmf` is normalized.

`Pmf` provides `Normalize`, which computes the sum of the probabilities and divides through by a factor:

```python
# class Pmf
    def Normalize(self, fraction=1.0):
    total = self.Total()
if total == 0.0:
    raise ValueError('Total probability is zero.')
factor = float(fraction) / total
for x in self.d:
    self.d[x] *= factor
return total
```

+ `fraction` determines the sum of the probabilities after normalizing; the default value is 1. If the total probability is 0, the `Pmf` cannot be normalized, so `Normalize` raises `ValueError`.

`Hist` and `Pmf` have the same constructor. It can take as an argument a `dict`, `Hist`, `Pmf` or `Cdf`, a pandas `Series`, a list of (value, frequency) pairs, or a sequence of values.

If you instantiate a `Pmf`, the result is normalized. If you instantiate a `Hist`, it is not. To construct an unnormalized `Pmf`, you can create an empty `Pmf` and modify it. The `Pmf` modifiers do not renormalize the `Pmf`.

#### `Cdf` Implementation
A CDF maps from values to cumulative probabilities, so I could have implemented `Cdf` as a `_DictWrapper`. But the values in a CDF are ordered and the values in a `_DictWrapper` are not. Also, it is often useful to compute the inverse CDF; that is, the map from cumulative probability to value. So the implementation I chose is two sorted lists. That way I can use binary search to do a forward or inverse lookup in logarithmic time.

The `Cdf` constructor can take as a parameter a sequence of values or a pandas Series, a dictionary that maps from values to probabilities, a sequence of (value, probability) pairs, a `Hist`, `Pmf`, or `Cdf`. Or if it is given two parameters, it treats them as a sorted sequence of values and the sequence of corresponding cumulative probabilities.

Given a sequence, pandas Series, or dictionary, the constructor makes a `Hist`. Then it uses the `Hist` to initialize the attributes:

```python
self.xs, freqs = zip(*sorted(dw.Items()))
        self.ps = np.cumsum(freqs, dtype=np.float)
        self.ps /= self.ps[-1]
```

+ `xs` is the sorted list of values
+ `freqs` is the list of corresponding frequencies.
+ `np.cumsum` computes the cumulative sum of the frequencies.

Dividing through by the total frequency yields cumulative probabilities. For _n_ values, the time to construct the `Cdf` is proportional to _n log n_.

Here is the implementation of `Prob`, which takes a value and returns its cumulative probability:

```python
# class Cdf
def Prob(self, x):
    if x < self.xs[0]:
        return 0.0
    index = bisect.bisect(self.xs, x)
    p = self.ps[index - 1]
    return p        
```

+ The `bisect` module provides an implementation of binary search.

And here is the implementation of `Value`, which takes a cumulative probability and returns the corresponding value:

```python
# class Cdf
def Value(self, p):
    if p < 0 or p > 1:
        raise ValueError('p must be in range [0, 1]')
    index = bisect.bisect_left(self.ps, p)
    return self.xs[index]        
```

Given a `Cdf`, we can compute the `Pmf` by computing differences between consecutive cumulative probabilities. If you call the `Cdf` constructor and pass a `Pmf`, it computes differences by calling `Cdf.Items`:

```python
# class Cdf
def Items(self):
    a = self.ps
    b = np.roll(a, 1)
    b[0] = 0
    return zip(self.xs, a-b)        
```

+ `np.roll` shifts the elements of `a` to the right, and “rolls” the last one back to the beginning. We replace the first element of `b` with 0 and then compute the difference a-b. The result is a NumPy array of probabilities.

`Cdf` provides `Shift` and `Scale`, which modify the values in the `Cdf`, but the probabilities should be treated as immutable.


#### Moments
Any time you take a sample and reduce it to a single number, that number is a statistic. The statistics we have seen so far include mean, variance, median, and interquartile range.

A raw moment is a kind of statistic. If you have a sample of values, _x<sub>i</sub>_, the
 _kth_ raw moment is:

\[
m^{'}_{k} = \frac{1}{n} \sum_{i}x^{k}_{i}
\]

Or if you prefer Python notation:

```python
def RawMoment(xs, k):
    return sum(x**k for x in xs) / len(xs)
```

When _k = 1_ the result is the sample mean, $\overline{x}$. The other raw moments don’t mean much by themselves, but they are used in some computations.

The central moments are more useful. The _kth_ central moment is:

\[
m_{k} = \frac{1}{n} \sum_{i} \left( x_{i} − \overline{x} \right) ^{k}
\]

Or in Python:  
```python
def CentralMoment(xs, k):
    mean = RawMoment(xs, 1)
    return sum((x - mean)**k for x in xs) / len(xs)
```

When _k = 2_ the result is the second central moment, which you might recognize as variance. The definition of variance gives a hint about why these statistics are called "moments." If we attach a weight along a ruler at each location, _x<sub>i</sub>_, and then spin the ruler around the mean, the moment of inertia of the spinning weights is the variance of the values. Read more about [moment of inertia](http://en.wikipedia.org/wiki/Moment_of_inertia) if unfamiliar.

When you report moment-based statistics, it is important to think about the units. For example, if the values _x<sub>i</sub>_ are in cm, the first raw moment is also in cm. But the second moment is in cm<sup>2</sup>, the third moment is in cm<sup>3</sup>, and so on.

Because of these units, moments are hard to interpret by themselves. That’s why for the second moment, variance, it is common to report standard deviation, which is the square root of variance, so it is in the same units as _x<sub>i</sub>_.


### Skewness
Skewness is a property that describes the shape of a distribution. If the distribution is symmetric around its central tendency, it is unskewed. If the values extend farther to the right, it is “right skewed” and if the values extend left, it is “left skewed.”

This use of “skewed” does not have the usual connotation of “biased.” Skewness only describes the shape of the distribution; it says nothing about whether the sampling process might have been biased.

Several statistics are commonly used to quantify the skewness of a distribution. Given a sequence of values, _x<sub>i</sub>_, the sample skewness, _g<sub>1</sub>_, can be computed like this:

```python
def StandardizedMoment(xs, k):
    var = CentralMoment(xs, 2)
    std = math.sqrt(var)
    return CentralMoment(xs, k) / std**k

def Skewness(xs):
    return StandardizedMoment(xs, 3)    
```

_g<sub>1</sub>_ is the third standardized moment, which means that it has been nor-malized so it has no units.

Negative skewness indicates that a distribution skews left; positive skewness indicates that a distribution skews right. The magnitude of _g<sub>1</sub>_ indicates the strength of the skewness, but by itself it is not easy to interpret.

In practice, computing sample skewness is usually _not a good idea_. If there are any outliers, they have a disproportionate effect on _g<sub>1</sub>_.

A better way to evaluate the asymmetry of a distribution is to look at the relationship between the mean and median. Extreme values have more effect on the mean than the median, so in a distribution that skews left, the mean is less than the median. In a distribution that skews right, the mean is greater.

_Pearson’s median skewness coefficient_ is a measure of skewness based on the difference between the sample mean and median:
\[
g_{p} = 3( \overline{x} − m) / S
\]

Where $\overline{x}$ is the sample mean, _m_ is the median, and _S_ is the standard deviation. Or in Python:

```python
def Median(xs):
    cdf = thinkstats2.Cdf(xs)
    return cdf.Value(0.5)

def PearsonMedianSkewness(xs):
    median = Median(xs)
    mean = RawMoment(xs, 1)
    var = CentralMoment(xs, 2)
    std = math.sqrt(var)
    gp = 3 * (mean - median) / std
    return gp    
```

This statistic is _robust_, which means that it is less vulnerable to the effect of outliers.

The sign of the skewness coefficient indicates whether the distribution skews left or right, but other than that, they are hard to interpret. _Sample skewness_ is less robust; that is, it is more susceptible to outliers. As a result it is less reliable when applied to skewed distributions, exactly when it would be most relevant. _Pearson’s median skewness_ is based on a computed mean and variance, so it is also susceptible to outliers, but since it does not depend on a third moment, it is somewhat more robust.


<BR><BR>

Ch 7 - all
