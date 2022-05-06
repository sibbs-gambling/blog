---
title: '__IN PROGRESS__ Comparing Standard Error Estimation Methodologies'
description: 'Comparing SE estimation methods as used in finance.'
date: 2022-04-23
tags: [
    "finance", "econometrics", "statistics"
]
markup: "mmark"
math: true
mathjax : true
katex: true
header-includes:
  - \usepackage{algorithm}
  - \usepackage{mathrm}
  - \usepackage{amsmath}
---

This article compares various ways of estimating the standard errors of
regression coefficients as used in the finance literature.

<!--more-->

__Disclaimer:__ This blog post is a write up following my reading of Petersen (2009).
This is a very well written paper intended to give researchers guidance on standard error
estimation, this type of contribution to the literature is invaluable!
I follow many of the paper's derivations and logic closely.
***Therefore, all credit (and big thanks) goes to Mitchell A. Peterson, I am merely reflecting my thoughts on his
original work.***


=====================
Note to self:
See here https://quant.stackexchange.com/questions/35756/fama-mac-beth-1973-vs-fixed-effect
and pages saved in raindrop statistics/econometrics
=====================


# Introduction
As the financial econometrics literature has progressed over the last handful of
decades there has much research into methodologies for estimating standard
errors. Standard OLS inference methodologies (i.e. those that follow the
Gauss-Markov theorem) rely on two parts of the Gauss-Markov theorem heavily.
Specifically that, in a standard panel regression setup,

$$ Y_{it} = X_{it} \beta + \epsilon_{it}, $$

the assumptions below are made.

$$
\mathrm{E}[\epsilon|X] = 0  
$$

which to be clear, asserts that $\mathrm{E}[\epsilon_i|X] = 0  \space \forall \space i$. One caveat is that
if $X$ is viewed as non-random and observable, we can simply drop the conditional i.e. $\mathrm{E}[\epsilon] = 0$.
As an aside, viewing $X$ as fixed or not is an ongoing debate between fields. Some researchers feel as if
the social sciences (e.g. econometrics) relies too heavily on this way of thinking because it should predominantly
be reserved for applications when the researcher directly controls $X$ such as in more experimental sciences.
For a discussion of this debate as it relates to econometrics see Binkley & Abbott (1987). The second
assumption that is relied on is,

$$
\mathrm{Var}[\epsilon | X] = \mathbb{E}[\epsilon\epsilon^T | X] = \sigma_\epsilon^2 I.
$$

This assumes so-called spherical errors, which is a term that encompasses two important
components. One is homoskedasticity, or uniform variance $\sigma_\epsilon^2$
across errors (over both the $i$ and $t$ dimensions). The second is independence
between the errors, i.e. $\mathrm{Cov}(\epsilon_{it}, \epsilon_{js})=0 \space \forall \space it \neq js$.
While this seems straightforward, lets take a moment to think about what this means.
It means both the across firm (i.e. $i \neq j$) and across time (i.e. $t \neq s$) the correlation
between errors is zero. Or as a concrete example, for some arbitrary panel regression of the monthly returns of S&P 500 stocks onto firm
  characteristics, say book-to-price, profitability, etc., AAPL's noise return (the error term) from January 2021
both has no correlation with AAPL's noise return from February 2021 nor AMZN's noise return from
January 2021. This is a fairly restrictive assumption that is more than likely *not*
met in the data.

The literature on standard error estimation deals exactly with what to do in the case that the
error covariance matrix $\mathbb{E}[\epsilon\epsilon^T | X] \neq \sigma_\epsilon^2 I$,
 either because of firm-effects (non-zero correlation *within* a firm, across time) or time-effects
(non-zero correlation *within* a given time period, across firms), or both.

What is surprising to me is the lack of uptake in the finance literature for a
more robust approach to standard error estimation. While it is a somewhat dated
reference, Peterson (2009), cites that 45% of the articles in JoF, JFE, and RFS
(three of the most respected academic journals in Finance), do not report adjusting
standard errors. In many papers the exact nature of the correlation in errors
and thus the inflation/deflation of t-statistics as used in inference tests is unclear.
This calls for testing multiple standard error estimation methodologies on the data and determining, if there
are large discrepancies between methodologies, why they exist.
Very few papers report SEs from multiple methodologies, doing so seems
a great way to both fight against so called p-hacking (read: my t-stat is 2.01, that means my
  proposed relationship is significant right??, not noted are the 10 other specifications that saw a t-stat < 2),
  as well as helping to push forward new developments in the standard error literature.


# Basics

#### Deriving $\hat{\beta}$
It is always best to return to the basics when doing any derivations,
so below we will derive the expression for our estimator of $\beta$ in a standard panel regression setup
(as any undergraduate course worth its salt covers),

$$ Y_{it} = X_{it} \beta + \epsilon_{it}.$$

Where the subscript $i$ denotes a firm of which there are $N$ and $t$ denotes a month
of which there are $T$. So $Y$ is $NT \times 1$, $X$ is $NT \times k$, and $\epsilon$ is $NT \times 1$.
$k$ is the number of RHS variables we include in our model, including a column of 1's as the intercept
(it is WLOG to include an intercept but gives us some easier to work with math later).
As in the previously mentioned example of
regressing returns on firm characteristics, one column of $X$
might be our book-to-price measurements for each firm-month observation.

Our stated goal in ordinary *least squares* (OLS) is to choose an estimate of $\beta$, which
we denote as $\hat{\beta}$, to minimize the sum of squared **residuals**, $e=Y-X\hat{\beta}$.
Note that these are not the unobservable errors $\epsilon$, this is a subtle but important distinction
that $ e \neq \epsilon$. We are thus minimizing the following expression,

$$
\begin{align}
e^Te &= (Y-X\hat{\beta})^T(Y-X\hat{\beta})\\
&= Y^TY - 2 \hat{\beta}^TX^Ty + \hat{\beta}^TX^TX\hat{\beta}.
\end{align}
$$

We then take the derivative of $e^Te$ wrt $\hat{\beta}$ and set to zero to find its minimum.

$$
\begin{align}
\frac{\partial }{\partial \hat{\beta}} e^Te &= \frac{\partial }{\partial \hat{\beta}}Y^TY - \frac{\partial }{\partial \hat{\beta}}2 \hat{\beta}^TX^TY + \frac{\partial }{\partial \hat{\beta}}\hat{\beta}^TX^TX\hat{\beta}\\
0 &= 0 - 2 X^TY + 2 X^TX\hat{\beta}\\
X^TX\hat{\beta} &= X^TY  \\
\underbrace{(X^TX)^{-1}X^TX}_{I}\hat{\beta} &= (X^TX)^{-1}X^TY\\
\hat{\beta} &= (X^TX)^{-1}X^TY
\end{align}
$$

#### Deriving the *Standard* Standard Errors (SEs)
The standard error of an estimator is the standard deviation of the estimator's sampling distribution.
Below we derive it (more specifically the variance, which is the SE$^2$) for the Gauss-Markov case (i.e. the standard setup) as
discussed in the Introduction. First we do some algebra after replacing $Y$ in our above equations.

$$
\begin{align}
\hat{\beta} &= (X^TX)^{-1}X^TY \\
&= (X^TX)^{-1}X^T(X\beta + \epsilon)  \space  \space \space \text{(note that $\beta \neq \hat{\beta}$ and $\epsilon \neq e$)}\\
&= \underbrace{(X^TX)^{-1}X^TX}_{I}\beta + (X^TX)^{-1}X^T\epsilon\\
&= \beta + (X^TX)^{-1}X^T\epsilon\\
\hat{\beta} - \beta &= (X^TX)^{-1}X^T\epsilon\\
\end{align}
$$

Note that the OLS estimate is unbiased, in that $\mathrm{E}[\hat{\beta}]=\beta$, which I leave to the reader to
look up the (short) proof of. Also note that $\beta$ is a constant population parameter, so has zero variance, think of it as a value given from on high but unobservable to us. This means that,

$$
\begin{align}
\mathrm{Var}[\hat{\beta} - \beta] &= \mathrm{E}[(\hat{\beta} - \beta)(\hat{\beta} - \beta)^T] - (\mathrm{E}[\hat{\beta} - \beta])(\mathrm{E}[\hat{\beta} - \beta])^T\\
&= \mathrm{E}[(\hat{\beta} - \beta)(\hat{\beta} - \beta)^T] - (\mathrm{E}[\hat{\beta}] - \mathrm{E}[\beta])(\mathrm{E}[\hat{\beta}] - \mathrm{E}[\beta])^T \\
&= \mathrm{E}[(\hat{\beta} - \beta)(\hat{\beta} - \beta)^T] - \underbrace{(\beta - \beta)(\beta - \beta)^T}_{0}\\
&= \mathrm{E}[(\hat{\beta} - \beta)(\hat{\beta} - \beta)^T].
\end{align}\\
$$

For full exposition, as many common reference texts simply assert it to be true, note the below proof that
$ \mathrm{Var}[\hat{\beta} - \beta] = \mathrm{Var}[\hat{\beta}]$, which is obvious when we view $\beta$ as a constant given the
property of the variance operater that $\mathrm{Var}[X+c]=\mathrm{Var}[X]$ where $c$ is a constant but I show it nonetheless.

$$
\begin{align}
\mathrm{Var}[\hat{\beta} - \beta] &= \mathrm{E}[(\hat{\beta} - \beta)^2] - (\mathrm{E}[\hat{\beta} - \beta])^2\\
&= \mathrm{E}[\hat{\beta}^2] - 2\mathrm{E}[\hat{\beta}\beta] + \mathrm{E}[\beta^2] - (\mathrm{E}[\hat{\beta}] - \mathrm{E}[\beta])^2\\
&= \mathrm{E}[\hat{\beta}^2] - 2\mathrm{E}[\hat{\beta}]\beta + \beta^2 - \mathrm{E}[\hat{\beta}]^2 + 2 \mathrm{E}[\hat{\beta}]\beta - \beta^2\\
&= \mathrm{E}[\hat{\beta}^2] - \mathrm{E}[\hat{\beta}]^2 \\
&= \mathrm{Var}[\hat{\beta}]
\end{align}
$$

We now have the building blocks to find $SE^2(\hat{\beta})$.

$$
\begin{align}
SE^2(\hat{\beta}) = \mathrm{Var}[\hat{\beta}] &= \mathrm{Var}[\hat{\beta} - \beta] \\
&= \mathrm{E}[(\hat{\beta} - \beta)(\hat{\beta} - \beta)^T] \\
&=\mathrm{E}[((X^TX)^{-1}X^T\epsilon)((X^TX)^{-1}X^T\epsilon)^T] \\
&=\mathrm{E}[((X^TX)^{-1}X^T\epsilon)(\epsilon^TX(X^TX)^{-1})] \\
& \text{(note that $X^TX$ is symmetric so $((X^TX)^{-1})^T=(X^TX)^{-1}$)}\\
&=\mathrm{E}[(X^TX)^{-1}X^T\epsilon\epsilon^TX(X^TX)^{-1}] \\
\end{align}
$$

And if we view $X$ as non-random we get,

$$
SE^2(\hat{\beta}) = \mathrm{Var}[\hat{\beta}] = (X^TX)^{-1}X^T\mathrm{E}[\epsilon\epsilon^T]X(X^TX)^{-1}.
$$

__The portion of this expression, $\mathrm{E}[\epsilon\epsilon^T]$, is primarily what we are concerned with in this post, that is what we cover in the next section.__


##### Standard Errors that Account for Non-Spherical Errors
##### Standard Errors that Account for Non-Spherical Errors

As noted above in the introduction, per the Gauss-Markov assumptions (and again that we are viewing $X$ as fixed) we have,

$$
\begin{align}
SE^2(\hat{\beta}) &= (X^TX)^{-1}X^T\sigma_\epsilon^2IX(X^TX)^{-1}\\
&= \sigma_\epsilon^2 (X^TX)^{-1}X^TX(X^TX)^{-1} \\
&= \sigma_\epsilon^2 (X^TX)^{-1}.
\end{align}
$$

One would normally estimate the population statistic $\sigma_\epsilon^2$ with
something like $\sigma_e^2 = \frac{e^Te}{N-k}$, see the widely used econometrics textbook by Greene for the derivation.

This means our error covariance matrix (which is $NT \times NT$) is estimated by the matrix graphically shown below.
For easy exposition I follow Peterson's clever way of displaying this, where we have
for example
$N=2$ firms and $T=4$ months, and thus the entry in (row 4, column 1) of the (Firm 1, Firm 2)
 sub-matrix refers
to our estimate of (or assumption about), the covariance between $\epsilon_{i=1, \space t=4}$ and $\epsilon_{i=2, \space t=1}$.

In the standard setting, all of the off-diagonal entries are zero (independence)
and the on-diagonal entries are all equal (homoskedastic).


<center>
__Standard (Non-Robust) Estimator of Error Cov__
</center>

<table>
    <thead>
        <tr>
            <td> </td>
            <td colspan=4><center>Firm 1</center></td>
            <td colspan=4><center>Firm 2</center></td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=5>Firm 1</td>
        </tr>
        <tr>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>             
        </tr>
        <tr>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>  
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>  
        </tr>        
    </tbody>
    <tbody>
        <tr>
            <td rowspan=5>Firm 2</td>
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>             
        </tr>                
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>
            <td>0</td>             
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>
            <td>0</td>  
        </tr>
        <tr>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>0</td>
            <td>$\frac{e^Te}{N-k}$</td>          
        </tr>
    </tbody>
</table>




##### Standard Errors that Account for Non-Spherical Errors
What I mean by non-spherical is any case where $\mathbb{E}[\epsilon\epsilon^T | X] \neq \sigma_\epsilon^2 I$.






For now we will set aside questions of estimating the error variance

# References
[1] Petersen, Mitchell A. "Estimating standard errors in finance panel data sets: Comparing approaches." The Review of financial studies 22.1 (2009): 435-480.\\
[2] Binkley, James K., and Philip C. Abbott. "The fixed X assumption in econometrics: can the textbooks be trusted?." The American Statistician 41.3 (1987): 206-214.






<!-- Include the list of cited works on the page -->
