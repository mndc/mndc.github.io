---
layout: post
title:  "Comparing distributions in kdb"
date:   2012-12-29 12:00:00
categories: kdb
---
I have to compare two data sets in KDB -- I can't tell you the source of data, unfortunately. I had to decide whether they are significantly different, and I have to do this often, quickly and programmatically, therefore I need to avoid exporting the data and processing it with some other tool. My wife recommended me to

   * decide whether the data is normally distributed using the Kolmogorov-Smirnov test
   * if it is, then use a paired-t test, otherwise use the Wilcoxon signed-rank test.

Kolmogorov-Smirnov can be used to compare a sample with a reference probability distribution, in my case, I have to compare the empirical distribution function with the normal distribution's CDF.

Side note: I could not find a built-in random generator which draws from the normal distribution -- only uniform samples can be generated. I decided to use the Box-Muller transform. 

$$
	Z_0 = R \cos(\Theta) =\sqrt{-2 \ln U_1} \cos(2 \pi U_2)
$$

.. which translates to

```
nrand:{:(sqrt neg 2 * log x?1.0) * sin 2*(acos -1)*x?1.0 }
```

-- to verify that this produces normal distribution:

```select avg x, dev x from ([] x:nrand 1000000) / result=~(0 1)```


We need the CDF for the normal distribution. I don't need a particularly fast or precise CDF for my purposes, hence I have used Marsaglia's method:


$$
\Phi(x) = \frac12 + \phi(x)\left( x + \frac{x^3}{3} + \frac{x^5}{3\cdot5} + \frac{x^7}{3\cdot5\cdot7} + \frac{x^9}{3\cdot5\cdot7\cdot9} + \cdots \right)
$$


translated to KDB/Q:

```npdf:{(1f%sqrt 2 * pi)*exp neg (1f%2f) * x xexp 2};```

```ncdf_marsaglia:{r:0.5 + (npdf x)*sum {(y xexp x) % prd 1 + 2* `real$til `int$0.5*x+1}[;x] each 1 + 2 * til 30;:(x>-4.9)*(r*x<4.9)+(1f*not x<4.9)}```


The Kolmogorov-Smirnov statistic is

$$
D_n=\sup_x |F_n(x)-F(x)|
$$

We have to normalise our data, then compare the EDF with the normal distribution's CDF.


```KS_stat:{nx:(x-avg x)%dev x;:max abs (ncdf_marsaglia asc nx) - (1%count nx) * til count nx}```

Example:

```KS_stat (nrand 1000) xexp 2 ```

> 0.23



This D value can be used to check the hypotheses using the K-S table. Remember to use relatively high confidence levels as the Marsaglia method above is not too precise (it'd be interesting to see how the K-S value changes with the number of iterations in the CDF-generator..)


