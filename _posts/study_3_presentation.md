---
title: 'Dissertation Study 3'
date: 2012-08-14
permalink: posts/dissertation_3/
tags:
  - cool posts
  - category1
  - category2
---

## Using a differences-in-differences framework to analyze to role of group
threat in social movement mobilization

One study of my dissertation analyzes the causal impact of changing
demographics within a school on student participatoin in boycotts of
annual standardized tests. Based on research in social movements theory
and educational policy, I hypothesize that when schools see increases in
the share of students of color, students are more likely to
participating in testing boycotts. This is called *group threat*.
Basically, it predicts that members of majority groups will mobilize to
take collective action when they are exposed to more members of minority
groups.

My case is the opt-out movement–a coalition of parents and educators who
oppose the use of standardized tests for school accountability
purposes–what they call “high-stakes” tests. The movement emerged
around 2010, but really picked up steam in 2013. By 2016, around 20% of
students in New York, where the movement is the strongest, boycotted the
annual tests.

Reasons for joining movements are multifaceted. People choose to join
movements for ideological or personal reasons. I hypothesize that group
threat played an important role, net of other factors.

To test this hypothesis, I use a panel of school-level accountability
data from New York state. These data are publicly available from the
Department of Education’s [website](http://data.nysed.gov). I
supplemented these data with data from the Common Core of Data, which
contains administrative and demographic data for every school in
America.

I have already created my dataset. I use data from the 2007-2008 until
the 2016-2017 school year. The dataset contains demographic data,
testing data, and–most important of all–participation data. So for each
school in New York, I have data on the number of student who
participated, the average tests scores by grade, and the proficiency
rates by grade. I will describe variables I use in a moment.

To identify the causal relationship between demographic changes and
testing boycotts, I use a **differences-in-differences** framework, a
common econometric technique for making causal inferences from
observational data. There are several problems with observational data,
in contrast with experimental data, when it comes to causal inference.
In this case, maybe the school that experienced demographic changes were
different from schools that did not experience those changes in critical
ways related to test boycotts. Say, for example, that the parents of
students of color were more likely to move their children to more
schools where parents were highly educated. Well, more educated parents
are more likely to have the time and resources to join social movements.
So any relationship between demographic changes and testing boycotts
would be spurious–the true cause would be parental education.

The differences-in-differences framework gets around this problem by
netting out any pre-existing differences between the two groups of
schools. How?

Image that we have an experiment. Schools in the experimental or
treatment condition have an increase the share of students of color. The
schools in the control group do not. The treatment is applied at a
particular point in time (here, beginning in the 2012-2013 school year,
the first year of meaningful participation in the boycott). In the
experiment, with random assignment of schools to treatment and control
conditions, we can simply find the mean difference in participation in
the boycott and *wham\!* We’ve estimated the causal impact of group
threat.

Well with observational data, we don’t have the luxury. We have to
accept that schools in the “treatment” group and the “control” group are
different in many ways, some of which we can statistically control for,
some of which (the more pernicious of which) we cannot because they are
unobservd. But we can make a reasonable assumption–that, regardless of
their differences, the schools in the two groups have *parallel trends*
in the outcome measure in absence of the treatment. That is to say,
while the schools have different *levels* of an outcome, the *trends
over time* are the same. The application of the treatment for the
treatment group changes the trend line for that group. This departure
from the hypothetical trend is the causal effect of the treatment.

To do this is actually quite simple–find the difference in the outcome
from time 0 to time *t* from the control group; find the differences in
the outcome from time 0 to time *t* from the treatment group; and then
find the difference between those differences. In a regression
framework, it is a bit more than simply finding differences, since I
will want to control for time-varying variables that I believe are
correlated with boycotts of the tests.

Enough background, here’s the data.

### Exploratory analysis

Prior to the 2012-2013 (from now on, I will refer to the school year by
the year-end) school year, basically all students in New York took the
annual standardized tests. These tests are mandatory and there are
potential penalities for schools if students do not take them. For the
most part, students did well on the test–about 70% proficiency rates
overall. But beginning in 2013, the state adopted a new, more
“rigorous”, assessment aligned with the recently developed Common
Core State Standards. Proficiency rates dropped, helping to fuel the
protest movement. No in this context, I hypothesize that group threat
will increase testing boycotts. Without getting bogged down in
sociological theory, the basic idea is that parents (particularly white
parents) associate students of color with poor school performance and
increased government regulation of schools. This threatens educational
goods they feel entitled to–such as a rich diverse curriculum,
extra-curricular activities, child-centered instruction. Research
suggests that these goods are less likely found in schools that
experience government regulation.

Note: For simplicity, I focus on the mathematics test. Students also
take tests in reading and science. Note 2: I only use data from K-8
schools. The testing situation in high schools is much more complicated.

Starting in 2013, the proportion of students not participating in annual
math testing increased, maxing out at about
20%.

![](study_3_presentation_files/figure-gfm/overall%20opt%20out%20trend-1.png)<!-- -->

The students participating in the boycott are primarily, but not
exclusively,
white.

![](study_3_presentation_files/figure-gfm/trends%20by%20race-1.png)<!-- -->

Schools that had increases in racial diversity between 2013 and 2016 had
more students participating in the boycott than schools that did not–a
bit of evidence for the group threat
hypothesis.

![](study_3_presentation_files/figure-gfm/differences%20by%20increases%20in%20diversity-1.png)<!-- -->

Again, we see this trend is true if we focus only on white
students.

![](study_3_presentation_files/figure-gfm/differences%20for%20white%20opting%20out-1.png)<!-- -->

### Differences-in-differences analysis

My main dependent variable is the proportion of students boycotting the
annual math standardized test.

My main independent variable is the racial diversity of a school,
captured using the Simpson diversity index common in ecological
research. The index measures the population of each group in some
setting–in this case, race/ethnicity groups within schools. My
hypothesis is that schools that experience increases in racial diversity
will have higher proportions of students boycotting the test. So schools
with increases are in the “treatment” group. So I will create an
indicator for schools that have a net increase in racial diversity
during the treatment period–between 2013 and 2016. I have to choose a
cutoff here–what increase is meaningful, according to this measure?

``` r
library(tidyverse)
ny_panel <-
  ny_panel %>% 
  mutate(
    school_racial_diversity = 
      ny_panel %>% 
      select(per_hisp, 
             per_white,
             per_black,
             per_asian)  %>% 
      vegan::diversity(index = "simpson")) %>% 
  group_by(entity_cd) %>% 
  mutate(
    change_diversity = 
      round(school_racial_diversity, 2) - round(dplyr::lag(school_racial_diversity), 2)
  )

summary(ny_panel$school_racial_diversity)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##  0.0000  0.1717  0.3792  0.3659  0.5446  1.0000

``` r
summary(ny_panel$change_diverse)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ## -1.0000 -0.0111  0.0010  0.0041  0.0190  1.0000    2313

A 0.02 point increase in the index is the 75th percentile, so that seems
like a reasonable cutoff. For schools that have an increase between 2013
and 2016, `increase_diverse == 1` and `increase_diverse == 0` otherwise.

``` r
ny_panel <-
  ny_panel %>%
  mutate(
    increase_diverse = 
      as.numeric(sum(change_diverse[year >= 2013 & year <= 2016], 
                     na.rm = T) > 0.02)
  )

table(ny_panel$increase_diverse)
```

    ## 
    ##     0     1 
    ## 12950 10180

To set up the differences-in-differences analysis, I need to create an
indicator for the treatment period. This is the period after 2013. So
for each school in the dataset, `post2013 == 1` when the year is after
2013 and `post2013 == 0` otherwise.

``` r
ny_panel <-
  ny_panel %>% 
  mutate(post2013 = as.numeric(year > 2013))

table(ny_panel$year, ny_panel$post2013)
```

    ##       
    ##           0    1
    ##   2008 2313    0
    ##   2009 2313    0
    ##   2010 2313    0
    ##   2011 2313    0
    ##   2012 2313    0
    ##   2013 2313    0
    ##   2014    0 2313
    ##   2015    0 2313
    ##   2016    0 2313
    ##   2017    0 2313

The differences-in-differences estimator is the interaction between
`post2013` and `increase_diverse`. This compares the proportion of
students boycotting the test in schools that had an increase in
diversity between 2013 and 2016 to those that did not *and* all schools
before 2013.

Here’s the basic regression
model:

\[Y_{it} = \alpha_0 + \beta_1 R_i * D + \beta_2 R_i + \beta_3 D + \beta_4 X_{it} + \epsilon_{it}\]
where \(Y_{it}\) is the proportion of students boycotting the test in
school \(i\) in year \(t\), \(D\) is the indicator for year after 2013,
and \(R_i\) is the indicator for the increases in racial diversity for
school \(i\). \(X_{it}\) is a vector of relevant time-varying covariates
and \(\epsilon_{it}\) is a school-year stochastic error term.

Since we have panel data, we can exploit within-school changes in order
to control for any fixed unobserved differences between schools. This
does a lot to help me aviod [omitted variables
bias](https://en.wikipedia.org/wiki/Omitted-variable_bias), a constant
threat to causal inference. Schools are different–situated in different
communities, with different teachers, administrators, school cultures.
All of these are unobserved in the data, but using fixed effects, we can
control for those between school differences. Likewise, there may be
changes across years that effect all schools–for example, the state may
adopt a new policy about testing that could impact the boycott. So I
will use both school and year fixed effects.

Now the model
is:

\[Y_{it} = \beta_1 R_i * D + \beta_2 X_{it} + \alpha_i + \gamma_t + \epsilon_{it}\]

where \(\alpha_i\) are school fixed effects and \(\gamma_t\) are year
fixed effects. The intercept \(\alpha_0\) and individual coefficients
for \(D\) and \(R_i\) drop out. In both cases, \(\beta_1\) is the
differences-in-differences estimator.

The relevant covariates in the vector \(X_{it}\) are: lagged proficiency
rates in mathematics, the percent of teachers with fewer than three
years experience, the percent of teachers with masters degrees or
higher, the percent of low income students, and the percent of students
who are English Language Learners.

### Differences-in-differences models

I will first do the vanilla regression (model 1) and then the fixed
effects regression (model 2). Since it is quite likely that there is
correlations within schools, it is wise to use clustered standard
errors.

``` r
did_reg <-
  as.formula(
    math_all_non_partic ~
      increase_diverse*post2013 +
      lag(math_non_prof_rate) +
      per_fewer_3yrs_exp +
      per_mas_plus +
      per_frpl +
      per_lep
  )

did_reg_fit <- 
  lm(did_reg, data = ny_panel)

lmtest::coeftest(did_reg_fit, 
                 vcov = vcovHC(did_reg_fit,
                               type = "HC1"))
```

    ## 
    ## t test of coefficients:
    ## 
    ##                             Estimate Std. Error  t value  Pr(>|t|)    
    ## (Intercept)                0.0346845  0.0017117  20.2632 < 2.2e-16 ***
    ## increase_diverse          -0.0125567  0.0005999 -20.9312 < 2.2e-16 ***
    ## post2013                   0.0960140  0.0022862  41.9974 < 2.2e-16 ***
    ## lag(math_non_prof_rate)    0.0145007  0.0025735   5.6347 1.777e-08 ***
    ## per_fewer_3yrs_exp        -0.0994172  0.0085490 -11.6291 < 2.2e-16 ***
    ## per_mas_plus               0.0115681  0.0034285   3.3741  0.000742 ***
    ## per_frpl                  -0.0485697  0.0024504 -19.8213 < 2.2e-16 ***
    ## per_lep                   -0.0764149  0.0062439 -12.2384 < 2.2e-16 ***
    ## increase_diverse:post2013  0.0631506  0.0039846  15.8487 < 2.2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

As we can see, the interaction between `increase_diverse` and `post2013`
is significant and positive. Schools with increases in diversity has
about 5 percentage points more students boycotting the test than schools
without increase.

Let’s see if adding the fixed effects makes this association go away.

``` r
library(plm)

did_fe_fit <-
  plm(did_reg,
      data = ny_panel,
      index = c("entity_cd", "year"),
      model = "within",
      effect = "twoway")

lmtest::coeftest(did_fe_fit, 
                 vcov = vcovHC(did_fe_fit,
                               type = "HC1"))
```

    ## 
    ## t test of coefficients:
    ## 
    ##                             Estimate Std. Error t value  Pr(>|t|)    
    ## lag(math_non_prof_rate)    0.0441438  0.0147313  2.9966 0.0027346 ** 
    ## per_fewer_3yrs_exp        -0.1200539  0.0174817 -6.8674 6.791e-12 ***
    ## per_mas_plus               0.0467349  0.0177512  2.6328 0.0084777 ** 
    ## per_frpl                   0.1946266  0.0124051 15.6892 < 2.2e-16 ***
    ## per_lep                    0.1933615  0.0497529  3.8864 0.0001022 ***
    ## increase_diverse:post2013  0.0555687  0.0051146 10.8646 < 2.2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

The coefficient on the interaction dropped by a percentage point, but
remains significant. But this is for all students; we can look
specifically at boycotting among white students.

``` r
did_reg <-
  as.formula(
    math_white_non_partic ~
      increase_diverse*post2013 +
      lag(math_non_prof_rate) +
      per_fewer_3yrs_exp +
      per_mas_plus +
      per_frpl +
      per_lep
  )

did_fe_fit <-
  plm(did_reg,
      data = ny_panel,
      index = c("entity_cd", "year"),
      model = "within",
      effect = "twoway")

lmtest::coeftest(did_fe_fit, 
                 vcov = vcovHC(did_fe_fit,
                               type = "HC1"))
```

    ## 
    ## t test of coefficients:
    ## 
    ##                             Estimate Std. Error t value  Pr(>|t|)    
    ## lag(math_non_prof_rate)    0.1630559  0.0185558  8.7873 < 2.2e-16 ***
    ## per_fewer_3yrs_exp        -0.1058064  0.0257502 -4.1089 4.003e-05 ***
    ## per_mas_plus               0.1000960  0.0227550  4.3989 1.098e-05 ***
    ## per_frpl                   0.1935351  0.0168013 11.5191 < 2.2e-16 ***
    ## per_lep                    0.0765386  0.0932864  0.8205     0.412    
    ## increase_diverse:post2013  0.0633321  0.0061544 10.2905 < 2.2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Again, the interaction is positive and significant, with about a 4
percentage point increases in testing boycotts among white students in
schools with increases in diversity.

Case closed, right? Not quite. This evidence is compelling, but there is
still potential bias in our results. Namely, if there are any changes
*within a school* that are correlated with both increases in racial
diversity and with testing boycotts, then the results are biased. There
is something else contributing to the boycotts that is not picked up by
my model. To give an exmaple, let’s say that schools make changes to
curriculum and instruction that attact families of color to them but are
not appealing to white families. In which case, these changes could
drive both increases in diversity *and* increases in boycotting. Ruh
roh. This is a plausible scenario, but one that I do not think is likely
based on my knowledge of the research literature. Regardless, it is
still plausible and I need to rule out this possibility. But how? These
within-school changes are unobserved\!

Well, for this to be the case, the changes in things like instruction
and curriculum must logically *precede* the changes in racial diversity.
So for schools that saw increases in diversity later on in the treatment
period, say in between 2016 and 2017, these changes had to have occurred
in the preceding years.

So, I will conduct a placebo test.

### A placebo test

I need to create a new “treatment” indicator for schools that had
increases in diversity after 2015, but not between 2013 and 2015.

``` r
ny_panel <-
  ny_panel %>% 
  mutate(
    placebo = 
      as.numeric(sum(change_diverse[year >= 2016], na.rm = T) > 0.02 &
                   sum(change_diverse[year >= 2013 & year <= 2015], na.rm = T) <= 0.02)
  )
```

Now, I will run the same models, using the placebo indicator.

``` r
placebo_reg <-
  as.formula(
    math_white_non_partic ~
      placebo*post2013 +
      lag(math_non_prof_rate) +
      per_fewer_3yrs_exp +
      per_mas_plus +
      per_frpl +
      per_lep
  )

placebo_fe_fit <-
  plm(placebo_reg,
      data = ny_panel,
      index = c("entity_cd", "year"),
      model = "within",
      effect = "twoway")

lmtest::coeftest(placebo_fe_fit, 
                 vcov = vcovHC(placebo_fe_fit,
                               type = "HC1"))
```

    ## 
    ## t test of coefficients:
    ## 
    ##                           Estimate Std. Error t value  Pr(>|t|)    
    ## lag(math_non_prof_rate)  0.1592429  0.0186390  8.5435 < 2.2e-16 ***
    ## per_fewer_3yrs_exp      -0.1289651  0.0264837 -4.8696 1.134e-06 ***
    ## per_mas_plus             0.1123692  0.0232965  4.8234 1.430e-06 ***
    ## per_frpl                 0.2096039  0.0172547 12.1476 < 2.2e-16 ***
    ## per_lep                  0.1450787  0.0955327  1.5186    0.1289    
    ## placebo:post2013         0.0058959  0.0076758  0.7681    0.4424    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Violà. The interaction is no longer significant, the point estimate
basically zero, but not precisely estimated.
