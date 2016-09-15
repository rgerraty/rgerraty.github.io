---
layout: post
category : [probability and statistics]
excerpt: "Estimating variance of interest in a multilevel model, part I"
title: "What is this thing called random effects?: a conversation with Niall Bolger"
comments: true
---

[Bruce Dore][link1] and I have been examining different ways of estimating uncertainty about variances, using an example of personality questionaires. How do you model differences in personality scores across universities? One of the interesting (to us at least!) questions that comes up in cases like this is how and when to pool your uncertainty at different levels of the model. A core part of multilevel modeling is to let my knowlege of the distribution of subject effects, for example, influence my estimates of each individual subject. You could write that out as follows, in the simple case of taking a mean:

The data for subject \\i\\ is:

$$y_{i} \sim N(\beta_{i},\tau_{i}^{2})$$

Subject-level means or effects are estimated as:

$$\beta_{i} \sim N(\beta_{G},\sigma^{2})$$

This step will pull subject-level effects towards the mean \\\beta_{g}\\ depending on the uncertainty at the population and group levels. So it regularizes your estimates for individual subjects by modeling them as "draws" from a normal distribution. But what is the right move if you don't think your effects are sampled from a larger population? What if you still want to regularize them with the mean? We ended up in an email conversation with [Niall Bolger][link2], repeated measures [wizard][link3]. The following is that conversation, edited for clarity, length, and errors on my part. Skip to the end if you want a summary!

+Read the conversation
 -----------------
**Niall Bolger:**
I had fun reading your report on your analyses of the personality datasets from Michigan State. Estimating a single model for all five personality factors is the right way to go. Whether to treat the factors themselves as fixed effects or as realizations of a random variable is a controversial issue. Big 5 personality psychologists would be dead against it, I imagine. They see each factor as a distinct, fixed dimension. 


Just to be clear about the point about treating factor as random. For example:

```{.r}
m<-lmer(score ~ 1 +  (1 | id)+ (1 | factor) + (1 | univ), data=mipiplong)
```

what you are doing is saying that the scores on the five different personality dimensions vary around a common value, the intercept, and that scores on particular scales are just random draws (or realizations) from a common normally distributed variable. That would make sense me if you were talking about the variation in item scores around a given fixed effect of a particular dimension, say, Openness. But it makes less sense to me for the five dimensions themselves. 


**Bruce Dore:**
I was thinking treating factor as random could be practical in terms of generating a single more informed estimate of the university-level variance -- especially since it doesn't seem like the different Big 5 factors each show different degrees of variance across universities. I was thinking like, even if it makes sense to estimate the means as fixed, do we really think it makes sense to make 5 separate noisy estimates of variance, rather than a single more informed estimate?


But in a way this is a matter of perspective, right?  that is, perhaps it is reasonable to think of the OCEAN factors we get from this questionnaire as a random sample of a distribution of all possible OCEAN factors -- or all possible personality factors -- that can be measured.


**Niall:**
I do see your point, Bruce, but at some level I don't get it. I remember coming across this idea years ago when I was playing with some models and data from Gelman & Hill. In a particular example they treated race (black, white, hispanic, other) as a random effect, and I could not get my head around it. 


**Bruce:**
I agree that treating big 5 factor as random is, on some level, really weird. also, i have been using the term "estimates" in a sort of sloppy way above. in the random variable case, i really mean BLUPs generated in the manner you describe above. 


**Raphael Gerraty (Me!):**
I tend to be on the side of pooling uncertainty in every way that is plausible, so I am usually pro making these things "random" variables. I think it's helpful to think about this as an estimation/uncertainty decision rather than a statement that the personality factors are inherently fixed vs random. I think you can interpret pooling uncertainty across personality factors as placing an "empirical prior" distribution on each personality likelihood. I like this because if the data strongly indicates that they are distinct (while pooling across individual subject and university sites, say), the model will incorporate this. 


In multilevel models, the prior itself (the random effects distribution) has a scale parameter that is fit to the data! So we can let the data speak for itself in the sense that the model will allow some combination of "outlier" random effects and high (approaching a uniform distribution) variance, if the likelihoods at the lower levels support this. And if the data are somewhat ambiguous, then i think our models and statements about the variables should reflect this ambiguity.


**Niall:**
I remember in grad school learning about empirical-Bayes analyses of experimental data that traded off potential bias for superior predictive accuracy. It allowed means across conditions to shrink toward one another. That could not happen in fixed effects analyses. The empirical part came from allowing the data to speak about how much to shrink (pool?). 


**Bruce:**
Just wanted to contribute to this discussion by pointing us to this relevant excerpt from the [2005 Gelman ANOVA paper][link4] , which i have read many times and am now starting to understand a little better.


**Raphael:**
I think the discussion of computing superpopulation \\\sigma\\ vs finite-population \\s\\ variance may be useful for thinking about in what sense factors whose levels have been exhausted can be considered grouping variables or random effects, but it's not clear to me whether the focus on this finite-population variance allows us to disregard "definition 3" from the ANOVA paper (that random effects must be sampled from a larger population). 


**Niall:**
In a Bayesian analysis, is the finite sample standard deviation the variation between the shrunken estimates/predictions of a categorical variable (when it is treated as random); and is the superpopulation standard deviation what I would normally call the random effect for that categorical variable?


**Raphael:**
The superpopulation \\\sigma\\ characterizes the uncertainty for predicting a new coefficient from your grouping variable, whereas the finite-population $s$ describes the existing coefficients


So on the one hand, these reflect one part of the FE/RE distinction, in that one of them refers to the variation between the effects sampled and the other refers to the effects you could sample. But on the other hand, for a random effects analysis you have estimated \\\sigma\\ to regularize your effects that you use to compute \\s\\!


I am not sure what exactly the \\\sigma\\ term is supposed to mean if you are claiming to have sampled every level, as in the personality example, but I also found this Yates quote from the paper helpful:

>“. . . whether the factor levels are a random selection from some defined set (as might be the case with, say, varieties), or are deliberately chosen by the experimenter, does not affect the logical basis of the formal analysis of variance or the derivation of variance components.”


**Niall:**
Thanks Rapael. I see the distinction you are making, but what I am confused about is what are the coefficients that constitute the finite population. They don't seem to be the OLS estimates you would get in traditional anova. That's why I'm wondering if they are the shrunken estimates--shrunken based on the superpopulation variance \\\sigma^{2}\\?


**Raphael:**
My understanding is that in lme4 they are ML/REML-derived modes of the conditional data likelihoods or in stan they are the full marginal posterior distributions of the effects given the data. In either case I think they would be estimates shrunk by the \\\sigma\\. Which is maybe why Doug Bates also says that they should be non-exhaustive samples from a population of levels, consistent with the older definition of random effects. 


50 states is maybe a better example for this question than the 5 personality measures. I guess if you think you have all the levels of some grouping variable, the question is what sense it makes to constrain estimates with a parameter that models something that may not exist (the super-population)?


**Niall:**
Ok Raphael. In my lingo that means they are sometimes called BLUPs, Empirical Bayes predictions, or realized values of the random effect distribution, which is what I thought they might be. This make sense, as long as there is a good rationale for a superpopulation.


Without a sensible superpopulation, it seems hard to justify treating levels of a factor as realizations of a random variable. You would be letting the data speak, but perhaps they are speaking gibberish. 


Repeated measures ANOVA and traditional mixed models do rely on normal distributions to characterize random effects. But I have always thought of that as a computational problem, one that is now obsolete so long as you are willing to use Bayesian estimation.


As far as definitions of fixed and random go, definition 5 in the ANOVA paper is by far the most common today (in books on mixed models and multilevel models). If effects are fixed, their estimates are not shrunken based on a variance estimate, whereas if they are random they are shrunken. 


**Raphael:**
You could theoretically use any distribution, but normal is normal. is there a better motivated distribution than normal for the following case: I want to pool my estimates to generate posteriors for a number of different levels, but I have no reason to believe these levels are "random" samples? That would basically solve all of this wouldn't it?


**Niall:**
I think the basic question is whether you want to treat them as realizations of a random variable--whatever its distribution might be. If they come from a random variable, then the best way to model them is as random effects. If there is no randomness at all in how they came about, then the only way I know to treat them is as fixed, and their estimates won't be shrunken in the BLUP or Bayesian sense. What if there is some randomness? I'm not sure how to handle that situation.


**Raphael:**
We didn't solve anything, but I think we have clarified some things. Here is where I think we are:

1) Whether you call it a random effect or varying estimates within a grouping variable, the procedure we are talking about models these effects as varying around an mean with something like $\beta \sim N(0,\sigma^{2})$.  Depending on how bayesian you are willing to get, the effects are either full posterior distributions or predictions on which the likelihood is conditional. 

2) While this approach differentiates between superpopulation $\sigma$ and finite-population $s$, it is still the case that you generate the finite-population $s$ by regularizing your effects with superpopulation $\sigma$. 

3) In cases where you can safely say these effects were sampled from some larger group of levels (like subjects, universities), we all agree this is a good approach: you should pool your uncertainty to regularize the estimates.

4) It is less clear what to do if you think or know that you have all the levels there are (states definitely, personality factors and races maybe?). The older interpretation is that these are not randomly sampled from a larger population, so it doesn't make sense to model them as normally distributed.

5) For some of us, though, there is a desire to regularize estimates whenever we can, especially if there are multiple estimates! We want to pool our uncertainty to get better estimates. Positing a normal distribution of effects does this, even if we can't really say "we sampled these effects from a larger population." How does this make sense though?

6) My naive response is just to say that the probability distribution is not a model of the frequency of random samples, it is a representation of our uncertainty about the distance of these effects from their average. This is probably not justification enough!

To be continued...


[link1]: https://brucedore.github.io/
[link2]: https://columbiacoupleslab.wordpress.com/lab-members/
[link3]: https://www.amazon.com/Intensive-Longitudinal-Methods-Introduction-Methodology/dp/146250678X/ref=sr_1_1?ie=UTF8&qid=1473815695&sr=8-1&keywords=niall+bolger
[link4]: http://www.stat.columbia.edu/~gelman/research/published/AOS259.pdf

