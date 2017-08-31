
# A/B Testing
#### P7:  Udacity Data Analyst Nanodegree
#### Author:  Whitney King (with excerpts from Udacity project template) 
#### Date:  8/29/2017


```R
# Load packages and settings
install.packages('ggplot2', repos = "http://cran.us.r-project.org")
library(ggplot2)

install.packages('rmarkdown', repos = "http://cran.us.r-project.org")
library(rmarkdown)

options("scipen"=100, "digits"=4)
```

    Installing package into 'C:/Users/dooki/Documents/R/win-library/3.3'
    (as 'lib' is unspecified)
    

    package 'ggplot2' successfully unpacked and MD5 sums checked
    
    The downloaded binary packages are in
    	C:\Users\dooki\AppData\Local\Temp\RtmpmezPRw\downloaded_packages
    

    Warning message:
    "package 'ggplot2' was built under R version 3.3.3"Installing package into 'C:/Users/dooki/Documents/R/win-library/3.3'
    (as 'lib' is unspecified)
    

    package 'rmarkdown' successfully unpacked and MD5 sums checked
    
    The downloaded binary packages are in
    	C:\Users\dooki\AppData\Local\Temp\RtmpmezPRw\downloaded_packages
    

    Warning message:
    "package 'rmarkdown' was built under R version 3.3.3"

## Experiment Design

### Overview

When this experiment was conducted, Udacity courses offer two options to visitors on the homepage: 

1. Start Free Trial
  * If a student clicks *start free trial*, they will be prompted to enter payment information. Once entered, they will be enrolled into a free trial of their course program which will provide them full access to the course materials for 14 days, after which point they will be billed for the tuition if they don't opt to cancel the trial.
2. Access Course Materials
  * If a student clicks *access course materials*, they will be able to immediately view the course videos and take the quizzes for free, however they will not be given coaching support, a verified certificate, and will not receive feedback on their final projects.

In the experiment, Udacity tested the following change where if the student clicked *start free trial*, they were asked how much time they had available to devote to the course:

1. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. 
2. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. 
  * At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. 

### Hypothesis

Hypothesis testing should be done to determine the probability of the experiment results occurring by chance. A standard null hypothesis would be that there is no significant difference between the results of the control and experiment groups.

The hypothesis was that this might set clearer expectations for students upfront, thus:

1. Reducing the number of frustrated students who left the free trial because they didn't have enough time
2. Without significantly reducing the number of students to continue past the free trial and eventually complete the course. 

If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.


### Metric Choice

#### Invariant Metrics

 * **Number of Cookies**: Number of unique cookies to view course overview page
  * The goal is to split cookies evenly between the control and experiment groups, and cannot be an evaluation metric
  * We don't expect to see a lot of variation between the two for this number, so it is an invariant metric


 * **Number of Clicks**: Number of unique cookies to click the *Start Free Trial* button (which happens before the free trial screener is triggered)
  * The scope of the experiment begins after the user has clicked the *Start Free Trial* button
  * The number of unique cookies that clicked should be relatively evenly distributed between the control and experiment groups, so is an invariant metric and cannot be an evaluation metric


 * **Click-Through-Probability**: Number of unique cookies to click the *Start Free Trial* button divided by number of unique cookies to view the course overview page
  * Click-Through-Probability should have slight variation between the control and experiment groups, and cannot be an evaluation metric
  * We don't expect to see a lot of variation between the two for this number, so it is an invariant metric

#### Evaluation Metrics

 * **Gross Conversion**: Number of user-ids to complete checkout and enroll in free trial divided by number of unique cookies to click "Start Free Trial" button
  * This metric will tell us if bullet point 1 of our hypothesis holds up (reduce number of students that leave the free trial due to lack of time to complete the coursework)
  * If this part of the hypothesis is true, we should be able to measure a statistically significant reduction in the Gross Conversion of the *experiment* group
  
  
 * **Net Conversion**: Number of user-ids to remain enrolled past the 14-day boundary divided by the number of unique cookies to click the "Start Free Trial" button
  * This metric will tell us if bullet point 2 of our hypothesis holds up (does not significantly reduce the number of students that continue past the free trial and complete the course)
  * If this part of the hypothesis is true, we should *not* be able to measure a statistically significant reduction of Net Conversion in the experiment group (i.e. it can be the same or higher than the control group)

#### Unused Metrics

 * **Number of user-ids**: Number of users who enroll in the free trial
  * This could have been chosen as an evaluation metric, however in this situation Gross Conversion was a better measure for the goals of this A/B test
  
  
 * **Retention**: Number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of users to enroll in the free trial
   * This number uses different units for the analysis and diversion, which can lead to inaccuracies
   * The number is expected to change, and wouldn't be an invariant metric
   * Gross Conversion and Net Conversion already provide us the framework we need for testing our hypothesis
   * Getting enough data to reliably measure retention can be a very time intensive process, and doesn't lend itself well to being chosen as an evaluation metric


### Measuring Standard Deviation

For each evaluation metric, standard deviation should be calculated. **Gross Conversion** and **Net Conversion** are measured as probabilities, so we can expect a normal, binomial distribution if the sample size is sufficient. The [baseline values](https://docs.google.com/spreadsheets/d/1MYNUtC47Pg8hdoCjOXaHqF-thheGpUshrFA21BAJnNc/edit#gid=0) are as follows, and we can guesstimate we'll need about 5000 cookies per day in each group for the experiment:



```R
sample_size <- 5000                    # Size of total sample of cookie visits being used in A/B Test
cookies.view <- 40000                  # Unique cookies to view page per day
cookies.click <- 3200                  # Unique cookies to click "Start free trial" per day
enrollments <- 660                     # Enrollments per day
click_thru_prob <- .08                 # Click-through-probability on "Start free trial"
enroll_prob_with_click <- .20625       # Probability of enrolling, given click (base conversion rate)
pay_prob_with_enroll <- .53            # Probability of payment, given enroll
pay_prob_with_click <- .1093125        # Probability of payment, given click

click_sample <- sample_size * click_thru_prob    # N = 400
```

To estimate the standard deviation of the evaluation metrics with an assumed binomial distribution, we can use the formula:

![SD = sqrt(p(1-p)/N)](https://latex.codecogs.com/png.latex?SD%20%3D%20%5Csqrt%7B%5Cfrac%7Bp*%281-p%29%7D%7BN%20%7D%7D)


```R
gross_converison.SD <- round(sqrt(enroll_prob_with_click*(1-enroll_prob_with_click)/click_sample), digits = 4)
net_conversion.SD <- round(sqrt(pay_prob_with_click*(1-pay_prob_with_click)/click_sample), digits = 4)

cat('Gross Conversion Standard Deviation:  ', gross_converison.SD, '\n')
cat('Net Conversion Standard Deviation:  ', net_conversion.SD)
```

    Gross Conversion Standard Deviation:   0.0202 
    Net Conversion Standard Deviation:   0.0156

**Gross Conversion** and **Net Conversion** are both calculated using the **Number of Cookies** metric as the unit of diversion. This metric is also the unit of analysis for both gross and net conversion, so estimates should be fairly consistent with empirical variability and can be used for further investigation down the line.

### Sizing

#### Number of Samples vs. Power
Since **Gross Conversion** and **Net Conversion** are already correlated, Bonferroni Correction was not used in the calculations to avoid increasing the number of potential false positives.

Using the sample size calculator in [Evan's Awesome A/B Tools](https://www.evanmiller.org/ab-testing/sample-size.html), we can determine the following sample sizes for **Gross Conversion** and **Net Conversion**:

* **Gross Conversion**
  * Baseline:  .20625
  * dMin: .01
  * α: .05
  * 1−β: .8
  * **Sample Size**: 25,835
  
  
* **Net Conversion**:
  * Baseline:  .1093125
  * dMin: .0075
  * α: .05
  * 1−β: .8
  * **Sample Size**: 27,413
  
Since **Net Conversion** requires the larger sample size, we'll take a closer look at that one. We can use the previously calculated click-through-probability to get the total number of pageviews we should have for each group:



```R
gross_conversion.size <- 25835
net_conversion.size <- 27413

pageviews.group <- net_conversion.size / click_thru_prob
cat('Pageviews Per Group:  ', pageviews.group, '\n')

pageviews.total = pageviews.group * 2
cat('Total Pageviews:  ', pageviews.total, '\n')
```

    Pageviews Per Group:   342663 
    Total Pageviews:   685325 
    

#### Duration vs. Exposure

Given that there are 40,000 view cookies each day, and we'll require 685,325 pageviews the two groups in the experiment, it can be expected that it will take a period to collect the data required for the two experiment groups. When the impact of the parameters involved in this experiment are considered, the risk to the user is minimal and it shouldn't be a deterrent to any current students. The data being evaluated doesn't pose any overt risks to the users, and doesn't gather sensitive information about them.

Diverting most of traffic to the experiment should be fine, but it's a best practice to not divert all traffic. Since the data would take minimally a few weeks to collect, a good goal would be in keeping the data collection time under a month. With these considerations, I would divert roughly 60% of site traffic to the experiment groups.



```R
traffic.percent <- .6
traffic.divert <- cookies.view * traffic.percent
traffic.days <- round(pageviews.total / traffic.divert, digits=0)

cat(traffic.percent*100, '% of Traffic:', traffic.divert, '\n')
cat('Days of Collection:  ', traffic.days)
```

    60 % of Traffic: 24000 
    Days of Collection:   29

## Experiment Analysis

### Sanity Checks

The data for the experiment was obtained from the [spreadsheet provided by Udacity](https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0). To begin, we'll calculate totals for cookies views, clicks, and click-through-probability. It's expected that the experiment and control groups should be relatively even. Then, we'll calculate the confidence interval, and upper and lower bounds for each invariant metric, and whether those values pass the sanity checks:


```R
# Load Data
control <- read.csv("control.csv", header = T, check.names = F,
                    na.strings = c(""))

experiment <- read.csv("experiment.csv", header = T, check.names = F,
                    na.strings = c(""))

p <- .5
z_score <- 1.96

control.pageviews <- sum(control$Pageviews)
experiment.pageviews <- sum(experiment$Pageviews)
cookies.total <- control.pageviews + experiment.pageviews
cookies.prob <- control.pageviews / cookies.total
cookies.sigma <- sqrt((p*(1-p))/(control.pageviews+experiment.pageviews))
cookies.me <- z_score * cookies.sigma
cookies.lower <- c(p-cookies.me)
cookies.upper <- c(p+cookies.me)
cookies.pass <- cookies.lower < cookies.prob && cookies.prob < cookies.upper

control.clicks <- sum(control$Clicks)
experiment.clicks <- sum(experiment$Clicks)
total.clicks <- control.clicks + experiment.clicks
click.prob <- control.clicks / total.clicks
click.sigma <- sqrt((p*(1-p))/(control.clicks + experiment.clicks))
click.me <- z_score * click.sigma
click.lower <- c(p-click.me)
click.upper <- c(p+click.me)
click.pass <- click.lower < click.prob && click.prob < click.upper

control.ctp <- round(control.clicks / control.pageviews, digits=4)
experiment.ctp <- round(experiment.clicks / experiment.pageviews, digits=4)
ctp.pool <- (control.ctp + experiment.ctp) / 2
ctp.diff <- experiment.ctp - control.ctp
ctp.sigma <- sqrt((ctp.pool*(1-ctp.pool))/control.pageviews)
ctp.me <- z_score * ctp.sigma
ctp.lower <- c(control.ctp-ctp.me)
ctp.upper <- c(control.ctp+ctp.me)
ctp.pass <- ctp.lower < ctp.pool && ctp.pool < ctp.upper

cat('Cookie Views: \n',
    '    Control Cookies:  ', control.pageviews, '\n',
    '    Experiment Cookies:  ', experiment.pageviews, '\n',
    '    Total Cookies:  ', cookies.total, '\n',
    '    Expected Probability:  0.5', '\n',
    '    -------------------------------', '\n',
    '    Observed Probability:  ', cookies.prob, '\n',
    '    Sigma:', cookies.sigma, '\n',
    '    ME:', cookies.me, '\n',
    '    CI:', '[', cookies.lower, ',', cookies.upper, ']', '\n',
    '    Pass:', cookies.pass, '\n\n\n')

cat('Clicks: \n',
    '    Control Clicks:  ', control.clicks, '\n', 
    '    Experiment Clicks:  ', experiment.clicks, '\n',
    '    Total Clicks:  ', total.clicks, '\n',
    '    Expected Probability:  0.5', '\n',
    '    -------------------------------', '\n',
    '    Observed Probability:  ', click.prob, '\n',
    '    Sigma:', click.sigma, '\n',
    '    ME:', click.me, '\n',
    '    CI:', '[', click.lower, ',', click.upper, ']', '\n',
    '    Pass:', click.pass, '\n\n\n')

cat('Click-Through-Probability: \n',
    '    Control CTP:  ', control.ctp, '\n',
    '    Experiment CTP:  ', experiment.ctp, '\n',
    '    Expected CTP:  0.0821', '\n',
    '    -------------------------------', '\n',
    '    Pooled CTP:  ', ctp.pool, '\n',
    '    CTP Difference:  ', ctp.diff, '\n',
    '    Sigma:  ', ctp.sigma, '\n',
    '    ME:  ', ctp.me, '\n',
    '    CI:', '[', ctp.lower, ',', ctp.upper, ']', '\n',
    '    Pass:', ctp.pass)
```

    Cookie Views: 
         Control Cookies:   345543 
         Experiment Cookies:   344660 
         Total Cookies:   690203 
         Expected Probability:  0.5 
         ------------------------------- 
         Observed Probability:   0.5006 
         Sigma: 0.0006018 
         ME: 0.00118 
         CI: [ 0.4988 , 0.5012 ] 
         Pass: TRUE 
    
    
    Clicks: 
         Control Clicks:   28378 
         Experiment Clicks:   28325 
         Total Clicks:   56703 
         Expected Probability:  0.5 
         ------------------------------- 
         Observed Probability:   0.5005 
         Sigma: 0.0021 
         ME: 0.004116 
         CI: [ 0.4959 , 0.5041 ] 
         Pass: TRUE 
    
    
    Click-Through-Probability: 
         Control CTP:   0.0821 
         Experiment CTP:   0.0822 
         Expected CTP:  0.0821 
         ------------------------------- 
         Pooled CTP:   0.08215 
         CTP Difference:   0.0001 
         Sigma:   0.0004671 
         ME:   0.0009156 
         CI: [ 0.08118 , 0.08302 ] 
         Pass: TRUE

After running the sanity checks, all three invariant metrics that were chosen pass, and are good picks.

### Result Analysis

#### Effect Size Tests

Now that the experiment has passed initial sanity checks, it's time to test the statistical and practical significance of the evaluation metrics using only complete rows of data. The evaluation metrics are considered statistically significant if 0 is not within the bounds of the confidence interval, which indicates there was a change between the groups.


```R
control.clicks <- sum(control$Clicks[1:23])
experiment.clicks <- sum(experiment$Clicks[1:23])
total.clicks <- control.clicks + experiment.clicks

control.enrollments <- sum(control$Enrollments[1:23])
experiment.enrollments <- sum(experiment$Enrollments[1:23])
total.enrollments <- control.enrollments + experiment.enrollments

control.payments <- sum(control$Payments[1:23])
experiment.payments <- sum(experiment$Payments[1:23])
total.payments <- control.payments + experiment.payments

gross_conversion.dmin <- .01
gross_conversion.pool <- total.enrollments / total.clicks
gross_conversion.diff <- (experiment.enrollments/experiment.clicks) - (control.enrollments/control.clicks)
gross_conversion.sigma <- sqrt((gross_conversion.pool*(1-gross_conversion.pool))*((1/control.clicks)+(1/experiment.clicks)))
gross_conversion.me <- z_score * gross_conversion.sigma
gross_conversion.lower <- c(gross_conversion.diff-gross_conversion.me)
gross_conversion.upper <- c(gross_conversion.diff+gross_conversion.me)

net_conversion.dmin <- .0075
net_conversion.pool <- total.payments / total.clicks
net_conversion.diff <- (experiment.payments/experiment.clicks) -(control.payments/control.clicks)
net_conversion.sigma <- sqrt((net_conversion.pool*(1-net_conversion.pool))*((1/control.clicks)+(1/experiment.clicks)))
net_conversion.me <- z_score * net_conversion.sigma
net_conversion.lower <- c(net_conversion.diff-net_conversion.me)
net_conversion.upper <- c(net_conversion.diff+net_conversion.me)



cat('Clicks', '\n',
    '    Control:', control.clicks, '\n',
    '    Experiment:', experiment.clicks, '\n',
    '    Total:', total.clicks, '\n\n\n')

cat('Enrollments:', '\n',
    '    Control:', control.enrollments, '\n',
    '    Experiment:', experiment.enrollments, '\n',
    '    Total:', total.enrollments, '\n\n\n')

cat('Payments:', '\n',
    '    Control:', control.payments, '\n',
    '    Experiment:', experiment.payments, '\n',
    '    Total:', total.payments, '\n\n\n')

cat('---------------------------------------------------', '\n')
cat('Gross Conversion: \n',
    '    Pooled:', gross_conversion.pool, '\n',
    '    Difference:', gross_conversion.diff, '\n',
    '    Sigma:', gross_conversion.sigma, '\n',
    '    ME:', gross_conversion.me, '\n',
    '    Practical Boundary:', gross_conversion.dmin, '\n',
    '    CI:', '[', gross_conversion.lower, ',', gross_conversion.upper, ']', '\n',
    '    Statistically Significant?:   YES', '\n',
    '    Practicall Significant?:   YES', '\n\n\n')

#prop.test(x=c(total.enrollments), n=c(total.clicks))

cat('Net Conversion: \n',
    '    Pooled:', net_conversion.pool, '\n',
    '    Difference:', net_conversion.diff, '\n',
    '    Sigma:', net_conversion.sigma, '\n',
    '    ME:', net_conversion.me, '\n',
    '    Practical Boundary:', net_conversion.dmin, '\n',
    '    CI:', '[', net_conversion.lower, ',', net_conversion.upper, ']', '\n',
    '    Statistically Significant?:   NO', '\n',
    '    Practicall Significant?:   NO', '\n\n\n')

#prop.test(x=c(total.payments), n=c(total.clicks))
```

    Clicks 
         Control: 17293 
         Experiment: 17260 
         Total: 34553 
    
    
    Enrollments: 
         Control: 3785 
         Experiment: 3423 
         Total: 7208 
    
    
    Payments: 
         Control: 2033 
         Experiment: 1945 
         Total: 3978 
    
    
    --------------------------------------------------- 
    Gross Conversion: 
         Pooled: 0.2086 
         Difference: -0.02055 
         Sigma: 0.004372 
         ME: 0.008568 
         Practical Boundary: 0.01 
         CI: [ -0.02912 , -0.01199 ] 
         Statistically Significant?:   YES 
         Practicall Significant?:   YES 
    
    
    Net Conversion: 
         Pooled: 0.1151 
         Difference: -0.004874 
         Sigma: 0.003434 
         ME: 0.006731 
         Practical Boundary: 0.0075 
         CI: [ -0.0116 , 0.001857 ] 
         Statistically Significant?:   NO 
         Practicall Significant?:   NO 
    
    
    

* **Gross Conversion** *is* statistically significant since the interval does *not* include 0, and it is practicially significant because it does not pass the practical significance boundaries. This metric passes.

* **Net Conversion** *is not* statistically *or* pracitically significant since the interval includes 0. This metrics doesn't pass. The confidence interval is beyond the boundary of practical significance, so we could interpret this data incorrectly. Due to these factors, this metrics doesn't pass.



#### Sign Tests

Two-tailed t-tests were run using the [Sign and Binomical Test](http://graphpad.com/quickcalcs/binomial2/) tool hosted by GraphPad.


```R
cat('Gross Conversion: \n',
    '    Successes:  4', '\n',
    '    # Trials:  23', '\n',
    '    Probability:  .5', '\n',
    '    p-value:  .0026', '\n',
    '    Statistically Significant?:  YES', '\n\n\n')

cat('Net Conversion: \n',
    '    Successes:  10', '\n',
    '    # Trials:  23', '\n',
    '    Probability:  .5', '\n',
    '    p-value:  .6776', '\n',
    '    Statistically Significant?:  NO', '\n\n\n')
```

    Gross Conversion: 
         Successes:  4 
         # Trials:  23 
         Probability:  .5 
         p-value:  .0026 
         Statistically Significant?:  YES 
    
    
    Net Conversion: 
         Successes:  10 
         # Trials:  23 
         Probability:  .5 
         p-value:  .6776 
         Statistically Significant?:  NO 
    
    
    

#### Summary

This experiment was divided into two groups: control and experiment. The control group was enrolled in a free trial version of their chosen course after clicking *start free trial*. Upon registration, they were asked how much time they expect to be able to devote to the learning each week. The control group was not taken through this trial flow. A null hypothesis was formed stating there was no significant difference in the evaluation metrics of the two different groups, and a practical significant was set for each of these metrics.

There were a a handful of metrics to choose from, some of which were suited as invariant mertrics, and others more suited towards evaluation metrics. The invariant metrics chosen were **Number of Cookies**, **Number of Clicks**, and **Click-Through-Probability**. The evaluation metrics chosen were **Gross Conversion**, and **Net Conversion**. Due to evaluation metrics already being correlated, the Bonferroni Correction was not used.

After sanitity checks were ran, and the results of the experiement were analysed, the experiment group experienced a statistically significant decrease in **Gross Conversion** when compared to the control group at a 95% CI, and aligned with the hypothesis. However, in the case **Net Conversion**, the results were *not* found to be statistically significant, and the confidence interval exceeded the negative end of the practical significance boundary. Both effect and size tests returned the same findings.

### Recommendation

The experiments design was built around gaining an understanding of if filterinf students using started study time commitment would improve the overall student experience, without negatively impacting the number of students that continue through the trial period and into full enrollment. We were able to determine there were both statistically and practically significant differences between the two groups when it came to **Gross Conversion**, however there were no significant differences with **Net Conversion**. What this means is that we saw a *decrease* in enrollment that wasn't accompanied by and increase in conversions to enrollment from the trial period. Given this fact, the reccomendation based on these findings would be to not launch this change, and to instead look into other options.

## Follow-Up Experiment

The previous experiment failed given the goals stated at the beginning. Considering the way these results went, there are other options based on the idea of converting users to trial, and trials to enrollments that could make potentially interesting follow up experiments. Focusing the flow around volunteered information is maybe not the best way to select which users are shuffered into which flow, since sometimes people can be extremely off with their estimatations around things like dedication to study time.

Perhaps trying a number of different trial periods (such as a month long free trial) versus the standard 2-week free trial would help calm insecurities students may have about not having enough time to really get a feel for how the program will go, and if it will be a good fit for them in the long run. Instead of having a system built around user inputting information, I'd simply have the flow divert traffic to the experiment samples from any users that clicked *start free trial*.

* **Null Hypothesis**: having access to a long free trial period will not increase the number of students that enroll after the free trial period
* **Unit of Diversion**: user-id, since we'll be tracking a change that occurs on a per user base upon enrollment
* **Invariant Metrics**:  Number of Cookies, Number of Clicks
* **Evaluation Metrics**: Gross Conversion, Net Conversion, Retention

If we see statistically significant change is seen in the evaluation metrics, the experiment could be launched.
