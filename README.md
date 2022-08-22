# Unit transform
For Bayesian models with Dirichlet prior one has to be able to transform vectors so that they sum to one. This is one such transformation.

In Stan's reference manual the unit simplex transform[1] is not simplex but rather complex. In this document I present a simpler transformation for all of your unit simplex needs. Well, at least in my opinion this is simpler to understand.

## A simpler transform

The simpler transform I propose here consists of two steps.

Given that we have sampled the vector y from unconstrained scale (the length of this vector is always one less than the number of the parameters of the Dirichlet distribution):

1) Transform y into ordered vector, y_ordered
2) Use some cumulative distribution function to transform y_ordered into proportions

The distribution to be used is here assumed to be the logistic distribution, since it has a very simple form.

The idea is similar to how cutpoints are defined in multinomial logit models: concretely, in the first step we sample the cut points and in the second step we find the areas of the distribution between the cutpoints. Since probability distributions always integrate to one, this leads to a vector which sums to one, by construct.

The ordered transform can be done simply by exponentiating all but the first sample and then finding the cumulative sum[2]. 

Finding the proportions is ever so slightly more complicated. The first proportion is simply the integral from negative infinity to the first sampled value; the last is, on the contrary, 1 minus integral to the last sampled value. Values in between are always found by first finding the integral the cutpoint i and substracting the integral to the cut point i - 1 from it.


## Jacobian

Jacobian for the ordered transform is simply the product of the exponentiated values of the sampled parameters[2]. For the logit transform the Jacobian is logistic(y) * (1 - logistic(y)). The first sample does not undergo the ordered transform, but every other sample is first transformed to the ordered scale and then logit transformed.

See R code of the simulation for a practical implementation.

## A simulation

To test the transformation, I programmed a simple Metropolis algorithm that samples from Dirichlet distribution. At face value, based on this simulation, it would seem that everything should be functioning as intended.

Histograms show the approximated marginals of Dirichlet(2, 56, 20) distribution against analytically known marginals:

![Results from the simulation](https://github.com/joanpaak/UnitTransform/blob/main/simulation_1_results.png)



# Sources

[1] https://mc-stan.org/docs/reference-manual/simplex-transform.html
[2] https://mc-stan.org/docs/reference-manual/ordered-vector.html
[3] https://mc-stan.org/docs/reference-manual/logit-transform-jacobian.html
