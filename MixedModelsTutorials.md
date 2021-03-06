--- 
title: "The MumfordBrainStats Mixed Models Series: Companion for the YouTube series"
author: "Jeanette Mumford"
date: "2020-01-31"
site: bookdown::bookdown_site
output: bookdown::gitbook
documentclass: book
cover-image: "./images/mixed_model_book_cover.png"
#bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
#github-repo: rstudio/bookdown-demo
description: ""
---


# Introduction {-}


\includegraphics[width=250px]{./images/mixed_model_book_cover} 

This is a collection of materials that accompanies a [YouTube series on the MumfordBrainStats channel about mixed models](https://www.youtube.com/watch?v=IGHm1XHFWMc&list=PLB2iAtgpI4YEAUiEQ1ZnfMXY-yewNzn9z).  Although I normall focus on material related to neuroimaging, this is for a general audience.  Each of these chapters should be understandable without watching the video, but one would probably gain the most by watching the videos as well.  The chapter titles indicate which video in the series goes along with that chapter.   Not all videos have chapter (yet), since I'm only including chapters with code for now.  In the future I may try to smooth this out more to read more like an actual book.

Although this series isn't necessarily set up to be a tutorial on how to run a mixed model properly, it would still be useful for somebody who is new to mixed models.  I'm focusing on odds and ends that I've seen come up that I feel are less frequently discussed.  Basically mistakes I've made or seen that I'd like to help you avoid.  Interestingly, many of these mistakes are made by those with some knowledge of mixed models.  I think they often arises from  a "knows enough to be dangerous" situation.  Do not misinterpret that as it is good to be dangerous, brave, make mistakes and then fix them.  That's how we learn!

Specific goals of this series:

* Make it easier for you to conceptualize what a mixed model is doing
* Describe the two stage random effects formulation and corresponding two stage summary statistics approach for modeling repeated measures data
* Clarify when regularization is occurring and what it is doing
* Clarifying what a conditional mode (BLUP) is and how they should *not* be used
* When a two stage summary statistics approach is likely okay to use

This series is still in development.  Future topics will involve: crossed/nested random effects, tips for planning out your analysis setup, simplifying random effects structures (when is it okay) and hopefully some examples from the audience of this series about their trials and tribulations with mixed models.  I'm also looking for solid ways to test model assumptions.  So far I haven't found anything super satisfying.

For feedback, you can find me on twitter: @mumbrainstats.

<!--chapter:end:index.Rmd-->

# V3: Simulation using 2-stage random effects

## Relating two-stage random effects to simulated data and model output {.unlisted}



## Introduction 
This chapter accompanies the 3rd video in the series, ["Simulating data using the 2-stage random effects formulation"](https://youtu.be/OL6UezgpmPo). The purpose of this chapter is to help in understanding the two-stage random effects formulation for mixed models by simulating data based on the formulation, analyzing the simulated data and comparing the two.  This is a great way to wrap your head around what a mixed model is doing.  This is written under the assumption that you have seen the first few videos in the [Mixed Models series](https://www.youtube.com/watch?v=IGHm1XHFWMc&list=PLB2iAtgpI4YEAUiEQ1ZnfMXY-yewNzn9z)  on the MumfordBrainStats YouTube channel.  If you haven't, go look for that playlist and watch those first.  As discussed in the videos, this is not a perfect setup for the mixed models and not all mixed models can fit into this formulation, but it is a great way to understand mixed models.  Sometimes it is helpful to see how repeated measures data are simulated in order to understand what the model is doing.  In this case the values in the simulation will show up again when the data are fit using the lmer() function.  Make sure to spend time matching up the values in the simulation code with the lmer output to increase your understanding of mixed models.  

## Data simulation {-}
The simulation follows the two-stage random effects formulation in reverse to make up some fake data that look similar to the Sleepstudy data. Typically simulations are used for estimating type I error and power, but they are great for learning too. The benefit in simulation is  the truth is known and be compared with the estimates.  Specifically,the true group intercept and slope as well as all of the variance parameters will be known in this case.  

#### Review of the two-stage formulation {-}
Reviewing the two-stage formulation, recall the first stage of the model is $$Y_i = Z_i\beta_i + \epsilon_i, \: \epsilon_i\sim N(0, \sigma^2I_{n_i})$$ and the second stage is $$\beta_i = A_i \beta + b_i, \: b_i \sim N(0,G).$$  In these early examples $A_i$ is an indentity matrix, so you can ignore it. This is because we're only averaging within-subject effects in these models.  In this case $Y_i$ contains the average reaction times for  subject $i$ over 10 days, $Z_i$ is the design matrix with a column of 1s and a column for days (0-9), $\beta_i$ is the vector with the slope and intercept for subject $i$ and $\sigma$ is the within-subject variance. Notably, this variance is assumed to be the same for all subjects, as it does not have an $i$ subscript.  Last, $n_i$, the number of observations for subject $i$, is assumed to be 10 for all subjects, so $I_{n_i}$ is a $n_i\times n_i$ identity matrix.  Here's an example using the first subject in the data set ($i$ would be 1), 

$$\left[\begin{array}{c}
265 \\ 252 \\ 451 \\ 335 \\ 376 \\370 \\ 357 \\ 514 \\ 350 \\ 500 \end{array}\right] = \left[\begin{array}{cc}
1 & 0 \\
1 & 1 \\
1 & 2 \\
1 & 3 \\
1 & 4 \\
1 & 5 \\
1 & 6 \\
1& 7 \\
1 & 8 \\
1 & 9 \end{array}\right]\left[\begin{array}{c} \beta_{0, i} \\ \beta_{1, i}  \end{array}\right] + 
\left[\begin{array}{c}
\epsilon_1 \\
\epsilon_2 \\
\epsilon_3 \\
\epsilon_4 \\
\epsilon_5 \\
\epsilon_6 \\
\epsilon_7 \\
\epsilon_8 \\
\epsilon_9 \\
\epsilon_{10} 
\end{array}\right].$$

The second level is looks like: $$\left[\begin{array}{c}\beta_{0,i} \\ \beta_{1,i}\end{array}\right] =\left[\begin{array}{c}\beta_{0} \\ \beta_{1}\end{array}\right] + \left[\begin{array}{c}b_{0,i} \\ b_{1,i}\end{array}\right],$$  since the interest is in the average intercept and slope across subjects.  

#### Data simulation based on two-stage formulation {-}

To simulate the data, one goes through the two stages backwards, starting with stage 2. This setting assumes the random slope and intercept are independent, the intercept's between-subject variance is $24^2$ and slope's between-subject variance is $10^2$, so $$G=\left[\begin{array}{cc} 24^2 & 0 \\ 0 & 10^2 \end{array}\right].$$  Another way to think of this, since the slope and intercept are independent is: $b_{0,i}\sim N(0, 24^2)$ and $b_{1,i}\sim N(0, 10^2)$. The true group intercept and slope are assumed to be 251 and 10, respectively, so subject-specific slopes and intercepts are generated using the multivariate normal distribution
$$N\left(\left[\begin{array}{c} 251 \\ 10 \end{array}\right], \left[\begin{array}{cc} 24^2 & 0 \\ 0 & 10^2 \end{array}\right]\right).$$

It may not always be the case that the slope and intercept are independent.  For example, sometimes the steepness of the slope may depend upon on where the subject started (the intercept).  If they are very slow on the task on day 0, they may not decline as much over the following days compared to a person who starts off really fast.  In this case, the off-diagonal of $G$ would have a negative covariance value.  In the interest of simplicity the independence assumption is used here, but the model fit will default to assuming they are correlated, which is fine.  More on this to come.  

To obtain the individual reaction times, the first stage is used based on the individual slope and intercept values that were generated with the multivariate Normal distribution above.  Basically noise is added to that subject's true regression line, with a variance of $30^2$ to yield the subject-specific reaction times.  The simulated data are then complete and ready for analysis.

Read through the function and pay attention to where the betas, G and sigma are.  Relate them to the equations above and later find their estimates in the lmer output!  I've flagged the numbers to take note of with comments.



```r
library(ggplot2)
library(lme4)
library(lmerTest)
library(MASS)

makeFakeSleepstudyData = function(nsub){
  time = 0:9
  rt = c() # This will be filled in via the loop
  time.all = rep(time, nsub)
  subid.all = as.factor(rep(1:nsub, each = length(time)))
  
  # Step 1:  Generate the beta_i's.  
  G = matrix(c(24^2, 0, 0, 10^2), nrow = 2)   ##!!! <- REMEMBER THESE NUMBERS!!!!
  int.mean = 251  ##!!! <- REMEMBER THESE NUMBERS TOO!!!!
  slope.mean = 10  ##!!! <- REMEMBER THESE NUMBERS TOO!!!!
  sub.ints.slopes = mvrnorm(nsub, c(int.mean, slope.mean), G)
  sub.ints = sub.ints.slopes[,1]
  time.slopes = sub.ints.slopes[,2]
  
  # Step 2:  Use the intercepts and slopes to generate RT data
  sigma = 30      ##!!! <- THIS IS THE LAST NUMBER TO REMEMBER!!!! (sorry for yelling)
  for (i in 1:nsub){
    rt.vec = sub.ints[i] + time.slopes[i]*time + rnorm(length(time), sd = sigma)
    rt = c(rt, rt.vec)
  }
  
  dat = data.frame(rt, time.all, subid.all)
  return(dat)
}
```

## Generate and plot data

The following generates data for 16 subjects and plots them.  This should look somewhat like the real sleepstudy data that were discussed in an earlier video.


```r
set.seed(10) # this ensures you get what I got here
dat = makeFakeSleepstudyData(16)
ggplot(data = dat, aes(x = time.all, y = rt)) +
  geom_line() + 
  geom_point() +
  facet_wrap(~subid.all)
```

![](1_video3_2_level_illustration_files/figure-latex/unnamed-chunk-2-1.pdf)<!-- --> 



## Run model and compare to simulation settings {-}




Below is the appropriate model, which includes a random slope an intercept as well as a correlation between the two.  To compare with the simulation values, first focus on the "Random Effects" section.  This is where to find the  between-subject variances for the slope and intercept as well as an estimate of the correlation between the two.  The within-subject variance is also in this part of the output.  Be generous when comparing these estimates with the truth since only 16 subjects with 10 observations each went into the analysis and the estimates are likely noisy. Recall from the $G$ matrix above that the between subject variance for the intercept was set to $24^2$, the between subject variance for the slope was $10^2$ and the correlation between the two was 0. The variances roughly match the estimates, specifically 26.19$^2$ and 9.77$^2$.  The correlation is estimated at -0.33.  Later the topic of testing and simplifying random effects structures will be covered.  Last, the within-subject variance was set to 30 in the simulations and is the Residual entry (last row) in the Random Effects section, 28.42$^2$.  

Moving on to the fixed effects column, our simulated intercept and slope values were 251 and 10, respectively.  The estimates are pretty close: 253.56 and 16.59.  Also take note of the degrees of freedom column.  Why only 15?  This actually makes a lot of sense when thinking about the two stage model.  In the first stage a separate slope and intercept are estimated for each subject and then the second stage averages these.  With 16 estimates, the degrees of freedom for an average would be 15, so this makes sense.  Always report and ask others to report degrees of freedom, because it is a quick way to sense whether their random effects structure may have not been correct.  More on this below.


```r
summary(lmer(rt~time.all + (1+time.all |subid.all), dat))
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: rt ~ time.all + (1 + time.all | subid.all)
##    Data: dat
## 
## REML criterion at convergence: 1588.5
## 
## Scaled residuals: 
##      Min       1Q   Median       3Q      Max 
## -2.43358 -0.64266  0.03094  0.72321  2.18928 
## 
## Random effects:
##  Groups    Name        Variance Std.Dev. Corr 
##  subid.all (Intercept) 685.73   26.186        
##            time.all     95.49    9.772   -0.33
##  Residual              807.84   28.423        
## Number of obs: 160, groups:  subid.all, 16
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  253.559      7.765  15.000  32.653 2.38e-15 ***
## time.all      16.594      2.565  15.000   6.469 1.06e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Correlation of Fixed Effects:
##          (Intr)
## time.all -0.405
```

## Omitting random effects can inflate Type I error {-}

Omitting the random slope often a very bad idea.  Generally, all within-subject variables that are continuous should be included as random effects unless there are convergence issues because there aren't enough data to estimate them.  Definitely check out this paper by [Matusheck and others](https://www.sciencedirect.com/science/article/pii/S0749596X17300013) as well as my [video](https://www.youtube.com/watch?v=pDNEgcl0YhI) to learn about simplifying the random effects structure. 
Below the random slope is omitted and  the biggest impact is on the degrees of freedom, which are quite large now!  Of course this results in a much smaller p-value.  Generally there is a risk of inflated type I errors when omitting a random effect.  It is best to start with a fully specified random effects structure and simplify if need be.  This is why it is a good idea to report degrees of freedom and make sure they are reported when reviewing manuscripts.  


```r
bad.model = lmer(rt~time.all + (1 |subid.all), dat)
summary(bad.model)  
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: rt ~ time.all + (1 | subid.all)
##    Data: dat
## 
## REML criterion at convergence: 1666.7
## 
## Scaled residuals: 
##      Min       1Q   Median       3Q      Max 
## -2.53803 -0.64128 -0.00096  0.67704  2.71143 
## 
## Random effects:
##  Groups    Name        Variance Std.Dev.
##  subid.all (Intercept) 1771     42.08   
##  Residual              1634     40.42   
## Number of obs: 160, groups:  subid.all, 16
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  253.559     12.082  21.768   20.99 6.23e-16 ***
## time.all      16.594      1.113 143.000   14.91  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Correlation of Fixed Effects:
##          (Intr)
## time.all -0.414
```

## Summary {-}

The goal of this document was to shed some light on the two stage random effects structure to aid in understanding the output of lmer.  In addition, the importance of including random effects was shown.  Omitting random effects can greatly inflate the Type I error.  If you have question about this document, do not hesitate to contact me via the MumfordBrainStats FB page or twitter.

<!--chapter:end:1_video3_2_level_illustration.Rmd-->

# V4: Introduction to regularization in mixed models 

## What is this regularization and why don't we analyze the data in 2 stages? {.unlisted}

## Introduction

This chapter is paired with the fourth video in the mixed model series, [What is this regularization and why don't we analyze the data in 2 stages?](https://youtu.be/sRhFeC-STdw). Although the two-stage formulation is a great way to conceptualize the mixed model, it also might inspire a shortcut for data analysis: the two-stage summary statistics approach.  For those familiar with fMRI data, this is exactly what is done to analyze those data due to the complexity and structure of the data.  The approaches vary greatly across software packages where the SPM approach is most similar to what will be done here, but I will not be making comparisons with or between fMRI software.  There is a [related paper](https://www.ncbi.nlm.nih.gov/pubmed/19463958), written by myself and others that compares the models of different fMRI software packages.  

There are many reasons why the all-in-one mixed model, which I will just call a mixed model or MM from now on, is better than using the two-stage summary statistics approach (called 2SSS from now on).  For one, you are not allowed simplified or more complex random effects structures.  For example, there may be cases where a within-subject variable will not be specified as random and you cannot really do this using 2SSS.  There are other reasons that will come up as we work through this topic.  The most compelling reason for the user might be that a mixed model takes about one line of code versus many lines of code for the summary statistics approach.  

##### Regularization: key difference

What about when the 2SSS and MM formulations are quite close?  How does the mixed model result differ?  Sometimes they'll be almost identical, but other times they will be quite different.  Why? The short answer is the within-subject values related to the fixed effects are regularized in a mixed model and that is the focus here.  This is a bit of a confusing statement because within-subject estimates are not actually estimated!  Assume we are working with the sleepstudy data, which would include a within- and between-subject variability.  Recall the mixed model equation, $$Y_i = X_i\beta + Z_ib_i + \epsilon_i,$$ where $Y_i$ is the dependent variable for subject $i$, $X_i$ is the design matrix for the fixed effects (see the last chapter for more details), $\beta$ is the *group* parameter vector, $Z_i$ is a matrix for the random effects,  $b_i$ are the random effects describing between-subject variability, and $\epsilon_i$ is describes the within-subject variability. There are not any within-subject parameters estimated in this model.  The term, $b_i$ is a random variable that describes the between-subject distribution, $b_i \sim N(0, G)$.  Importantly, the $b_i$ are not estimated, but the corresponding covariance of the $b_i$, $G$,  is estimated and serves as the between-subject variability estimate.    Although the subject specific estimates are not estimated, the subjects with less data will contribute less to the estimate of the group parameter, $\beta$.  Much, much more exploration will be done on this topic as time goes on.  

For now I would like to illustrate what the regularization looks like for means, but there isn't a perfect way to illustrate it.  Typically the regularization is shown by comparing within-subject estimates to predicted within-subject estimates from the mixed model, based on values called BLUPS or best linear unbiased predictors.  Some are not fans of this term, so I will simply refer to these as conditional modes, following the notation used by Douglas Bates, the author of the lme4 library.  As stated in a draft of his book, "it is an appealing acronym, I don't find the term particularly instructive (what is a 'linear unbiased predictor' and in what context are these the 'best'?) and prefer the term conditional mode." (book can be found [here](http://webcom.upmf-grenoble.fr/LIP/Perso/DMuller/M2R/R_et_Mixed/documents/Bates-book.pdf) at the time of this writing).  The reason why this isn't the perfect way of illustrating regularization is because it doesn't perfectly reflect how the regularization ends up impacting the group-level results.  That will be the focus in the future, but now the goal is to understand that there is regularization happening.

The specific goal of the simulations below is to understand the impact of regularization by looking at estimates based on within-subject averages from the first stage of 2SSS and the conditional modes from mixed models.  The first level estimates from 2SSS will be referred to as OLS (Ordinary Least Squares) estimates or $\hat\beta_i^{OLS}$ and the conditional modes will be called just that or referred to as $\hat\beta_i$.

##### What is a conditional mode?

In an effort to keep equations at a minimum, I will explain conditional model conceptually.  For more information I recommend looking at the Bates book I linked to above or the textbook, ["Applied Longitudinal Analysis" by Fitzmaurice, Laird and Ware](https://www.amazon.com/Applied-Longitudinal-Analysis-Garrett-Fitzmaurice/dp/0470380276).  To understand the difficulty of this problem, remind yourself that typically when we have a random variable that follows a distribution, say $X \sim N(\mu, \sigma^2)$, we focus on the estimation of $\mu$ and $\sigma^2$.  Asking what value $X$ is doesn't even make much sense, since it is a a random variable.  What can be done is to use the mode of the distribution as a prediction of $X$, which is where the "mode" in "conditional  mode" comes from.  The conditioning part is somewhat simple.  The mixed model equation above describes the distribution of $Y_i$ but we want the distribution of $b_i$ for a specific subject, $i$.  To get at this, the distribution of the random effects conditional on the *data*, $Y$, is used.  The part of this process that is very different from when we estimate things, is that *estimated* parameters (within-subject variance, between-subject covariance and fixed effects) are substituted in place of the truth in the distributions in order to derive these modes instead of the true values because they are unknown.  This introduces an additional source of variability.  The take-away is simply, again, the within-subject estimates are not estimated by default in a mixed model but we can predict them using conditional modes.  These predictions can be quite noisy because they rely on estimates of parameters in order to specify the conditional distribution.

Next, some fake data will be generated and used to estimate some conditional modes and start building intuition about the regularization in mixed models.  Fake data are used for the convenience of knowing the truth.

## Simulated data

The simulated data consist of 10 subjects where 5 have 50 observations and 5 only have 5 observations from some type of behavioral experiment measuring reaction time.  As in the last chapter, the two-stage random effects formulation will be used to simulate the data.






```r
library(lme4)
library(lmerTest)
library(ggplot2)
# Simulate true subject-specific means:
nsub = 10
btwn.sub.sd = 10
#within-subject means 
win.means = rnorm(nsub, mean = 250, sd = btwn.sub.sd)

# Simualte data for each subject by wiggling around their means
win.sub.sd = 20
# The following indicates how many data per subject, the first 5 only have 5 observations.
n.per.sub = rep(50, nsub)
n.per.sub[1:5] = 5
rt = c()
subid = c()
for (i in 1:nsub){
  rt.loop = rnorm(n.per.sub[i], win.means[i], sd = win.sub.sd)
  rt = c(rt, rt.loop)
  subid = c(subid, rep(i, n.per.sub[i]))
}
dat = data.frame(subid, rt)
```

Here is a plot of the resulting data.  As you can see the first five subjects have much less data than the rest of the subjects.  This was done on purpose to illustrate the regularization since the factor that impacts the regularization is the amount of data within-subject.  There is much more to this, but it will be covered in the following chapters.



```r
ggplot(dat, aes(x=as.factor(subid), y = rt)) +
  geom_boxplot() + 
  geom_jitter(shape=16, position=position_jitter(0.2)) +
  xlab("Subject")+ylab("RT")
```

![](2_video4_two_stage_temptation_files/figure-latex/unnamed-chunk-3-1.pdf)<!-- --> 


## 2-stage summary statistics approach compared to conditional modes

Now that the data have been generated, the 2SSS uses a within-subject OLS linear regression model to obtain the within-subject means, $\hat\beta_i^{OLS}$ and then these are averaged in a second OLS model to obtain the group estimates. The following runs these two stages and also estimates the proper mixed model.


```r
# Stage 1
stage1.est = rep(NA, nsub)
for (i in 1:nsub){
  # Estimating the mean RT via OLS
  mod.loop = lm(rt ~ 1, dat[dat$subid==i,])
  stage1.est[i] = mod.loop$coef[1]
}

#Stage 2
summary(lm(stage1.est ~ 1))
```

```
## 
## Call:
## lm(formula = stage1.est ~ 1)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -17.1940  -4.7550  -0.5041   8.5768  11.5654 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  251.754      3.047   82.62 2.82e-14 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 9.636 on 9 degrees of freedom
```

```r
## Proper mixed model
mod.lmer = lmer(rt ~ 1 + (1|subid), dat)
summary(mod.lmer)
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: rt ~ 1 + (1 | subid)
##    Data: dat
## 
## REML criterion at convergence: 2449.6
## 
## Scaled residuals: 
##      Min       1Q   Median       3Q      Max 
## -2.54753 -0.69671  0.02651  0.72996  2.80661 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  subid    (Intercept)  39.81    6.309  
##  Residual             423.42   20.577  
## Number of obs: 275, groups:  subid, 10
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  253.885      2.638   6.206   96.25 4.35e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

Comparing the lm summary to the fixed effects summary from the lmer model, the results are almost identical.  Although this makes it tempting for many to use 2SSS instead of a mixed model, the next series of videos will set these two modeling approaches apart.  

For practice, find the within- and between-subject variance estimates in the lmer summary and compare to the true values used in the simulation.

## OLS versus conditional modes

To see the impact of regularization it is necessary to predict the subject-specific values from lmer.  As mentioned above, these values are not estimated in any way shape or form during the modeling process above.  In the following, the code asks lmer for these predicted values.


```r
# Mixed model conditional modes
mmcm = coef(mod.lmer)$subid[, 1]
```

The following plot compares the mixed model estimates based on the conditional mean (MMCM, orange) in orange to the within-subject OLS estimates (OLS, blue) from the first stage of the 2SSS.  The true mean was....well, the reader should practice understanding the above code by finding that themselves.  Once you've located that value above, you will notice that the orange dots (MMCM) are shrunken toward the overall group mean compared to the blue OLS estimates.  That is a result of the regularization.  Recall the first 5 subjects had much less data than the last 5 subjects, which means there's more likely to be *more* regularization for the first 5 subjects.  I will introduce an equation that describes this exactly in the next chapter, but for now just note the basic trend.  It is easiest to see if comparing subject 1 to subject 9.  In both cases their within-subject OLS estimates are almost identical, but the MMCM estimate for subject 1 is much smaller!  This is reflecting the fact that subject 1 had much less data than subject 9.  


```r
subject = rep(c(1:nsub), 2)
estimate = c(mmcm, stage1.est)
estimate.type = rep(c("MMCM", "OLS"), each = nsub)
dat.sub.est = data.frame(subject, estimate, estimate.type)

# Nice colorblind-friendly pallette
# http://www.cookbook-r.com/Graphs/Colors_(ggplot2)/#a-colorblind-friendly-palette
cbPalette <- c("#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7","#999999")


ggplot(dat.sub.est, aes(x = subject, y = estimate, 
                        color = estimate.type)) +
  geom_point()+ scale_colour_manual(values=cbPalette) + xlab("Subject")+ylab("RT") + labs(color = "Estimate Type") + 
  scale_x_continuous(breaks = seq(0 , 10, 1), minor_breaks=seq(0, 10,1))
```

![](2_video4_two_stage_temptation_files/figure-latex/unnamed-chunk-6-1.pdf)<!-- --> 

## Summary

This document is just a taste of what is to come.  This is just a beginning to convince folks analyzing data that data analysis can be done better without using the two-stage summary statistics model, so don't be tempted! There are cases where you will not have a choice, but generally you will be better off with a mixed model.  Future chapters will help clarify when these two approaches will yield similar results.

Although not a perfect way to view the phenomenon, conditional mode-based within-subject estimates from mixed models help in understanding the regularization that happens in the mixed model framework.  On the surface it can be thought of as trying to stabilize the estimate from subjects with less data.  The important take away for now is that this is driven by the amount of data within-subject.   

What inspired this series is that when I explained this point to my colleague she immediately wanted to know how this regularization might impact her between-subject analyses.  For example, what if in the fake data above the first 5 subjects were patients and the last 5 were controls.  It is a reasonable scenario that some patients may have less data than controls, which is definitely the case for my colleague's data.  If the patients' estimates are shrunken toward the overall mean because they have less data, could that introduce false positives or reduce power?  A really great question!  That is what the focus will be in the following chapters where I will use simulated data to compared different modeling approaches.  It will also reveal the misleading nature of these conditional mode estimates.


<!--chapter:end:2_video4_two_stage_temptation.Rmd-->

# V5: Conditional modes vs means

## Relationship between mixed model conditional modes (aka BLUPS) and OLS estimates {.unlisted}

## Introduction

This chapter accompanies the fifth video in the mixed models series,   ["Relationship between mixed model conditional modes (aka BLUPS) and OLS estimates"](https://youtu.be/QqEUKlKPos4).
In the last chapter, mixed models were contrasted to using the two-stage summary statistics model (2SSS) focusing on the regularization that is built into mixed models.  Specifically, the ordinary least squares (OLS) estimates from stage 1 of 2SSS were compared to the conditional mode-based subject predictions from mixed models to show the regularization of the estimates, within-subject, in the mixed model setting.  Although the conditional mode predictions are not perfect, that exploration will be started in the next segment.  Here the focus is on the details of the relationship of the OLS estimates and the conditional mode   estimates as there is an equation that describes this relationship!  Why is this exciting?  Because it helps build intuition about when these two approaches will greatly differ.  I will be briefly discussing this equation below, but for more technical details see section 8.7 of the [Applied Longitudinal Analysis text by Fitzmaurice and others](https://www.amazon.com/Applied-Longitudinal-Analysis-Garrett-Fitzmaurice/dp/0470380276). 

I will stick with the simplified example from last time, where there are multiple measures of reaction time for each subject and the interest is in the mean. Here's a reminder of how the data were generated (exactly the same code as last time and seed is fixed so same numbers are generated):


```r
library(lme4)
library(lmerTest)
library(ggplot2)
set.seed(1850)  # This is simply fixing the seed so the code always gives the same result.  
#Obviously omit this if you are running new simulations

# Simulate true subject-specific means:
nsub = 10
btwn.sub.sd = 10
#within-subject means
win.means = rnorm(nsub, mean = 250, sd = btwn.sub.sd)

# Simualte data for each subject by wiggling around their means
win.sub.sd = 20
# The following indicates how many data per subject, the first 5 only have 5 observations.
n.per.sub = rep(50, nsub)
n.per.sub[1:5] = 5
rt = c()
subid = c()
for (i in 1:nsub){
  rt.loop = rnorm(n.per.sub[i], win.means[i], sd = win.sub.sd)
  rt = c(rt, rt.loop)
  subid = c(subid, rep(i, n.per.sub[i]))
}
dat = data.frame(subid, rt)
```


## The equation for shrinkage

As a reminder, the formula for the mixed model is: $$Y_i = X_i\beta + A_ib_i + \epsilon_i. $$ With the data setup for this chapter,  $Y_i$ is a vector of reaction times for subject $i$, $X_i = Z_iA_i$ simplifies to $Z_i$, since we are only estimating an average over subjects.  Since we are also only estimating a within-subject average,  $Z_i$ is simply a column of 1s with length $N_i$, the number of observations for subject $i$. The random effect, $b_i$, is of length 1 since only a random intercept in necessary.  Specifically, $b_i\sim N(0, G)$, where $G$ is a scalar, and $\epsilon_i$ describes the within-subject variability, $\epsilon_i \sim N(0, \sigma^2I_{N_i})$. If needed, review the more detailed explanation of these matrices and vectors from previous lessons.


The following are what we need to focus on and I quickly relate the parameters to the output of the code below, which is repeated code from the last lesson.

* $\hat\beta$:  The fixed effects (group) estimate of the mean from the mixed model.  This is the estimated group intercept, 253.88 below.
* $\hat\beta_i$: The conditional mode, predicted value for subject $i$ from the mixed model.  These values are stored in the mmcm vector created with the coef function below.
* $\hat\beta_i^{OLS}$:  The OLS-based estimate from stage 1 of 2SSS, $Y_i = Z_i\beta_i^{OLS} + \gamma_i.$  These are stored in the stage1.est vector below.
* $N_i$: the number of data points for subject $i$.  For now this is 5 for the first 5 subjects and 50 for the second 5 subjects.
* $G$ and $\sigma^2$:  Between-subject variance/covariance matrix and within-subject variance.  In this case we'll work with the estimates which are $G=6.309^2$ and $\sigma^2 = 20.577^2$ and stored in vcov.vals.  Find them in the mixed model output below.



```r
# Stage 1
stage1.est = rep(NA, nsub)
for (i in 1:nsub){
  # Estimating the mean RT via linear regression
  mod.loop = lm(rt ~ 1, dat[dat$subid==i,])
  stage1.est[i] = mod.loop$coef[1]
}
mod.lmer = lmer(rt ~ 1 + (1|subid), dat)
summary(mod.lmer)
```

```
## Linear mixed model fit by REML. t-tests use Satterthwaite's method [
## lmerModLmerTest]
## Formula: rt ~ 1 + (1 | subid)
##    Data: dat
## 
## REML criterion at convergence: 2449.6
## 
## Scaled residuals: 
##      Min       1Q   Median       3Q      Max 
## -2.54753 -0.69671  0.02651  0.72996  2.80661 
## 
## Random effects:
##  Groups   Name        Variance Std.Dev.
##  subid    (Intercept)  39.81    6.309  
##  Residual             423.42   20.577  
## Number of obs: 275, groups:  subid, 10
## 
## Fixed effects:
##             Estimate Std. Error      df t value Pr(>|t|)    
## (Intercept)  253.885      2.638   6.206   96.25 4.35e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
mmcm = coef(mod.lmer)$subid[, 1]
vcov.vals = as.data.frame(VarCorr(mod.lmer)) #random effects values
beta.hat = summary(mod.lmer)$coefficients[1]
```


  If one is interested in the derivation of the relationship between $\hat\beta_i$ and $\hat\beta_i^{OLS}$, see section 8.7 of the [Applied Longitudinal Analysis text by Fitzmaurice and others](https://www.amazon.com/Applied-Longitudinal-Analysis-Garrett-Fitzmaurice/dp/0470380276).  The end result is: $$\hat\beta_i = W_i\hat\beta_i^{OLS}+(I_q-W_i)A_i\hat\beta,$$
where $q$ is the dimension of G and $$W_i= G\{G+\sigma^2(Z_i'Z_i)^{-1}\}^{-1}.$$
Before tackling what $W_i$ is, exactly, focus on the general structure of the relationship assuming $A_i$ is the identity matrix (more on this in a future lesson).   If the weight, $W_i$, is very small, then what counts the most toward the estimate of $\hat\beta_i$ is the fixed effects estimate of $\beta$.  From the last lesson you might suspect that $W_i$ will be small when a subject has less data, since that's when more regularization was present.  On the other hand, if $W_i$ is very large, the conditional mode-based estimate from the mixed model will look very similar to the within-subject estimate from the first stage of 2SSS, which is what we saw for the subjects who had more data in the last lesson.  

Let's test that theory by deriving $W_i$ for our specific model.  Recall $Z_i$ is just a column of 1s of length $N_i$, so $(Z_i'Z_i)^{-1} = 1/N_i$ and $G$ is just a scalar (between-subject variance of the mean), so we have: $$W_i  = \frac{G}{G+\sigma^2/N_i}=\frac{6.31^2}{6.31^2+20.58^2/N_i}.$$

Now it is crystal clear that our suspicions from the last lesson are true: for small $N_i$ the overall weight will be small, favoring $\hat\beta$ and if $N_i$ is large, the overall weight will be large, favoring $\hat\beta^{OLS}_i$.  Here's a plot of the weight as a function of $N_i$


```r
Ni.plot = 1:50
Wi.plot = vcov.vals[1,4]/(vcov.vals[1,4] + vcov.vals[2,4]/Ni.plot)
datplot = data.frame(Ni.plot, Wi.plot)
ggplot(datplot, aes(x=Ni.plot, y =Wi.plot)) + geom_point() +labs(y=expression(W[i]), x = expression(N[i]))
```



\begin{center}\includegraphics{3_video5_win_sub_mns_vs_cond_modes_files/figure-latex/unnamed-chunk-4-1} \end{center}

Granted things will be more difficult if the model has a slope and an intercept (when G is a matrix) and when between-subject variables are included ($A_i$ will not be the identity), but we will deal with that later.

## Verifying coef.lmer is following the equation

Although it is satisfying to find this equation it is only fully satisfying if that is what lmer is doing when we ask for the conditional modes.  The following verifies this for each subject. 


```r
Wi = vcov.vals[1,4]/(vcov.vals[1,4] + vcov.vals[2,4]/n.per.sub) 
# See first code chunk for n.per.sub
cond.mode.est = Wi*stage1.est + (1-Wi)*beta.hat

# Stack my estimates on top of lmer's conditional model estimates
rbind(cond.mode.est, mmcm)                     
```

```
##                   [,1]     [,2]     [,3]    [,4]     [,5]     [,6]     [,7]
## cond.mode.est 256.9017 251.9737 247.7054 251.652 250.0628 260.2288 248.4464
## mmcm          256.9017 251.9737 247.7054 251.652 250.0628 260.2288 248.4464
##                   [,8]     [,9]    [,10]
## cond.mode.est 256.1146 261.2941 254.4678
## mmcm          256.1146 261.2941 254.4678
```

Satisfying, isn't it?  

What do you think will happen if the within-subject variance is smaller, which is more typical?  Will the weights vary as much?  No need to re-simulate data, just plug in values for $G$ and $\sigma^2$ and plot the weights as shown below.


```r
# Large within-subject variance with respect to between-subject variance (go bold or go home)
Ni.plot = seq(1, 50, 5)

Wi.plot.large.within = 1/(1 + 100/Ni.plot)

# Large between-subject variance with respect to within-subject variance
Wi.plot.large.between = 100/(100 + 1/Ni.plot)

Wi.all = c(Wi.plot.large.within, Wi.plot.large.between)
Setting = rep(c("Large within-subject", "Large between-subject"), each = length(Ni.plot))
Ni.all = c(Ni.plot, Ni.plot)
datplot2 = data.frame (Wi.all, Ni.all, Setting)
cbPalette <- c("#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7","#999999")

ggplot(datplot2, aes(x=Ni.all, y =Wi.all, color = Setting)) + geom_point() +labs(y=expression(W[i]), x = expression(N[i]))+ scale_colour_manual(values=cbPalette)
```



\begin{center}\includegraphics{3_video5_win_sub_mns_vs_cond_modes_files/figure-latex/unnamed-chunk-6-1} \end{center}


## Summary 

The settings in the above were pretty dramatic, but do show the 2SSS approach will likely be very similar to the mixed model result if the between-subject variance is large with respect to the size of the within-subject variance.  Further, the weights will vary most across subjects if the sample sizes vary wildly.  I would like to highly encourage the reader to run the above code with more settings to see how the weights change.

In the next chapter we will study the group results in more detail, but where the goal isn't a single group mean, but the difference in means between two groups.  Does the 2SSS still work okay is most situations?  What if one group has generally less data per subject than the other group?  Can that cause false positives or reduced power in the mixed model result?  What if we use the conditional modes in an analysis?  Does that work okay?  All of these questions will be addressed in the next chapter.

<!--chapter:end:3_video5_win_sub_mns_vs_cond_modes.Rmd-->

# V6: Model strategy comparison - group differences

## Group differences of within-subject averages: How do modeling approaches compare with unequal groups and unequal observations within-subject? {.unlisted}


## Introduction
This chapter accompanies the 6th video in the series, ["Can the regularization in mixed models be problematic?"](https://youtu.be/C4r20a_pqlA).  Previous chapters focused on the regularization built into a mixed model. As I mentioned earlier, when I explained this to a colleague of mine, she asked how the regularization would impact her results.  She's studying a patient group that tends to be much smaller than her control group and often she also has less data per subject in her patient group.  Will the regularization of the mixed model hurt her power to detect group differences?  In this chapter I focus on different approaches to modeling the data (mixed models, 2SSS and two pseudo 2SSS approaches) to investigate type I error and power when data are both balanced and unbalanced within-group and within-subject.  Additionally, these simulations will address my previous comments about why the conditional modes are not the perfect way of viewing regularization.

### Modeling strategies considered

There are four modeling approaches, total.  First, the standard mixed model, with the group variable as a fixed effect and a random intercept.  Second, the 2SSS using OLS at both stages.  The last two models are similar to 2SSS, but for the first stage use conditional mode-based subject estimates from each of two different mixed models:  one that models group as a fixed effect and one that does not.  Although it is easy to argue that neither of these pseudo 2SSS models makes any sense, I can assure you that I have definitely seen this done in practice and therefore wanted to discuss it here.  As we will see, neither of these approaches is a good idea.  Surely, the second pseudo 2SSS model is most nonsensical since the model from which the conditional modes are extracted actually contains the inference of interest (group difference). Why do that extra work?  

First think through how each of these approaches might behave.  The 2SSS approach will not deal with the differing number of time points, so I would expect to see some issues there due to the variance differences between groups.  For the mixed model there shouldn't be an issue.  Sure, there could be some regularization, but since each group's mean is modeled, the bias would be toward the group means and not the overall mean.  The conditional modes from the model with an intercept only will definitely yield some bias that could hide or emphasize the difference between groups. Also, the extra variability introduced by using these predicted values will likely lead to an underestimate of the variance.  The 2SSS with conditional modes from the two group mixed model should have less bias, but the variance estimates will still be incorrect.  

A very important consideration when you are interested in power, is you must first verify the type I error is controlled.  That is how these simulations are set up.  For the type I error investigation, the group means are set equal to each other.  Then power is investigated in all scenarios, but it is very important to ignore the power estimates for the models that were not found to have controlled type I error. For that reason I have only put "ghosts" of the power estimates for those models in the plots.  Bias is also important.  Obviously the mixed model's conditional modes will be biased on purpose, but we don't want our final estimates to also be biased, because that can be misleading. You should suspect the pseudo 2SSS using conditional modes from the mixed model without modeling group will have a biased group difference estimate.   

In the following the function for data simulation is constructed first, followed by a function that runs the analyses on numerous data sets so type I error and power can be calculated.  After that I run 4 simulations for different study designs investigating type I error.  Last power is estimated across the four designs.  You are highly encouraged to try out your own designs.  I have made the functions fairly flexible so you could do so.  Do keep in mind that quite a few simulations are required and so it will take some time for them to complete.

## Functions for data simulation and analyses

This first function simulates data for two groups. The code should seem very familiar, since  it is based on the previous chapters that simulated data using the 2 stage approach.  The one difference is there is a group variable here, while the previous simulations were for a single group. 


```r
library(lme4)
library(lmerTest)
library(ggplot2)
library(knitr)
library(tidyverse)
library(ggpubr)

make.data.grps = function(win, btwn, n.win.sub, group.vec, grp1.mean, grp2.mean){
  #win:  A vector of the within-subject sd of length nsub
  #btwn: Scalar, the between-subject sd
  # group.vec: 1's and 0's indicating group membership.  Length nsub.
  # grp1.mean:  Mean for group 1 (group.vec=0)
  # grp2.mean:  Mean for group 2 (group.vec=1)
  
  nsub = length(group.vec)
  # Simulate means for each subject
  mns.sub = grp1.mean*group.vec + (1-group.vec)*grp2.mean +  rnorm(nsub, mean = 0, sd = btwn)
  # Simulate data within-subject
  y =c()
  id = c()
  grp.all = c()
  for (i in 1:nsub){
    y.loop = rnorm(n.win.sub[i], 
                   mean = mns.sub[i], sd = win[i])
    y = c(y, y.loop)
    id = c(id, rep(i, n.win.sub[i]))
   grp.all = c(grp.all, rep(group.vec[i], n.win.sub[i]))
  }
  id = as.factor(id)
  dat.mod = data.frame(id, y, grp.all)
  return(dat.mod)
}
```

This function is created to run the simulation and organize the output.  We will investigate the mean estimates for each group, the standard error of the difference in means according to each model, and the p-values, which will be used to either calculate power or type I error.


```r
run.sim = function(nsim, win, n.win.sub, btwn,  group.vec, grp1.mean, grp2.mean, rand.ord.nsub){
  #nsim:  Number of simulations
  #win: within-subject variances.  Vector of length nsub
  #n.win.sub:  Number of measures within-subject.  Vector of length nsub
  #btwn:  between-subject variance
  #group.vec:  1/0 vector indicating groups
  #grp1.mean, grp2.mean:  Group means
  #rand.ord.nsub:  Randomly order subjects (mixes up n.win.sub so random subjects have smaller n)
 
  g1.mean.lmer = rep(NA, nsim)
  g2.mean.lmer = rep(NA, nsim)
  g1.mean.2sss = rep(NA, nsim)
  g2.mean.2sss = rep(NA, nsim)
  g1.mean.2sss.cond.mode.2grp = rep(NA, nsim)
  g2.mean.2sss.cond.mode.2grp = rep(NA, nsim)
  g1.mean.2sss.cond.mode.nogrp = rep(NA, nsim)
  g2.mean.2sss.cond.mode.nogrp = rep(NA, nsim)
  
  diff.se.lmer = rep(NA, nsim)
  diff.se.2sss = rep(NA, nsim)
  diff.se.2sss.cond.mode.2grp = rep(NA, nsim)
  diff.se.2sss.cond.mode.nogrp = rep(NA, nsim)
  
  p.lmer = rep(NA, nsim)
  p.2sss = rep(NA, nsim)
  p.2sss.cond.mode.2grp = rep(NA, nsim)
  p.2sss.cond.mode.nogrp = rep(NA, nsim)
  
  for (i in 1:nsim) {
     # This is used when I want to randomly 
     #  select who has less data
    nsub = length(group.vec)
    if (rand.ord.nsub == 'yes'){
      n.win.sub = sample(n.win.sub)
    }
  
    # Generate the data
    dat.mod = make.data.grps(win, btwn, n.win.sub,
                             group.vec, grp1.mean,
                             grp2.mean)
  
    # Run the 4 models
    # mixed model
    mod.lmer = lmer(y ~ 1 + grp.all + (1 | id), dat.mod)

    # 2SSS OLS
    # trick for getting OLS means for each subject
    ols.stage1 = lm(y ~ id -1, dat.mod)$coeff
    twosss = lm(ols.stage1 ~ group.vec)
    
    # 2SSS using cond modes from model including group
    # pull out conditional modes
    cond.modes.2grp.mod = coef(mod.lmer)$id[, 1]+
                          coef(mod.lmer)$id[, 2]*group.vec
    twosss.cond.mode.2grp = lm(cond.modes.2grp.mod ~
                                 group.vec)
    #2SSS using cond modes from model without group  
    mod.lmer.nogroup = lmer(y ~ 1 + (1 | id), dat.mod)
    cond.modes.nogroup.mod = coef(mod.lmer.nogroup)$id[, 1]
    twosss.cond.mode.nogrp = lm(cond.modes.nogroup.mod ~
                                  group.vec)
    
    g1.mean.lmer[i] = sum(fixef(mod.lmer))
    g2.mean.lmer[i] = fixef(mod.lmer)[1]
    g1.mean.2sss[i] = sum(coef(twosss))
    g2.mean.2sss[i] = coef(twosss)[1]
    g1.mean.2sss.cond.mode.2grp[i] = sum(coef(twosss.cond.mode.2grp))
    g2.mean.2sss.cond.mode.2grp[i] = coef(twosss.cond.mode.2grp)[1]
    g1.mean.2sss.cond.mode.nogrp[i] = sum(coef(twosss.cond.mode.nogrp))
    g2.mean.2sss.cond.mode.nogrp[i] = coef(twosss.cond.mode.nogrp)[1]
  
    diff.se.lmer[i] = summary(mod.lmer)$coeff[2,2]
    diff.se.2sss[i] = summary(twosss)$coeff[2,2]
    diff.se.2sss.cond.mode.2grp[i] = summary(twosss.cond.mode.2grp)$coeff[2,2]
    diff.se.2sss.cond.mode.nogrp[i] = summary(twosss.cond.mode.nogrp)$coeff[2,2]
  
    p.lmer[i] = summary(mod.lmer)$coeff[2,5]
    p.2sss[i] = summary(twosss)$coeff[2,4]
    p.2sss.cond.mode.2grp[i] = summary(twosss.cond.mode.2grp)$coeff[2,4]
    p.2sss.cond.mode.nogrp[i] = summary(twosss.cond.mode.nogrp)$coeff[2,4]
  }
  
  # put it all into a data frame
  out = data.frame(g1.mean.lmer, g2.mean.lmer, 
                   g1.mean.2sss, g2.mean.2sss,
                   g1.mean.2sss.cond.mode.2grp,   
                   g2.mean.2sss.cond.mode.2grp,
                   g1.mean.2sss.cond.mode.nogrp,
                   g2.mean.2sss.cond.mode.nogrp,
                   diff.se.lmer, diff.se.2sss, diff.se.2sss.cond.mode.2grp, 
                   diff.se.2sss.cond.mode.nogrp, p.lmer,
                   p.2sss, p.2sss.cond.mode.2grp,
                   p.2sss.cond.mode.nogrp)
  return(out)
}
```


## Type I error

This first batch of simulations will calculate the type I error.  In order to do so, the means for the groups are set to be equal (10). I would recommend at least 1000 simulations if you were to run this on your own.  The code allows you to change the within-subject variance across subjects, but all model estimation methods assume a constant within-subject variance, so there wasn't really a reason to vary that here.  Feel free to try it out on your own!  I start of with balanced groups where either everybody has 30 observations or one group has 5 subjects with only 5 observations.  Then, to mimic the real life data my colleague has where her patient group is much smaller and often have less data, I set the group sizes to 10 and 40 where they either all have 30 observations or half the patients only have 5 observations.

I realize the settings are a bit extreme, BUT if I can break a model in the extreme I will never ever use it on my real data.  Why take the risk when I don't know how "extreme" my real data are.  If I can't break a model in the extreme, then I know I can probably almost always trust that model.

By the way, if you've been paying close attention, take a look at the simulation settings and think back to the previous chapter.  You will see there are differences in model performance here, but what would likely make them more similar?  Hint:  You'd need to change one number in the simulations I ran below.  Again, aiming for the extreme to see what breaks!


```r
# 25 subjects each group, everybody has 30 observations
nsim = 1000
win = rep(10, 50)
n.win.sub = rep(30, 50)
btwn = 5
group.vec = rep(c(1, 0), each = 25)
# Set the means equal, since we're looking at Type I error
grp1.mean = 10
grp2.mean = 10
rand.ord.nsub = 'no'

out.25.25.nolow.type1 = run.sim(nsim, win, n.win.sub, btwn,
                          group.vec, grp1.mean, 
                          grp2.mean, rand.ord.nsub)

# Change so 5 in first group have low n
n.win.sub[1:5] = 5
out.25.25.5g1.type1 = run.sim(nsim, win, n.win.sub, btwn,
                        group.vec, grp1.mean, 
                        grp2.mean, rand.ord.nsub)

#imblanaced groups with equal n
n.win.sub = rep(30, 50)
group.vec = c(rep(1, 10), rep(0, 40))
out.10.40.nolow.type1 = run.sim(nsim, win, n.win.sub, btwn,
                          group.vec, grp1.mean, grp2.mean,
                          rand.ord.nsub)

# imbalanced groups with some in small group with low n
n.win.sub[1:5] = 5
out.10.40.5g1.type1 = run.sim(nsim, win, n.win.sub, btwn, 
                        group.vec, grp1.mean, grp2.mean,
                        rand.ord.nsub)
```

The following rearranges the data for plotting.  First, the mean estimates (within each group and difference of means) are presented for each combination of data setup and analysis type.  The boxplots are constructed from the estimates across the simulations.  Look for bias in these estimates.  Since the truth is known, I have overlayed lines to indicate the true value. Also, one would typically look to see if the estimates varied differently across the models.  They do not in this case.  There is a discussion about the plots after they are displayed.


```r
# arrange for plotting

cbPalette <- c("#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7","#999999")

 dat.all.type1 = rbind(out.25.25.nolow.type1,
                       out.25.25.5g1.type1,
                       out.10.40.nolow.type1,
                       out.10.40.5g1.type1)
 
 dat.all.type1$sim.type = factor(rep(c("25.25.nolow",
                                       "25.25.5g1",
                                       "10.40.nolow", 
                                       "10.40.5g1"),
                                 each =nsim), levels =
                                 c("25.25.nolow", "25.25.5g1",
                                 "10.40.nolow", "10.40.5g1"))
 dat.all.type1$n1 = rep(c(25, 25, 10, 10), each = nsim)
 dat.all.type1$n2 = rep(c(25, 25, 40, 40), each = nsim)
 dat.all.type1$lown = rep(c("none", "5 in g1", "none", 
                            "5 in g1"),
                          each = nsim)
 
 col.g1.mean = grep('g1.mean', names(dat.all.type1), 
                    fixed = T)
 dat.all.g1.mean.type1 = dat.all.type1 %>%
                   gather(key="key", 
                          value="group1mean", 
                          col.g1.mean) %>%
                   separate(key, 
                            c("junk1", "model"), 
                            sep = "g1.mean.")
 dat.all.g1.mean.type1$model = factor(dat.all.g1.mean.type1$model, 
                                levels = 
                                  c("lmer", "2sss",
                                   "2sss.cond.mode.nogrp",
                                    "2sss.cond.mode.2grp"))
 
g1.plot.type1 = ggplot(dat.all.g1.mean.type1, aes(x = sim.type,
                                      y = group1mean, 
                                      fill = model)) +
                geom_boxplot() + 
                geom_hline(yintercept = 10, color = "gray26") +
                scale_fill_manual(values=cbPalette)+
                ggtitle("Group 1 mean")

# Repeat for g2
col.g2.mean.type1 = grep('g2.mean', names(dat.all.type1), fixed = T)
dat.all.g2.mean.type1 = dat.all.type1 %>%
                   gather(key="key", value="group2mean",
                          col.g2.mean.type1) %>%
                   separate(key, c("junk1", "model"), 
                            sep = "g2.mean.")
dat.all.g2.mean.type1$model = factor(dat.all.g2.mean.type1$model, 
                                levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp")) 
g2.plot.type1 = ggplot(dat.all.g2.mean.type1, 
                       aes(x = sim.type, y = group2mean, 
                           fill = model)) + geom_boxplot() +
                geom_hline(yintercept = 10, color = "gray26")+
                scale_fill_manual(values=cbPalette)+
                ggtitle("Group 2 mean")
 
# Estimate the differences and then plot the differences
dat.all.diff.type1 = left_join(dat.all.g1.mean.type1, dat.all.g2.mean.type1)
dat.all.diff.type1$diff = dat.all.diff.type1$group2mean -
                    dat.all.diff.type1$group1mean
dat.all.diff.type1$model = factor(dat.all.diff.type1$model, 
                                levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp")) 
diff.plot.type1 = ggplot(dat.all.diff.type1, aes(x = sim.type, 
                                                 y = diff, 
                                                 fill = model)) +
                   geom_boxplot() + 
                   geom_hline(yintercept = 0, color = "gray26")+
                   scale_fill_manual(values=cbPalette)+
                   ggtitle("Difference")

# plot all 3 together
ggarrange(g1.plot.type1, g2.plot.type1, diff.plot.type1, common.legend = TRUE,
          nrow = 3, legend = "right")
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-4-1.pdf)<!-- --> 

In the above plots, there is no bias present.  This doesn't mean all methods are generally unbiased, it simply means when both group means are 0 there isn't bias.  Think about how the regularization works in the mixed model and ask yourself whether it makes sense that there isn't bias here, even when some subjects have much less data.  Do you predict there will be bias when the group means are different from each other?

Next the standard error of the group differences will be displayed.  Generally we want our standard errors to be small BUT that's only helpful if our resulting inferences are valid (type I error is controlled).  The setup is similar to the last plots.


```r
# Look at the standard errors
col.se.type1 = grep('diff.se', names(dat.all.type1), fixed = T)
dat.all.se.type1 = dat.all.type1 %>%
                   gather(key="key", value="se", col.se.type1) %>%
                   separate(key, c("junk1", "model"), 
                            sep = "diff.se.")
dat.all.se.type1$model = factor(dat.all.se.type1$model, 
                          levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp"))
 
ggplot(dat.all.se.type1, aes(x = sim.type, y = se, fill = model)) +
       geom_boxplot() + scale_fill_manual(values=cbPalette)
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-5-1.pdf)<!-- --> 
 
It is tempting to get excited about the pseudo 2SSS approaches that use conditional modes based on these standard error distributions BUT remember, we still haven't looked at type I error and fully investigated bias. 

Next up is the type I error investigation, which clearly ends any hope one may have had for the 2SSS model using the conditional mode from the mixed model that had group included as a fixed effect.  It doesn't matter in real life, though, since if one already ran that model they wouldn't have likely continued to extract conditional modes, etc, because the inference for the group difference was already included in the mixed model and, as the plot shows, the type I errors are preserved in those cases.

There are horizontal lines at 0.05 (solid) and the 95% confidence interval (dashed).  We will use type I error beyond the upper confidence bound as the threshold for an invalid test, which will be used to help interpret the power results later.

 

```r
 # Calculate type I error
 col.p = grep('p.', names(dat.all.type1), fixed = T)
 dat.all.p.type1 = dat.all.type1 %>%
              gather(key="key", value="pval", col.p) %>%
              separate(key, c("junk1", "model"), sep = "p.")
dat.all.p.type1$p.sig = dat.all.p.type1$pval <= 0.05
type1.mat = aggregate(p.sig ~ sim.type + model,
                    data = dat.all.p.type1, mean)
type1.mat$model = factor(type1.mat$model, 
                       levels = c("lmer", "2sss",
                                  "2sss.cond.mode.nogrp",
                                  "2sss.cond.mode.2grp"))

#bound for type I error (upper bound 95% CI)
bound.up = .05+qnorm(1-0.05/2)*sqrt(0.05*(1-0.05)/nsim)
bound.low = .05-qnorm(1-0.05/2)*sqrt(0.05*(1-0.05)/nsim)


ggplot(type1.mat, aes(x=sim.type, y = p.sig, fill = model)) + geom_bar(stat = "identity", position=position_dodge())+ scale_fill_manual(values=cbPalette)+ 
  geom_hline(yintercept = bound.up,color = "gray26", linetype = "dashed")+ 
  geom_hline(yintercept = bound.low,color = "gray26", linetype = "dashed")+ 
  geom_hline(yintercept = 0.05,color = "gray26")
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-6-1.pdf)<!-- --> 

```r
# Add column that indicates when p is beyond upper bound
type1.mat$valid = type1.mat$p.sig<bound.up
```

So far the 2SSS using conditional modes from the mixed model including group is a clear loser.  Also, we can see that the OLS-based 2SSS fails when the smaller group has subjects with missing data.  This actually makes complete sense since this is exactly what Welch's t-test is for!  Specifically heteroscedasticity, which in this case is driven by the differing within-subject sample sizes.  Standard OLS won't do anything for differing variances, but the mixed model does incorporate this information when the variance differences is driven by different sample sizes.

Last, the 2SSS approach where the conditional mode from the model without group looks somewhat hopeful here, but stay tuned.  There's no avenue for bias to creep in due to the regularization here, since the means for both groups are set to zero!  


## Power

The same setup is used below, but the mean for the first group has been increased to 13, so the true difference between groups is 3.  I haven't calculated what the true power should be in this case, so power will just be compared between different models within the same data type, but ignoring any power results for which the type I error was not controlled.


```r
# 25 subjects each group, everybody has 30 observations
win = rep(10, 50)
n.win.sub = rep(30, 50)
btwn = 5
group.vec = rep(c(1, 0), each = 25)
grp1.mean = 13
grp2.mean = 10
rand.ord.nsub = 'no'

out.25.25.nolow = run.sim(nsim, win, n.win.sub, btwn,
                          group.vec, grp1.mean, 
                          grp2.mean, rand.ord.nsub)

# Change so 5 in first group have low n
n.win.sub[1:5] = 5
out.25.25.5g1 = run.sim(nsim, win, n.win.sub, btwn,
                        group.vec, grp1.mean, 
                        grp2.mean, rand.ord.nsub)

#imblanaced groups with equal n
n.win.sub = rep(30, 50)
group.vec = c(rep(1, 10), rep(0, 40))
out.10.40.nolow = run.sim(nsim, win, n.win.sub, btwn,
                          group.vec, grp1.mean, grp2.mean,
                          rand.ord.nsub)

# imbalanced groups with some in small group with low n
n.win.sub[1:5] = 5
out.10.40.5g1 = run.sim(nsim, win, n.win.sub, btwn, 
                        group.vec, grp1.mean, grp2.mean,
                        rand.ord.nsub)
```

Starting again with plots of the within-group means and mean difference.  You can definitely see some bias showing.  Try and understand where the bias is coming from and why it is in the direction it is.  Even though all results are present here, recall we've written off the 2SSS using conditional modes from the mixed model that included group.  Also, the 2SSS for imbalanced groups where one group has less data.


```r
# arrange for plotting

cbPalette <- c("#E69F00", "#56B4E9", "#009E73", "#F0E442", "#0072B2", "#D55E00", "#CC79A7","#999999")

 dat.all = rbind(out.25.25.nolow, out.25.25.5g1,
                 out.10.40.nolow, out.10.40.5g1)
 
 dat.all$sim.type = factor(rep(c("25.25.nolow", "25.25.5g1",
                                 "10.40.nolow", "10.40.5g1"),
                               each =nsim), levels =
                             c("25.25.nolow", "25.25.5g1",
                               "10.40.nolow", "10.40.5g1"))
 dat.all$n1 = rep(c(25, 25, 10, 10), each = nsim)
 dat.all$n2 = rep(c(25, 25, 40, 40), each = nsim)
 dat.all$lown = rep(c("none", "5 in g1", "none", "5 in g1"),
                    each = nsim)
 
 
 col.g1.mean = grep('g1.mean', names(dat.all), fixed = T)
 dat.all.g1.mean = dat.all %>%
                   gather(key="key", 
                          value="group1mean", 
                          col.g1.mean) %>%
                   separate(key, 
                            c("junk1", "model"), 
                            sep = "g1.mean.")
 dat.all.g1.mean$model = factor(dat.all.g1.mean$model, 
                                levels = 
                                  c("lmer", "2sss",
                                   "2sss.cond.mode.nogrp",
                                    "2sss.cond.mode.2grp"))
 
g1.plot = ggplot(dat.all.g1.mean, aes(x = sim.type,
                                      y = group1mean, 
                                      fill = model)) +
          geom_boxplot() + 
          geom_hline(yintercept = 13, 
                     color = "gray26") +
          scale_fill_manual(values=cbPalette)+
          ggtitle("Group 1 mean")

# Repeat for g2
col.g2.mean = grep('g2.mean', names(dat.all), fixed = T)
dat.all.g2.mean = dat.all %>%
                   gather(key="key", value="group2mean",
                          col.g2.mean) %>%
                   separate(key, c("junk1", "model"), 
                            sep = "g2.mean.")
dat.all.g2.mean$model = factor(dat.all.g2.mean$model, 
                                levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp")) 
g2.plot = ggplot(dat.all.g2.mean, aes(x = sim.type, y = group2mean, fill = model)) + 
        geom_boxplot() + 
        geom_hline(yintercept = 10, color = "gray26") +      
        scale_fill_manual(values=cbPalette)+ggtitle("Group 2 mean")
 
# Estimate the differences and then plot the differences
dat.all.diff = left_join(dat.all.g1.mean, dat.all.g2.mean)
dat.all.diff$diff = dat.all.diff$group2mean -
                    dat.all.diff$group1mean
dat.all.diff$model = factor(dat.all.diff$model, 
                                levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp")) 
diff.plot = ggplot(dat.all.diff, aes(x = sim.type, y = diff,
                                     fill = model)) +
            geom_boxplot() + geom_hline(yintercept = -3,
                                        color = "gray26")+
            scale_fill_manual(values=cbPalette)+
            ggtitle("Difference")

# plot all 3 together
ggarrange(g1.plot, g2.plot, diff.plot, common.legend = TRUE,
          nrow = 3, legend = "right")
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-8-1.pdf)<!-- --> 

There is always some bias on the 2SSS approach that used conditional modes from the mixed model without a fixed effect for group.  It gets worse when group 1 is small and some of those subjects only had 5 observations, because that's when the regularization will be strongest and the regularization is pulling the estimates toward the overall mean, not the mean of group 1.  The bias is not present in the 2SSS model that uses the conditional modes from the mixed model with group because the bias in that case will be toward the group mean instead of the overall mean.

An important point to make here is the bias is in the direction of the null.  So, although the standard errors are too small, this is overridden by the bias causing the difference estimate to also be too small.

Moving on to the standard errors, the results are similar to before.


```r
# Look at the standard errors
col.se = grep('diff.se', names(dat.all), fixed = T)
dat.all.se = dat.all %>%
                   gather(key="key", value="se", col.se) %>%
                   separate(key, c("junk1", "model"), 
                            sep = "diff.se.")
 dat.all.se$model = factor(dat.all.se$model, 
                          levels = c("lmer", "2sss",
                                      "2sss.cond.mode.nogrp",
                                      "2sss.cond.mode.2grp"))
 
 ggplot(dat.all.se, aes(x = sim.type, y = se, fill = model)) + geom_boxplot() + scale_fill_manual(values=cbPalette)
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-9-1.pdf)<!-- --> 

Last, but not least, power.  I have faded out the bars of the power for the methods that were found to not be valid when type I error was estimated above.  


```r
 # Calculate power
 col.p = grep('p.', names(dat.all), fixed = T)
 dat.all.p = dat.all %>%
              gather(key="key", value="pval", col.p) %>%
              separate(key, c("junk1", "model"), sep = "p.")
 dat.all.p$p.sig = dat.all.p$pval <= 0.05
pow.mat = aggregate(p.sig ~ sim.type + model,
                    data = dat.all.p, mean)
pow.mat$model = factor(pow.mat$model, 
                       levels = c("lmer", "2sss",
                                  "2sss.cond.mode.nogrp",
                                  "2sss.cond.mode.2grp"))
# Add in valid test info from type 1 error
valid.info = type1.mat[,c("sim.type", "model", "valid")]
pow.mat = full_join(pow.mat, valid.info)

ggplot(pow.mat, aes(x=sim.type, y = p.sig, fill = model, alpha = valid)) + geom_bar(stat = "identity", position=position_dodge())+ scale_fill_manual(values=cbPalette)+scale_alpha_discrete(range = c(0.2, 1))
```

![](4_video6_group_comparison_three_models_files/figure-latex/unnamed-chunk-10-1.pdf)<!-- --> 

Firstly, the star of the show, not surprisingly, is lmer.  Not only was type I error always, preserved, but it either ties for highest power or has the highest power.  Also, remember it was only 1 simple line of code.  The 2SSS almost looks like the winner when the patient group is smaller and some patients have fewer data points, but that test had inflated type I error, so the power cannot be considered.  As far as the 2SSS approach using conditional modes from the mixed model without group, it seems like maybe this one is okay but remember that bias! Biased estimates aren't going to be of much use, so that model cannot be recommended.

## Summary

Overall, when comparing group means the mixed model is the safest bet. The clear loser is the pseudo 2SSS using conditional modes from the model with group as a fixed effect. The OLS-based 2SSS approach might be okay in a pinch if your groups are balanced and data within-subject are balanced, but I will remind you the mixed model was a single line of code.  The psuedo 2SSS approach with conditional modes extracted from the mixed model without an intercept may yield biased results, due to the regularization, so it should also be avoided.

I would like to circle back to when I used the conditional modes to illustrate the regularization earlier.  I kept mentioning that they were not perfect and these simulations show why.  The estimates from the 2SSS approach using conditional modes that modeled group were extracted from the exact same model used for the lmer results, yet look how different the performance is!  I know it is often the case that these values are plotted in manuscripts to help understand what the individual subjects were doing BUT it is very important when you make plots that they are representative of what the model was actually doing.  In this case it can be misleading so I would recommend one be careful if using such plots and make sure the reader understands that the values are simply predictions with high variability and do not perfectly represent how individual subjects behaved in the model.  Again, this is because the mixed model isn't actually using those estimates and those estimates are very noisy predictions.  So, they were nice for visualizing how the regularization works, but they are not the same as what is going on inside of the model!

If you haven't already figured it out, I have set the between-subject variance to be quite small here.  If increased, the results will be more similar across modeling approaches.  Since we don't know what our true within- and between-subject variances are, I would never assume the between subject was large enough that I didn't need to worry when it is often the case that I can simply use the appropriate model.



<!--chapter:end:4_video6_group_comparison_three_models.Rmd-->

