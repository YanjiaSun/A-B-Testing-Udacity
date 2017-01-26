# A/B Testing Udacity's Free Trial Screener

## Experiment Overview
>At the time of this experiment, Udacity courses currently have two options on the home page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

>In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. [This screenshot](https://drive.google.com/file/d/0ByAfiG8HpNUMakVrS0s4cGN2TjQ/view) shows what the experiment looks like.

>The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

>The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## Experiment Design

### All Metrics

* **Number of cookies**: Number of unique cookies to view course overview page.
* **Number of clicks**: Number of unique cookies to click the "Start Free Trial" button (which happens before the free trial screener is triggered).
* **Number of user-ids**: Number of users who enroll in the free trial.
* **Click-through-probability**: Number of unique cookies to click the "Start Free Trial" button divided by number of unique cookies to view the course overview page.
* **Gross conversion**: Number of user-ids to complete checkout and enroll in free trial divided by number of unique cookies to click "Start Free Trial" button.
* **Retention**: Number of user-ids to remain enrolled past the 14-day boundary (and thus make atleast one payment) divided by number of users to enroll in the free trial.
* **Net conversion**: Number of user-ids to remain enrolled past the 14-day boundary divided by the number of unique cookies to click the "Start Free Trial" button.

### Metric Choice

#### Invariant metrics:
  1. **Number of cookies**: This is the unit of diversion for the A/B test. Since the visit to the course overview page occurs before the experiment, this metric is independent of the experiment and thus should be evenly distributed between the control and experiment group.

  2. **Number of clicks**: Since the users click before the free trial screener pops up, this metric is also independent of the experiment, and should be evenly distributed between the control and experiment groups.

  3. **Click-through-probability**: This is simply a ratio of the above two metrics, and since they are both independent, this too is independent, and should be evenly distributed between the control and experiment groups.


#### Evaluation metrics:

  1. **Gross conversion**: Ideally, we hypothesize and want the screener to produce a lower conversion rate (because the people who were deterred by the screener are unlikely to make a payment and complete the course, hence freeing up coaching resources), but not at the cost of a lower net conversion rate.

  2. **Net conversion**: While wanting a lower gross conversion rate, we do not want the screener to *significantly reduce* the number of students to continue past the free trial and eventually complete the course, which means the net conversation should either remain the same (within boundaries), or increase. While a significantly higher net conversion might be unexpected (and more tests must definitely be done to check), it is quite possible. For instance, students who are willing to put more than 5 hours per week might appreciate the honesty conveyed by the screener, and as a result trust Udacity and give the course a shot - the same students who might have left within the 14 days had the message not been there.


#### Unused metrics:

  1. **Number of user-ids**: This is not a good invariant metric because it is expected to reduce as a result of the experiment. It is an applicable evaluation metric because it would record the number of students continue past the free trial, but it's not the best metric because it's not normalized (and Gross Conversion would be better).

  2. **Retention**: This one is more tricky. It's not an invariant metric because it is expected to change, but it's not a good evaluation metric either because the magnitude of change is dependent on the variant number of user-ids (and hence related to Gross conversion). If an experiment has a lower number of user-ids but the same retention rate as the control, that means the *total number* of users passing the free trial is more in the control. Therefore, a higher retention rate could mean that the *a similar number* of users are passing the free trial, and hence is not precise in its meaning. A high retention rate could lead to both a lower, similar, or higher net conversion, depending on the gross conversion. Another way of looking at is that retention is simply net conversion divided by gross conversion, and hence does not provide any meaningful information.

#### What to Look For

In order to launch this experiment, the gross conversion must have a statistically significant and practically significant decrease *and* the net conversion must *not* significantly decrease (statistically and practically). That is, the net conversion can either remain the same as the control (within the confidence interval), or increase (significantly or not). If there is a significant increase in net conversion, that's good, but it was not the intended effect and so more tests must be done to deduce what's causing the change.


### Measuring Variability

Now let's calculate the Standard Deviation for both our Evaluation Metrics. Since Gross Conversion and Net Conversion are both probabilities, we can assume a binomial distribution, which will take on a normal distribution for a large enough sample size.

The Udacity baseline values can be [found here](https://docs.google.com/spreadsheets/d/1MYNUtC47Pg8hdoCjOXaHqF-thheGpUshrFA21BAJnNc/edit#gid=0).

![SE](http://www.sciweavers.org/upload/Tex2Img_1485172999/render.png "Standard Error Formula")

Using the above formula and a sample size of 5000 cookies visiting the course overview page and using the ratio shown in the baseline values, the number of cookies we expect to click on the "Start Free Trial" would be:

```
N = 5000 * 3200/40000
N = 400
```

Then, the Standard Errors for 5000 pageviews will be:

```
Gross Conversion SE...... 0.0202

Net Conversion SE........ 0.0156
```

Since the denominators of both the Evaluation Metrics are the same as the unit of diversion (i.e. cookie), the analytical variability will be similar to the empirical variability, but let's calculate the latter as well.


### Sizing

#### Number of Samples vs Power

Since there are multiple tests being done, the chance of a rare event in the confidence interval increases. The Bonferroni correction compensates for that increase, but at the cost of increasing the probability of producing false positives (reducing statistical power), and can be conservative if there are large number of tests and/or the test statistics are positively correlated.

I will not be using the Bonferroni correction for this experiment.

Using [this online calculator](http://www.evanmiller.org/ab-testing/sample-size.html), we can calculate how large our sample size needs to be:

```
Gross Conversion:
    Base conversion rate = 20.625%
    Practical significance boundary = 1%
    ...
    Sample size: 25,835

Net Conversion:
    Base conversion rate = 10.93125%
    Practical significance boundary = 0.75%
    ...
    Sample size: 27,413
```

Net Conversion's sample size covers Gross Conversion so let's select that. We know the click-through-probability on the "Start Free Trial" button is `0.08`, so now let's calculate the total number of pageviews per group, and then double that to get the total pageviews for both the control group and experiment group:

```
Pageviews (per group) = 27413 / 0.08
                      = 342,662.5

Total pageviews = 342662.5 * 2
                = 685,325
```

#### Duration vs Exposure

There are 40,000 unique cookies that view the course overview page each day, and our A/B Test needs a total of 685,325 pageviews. Since this experiment does not pose much of a risk, I'd divert 75% of the daily traffic (30,000 cookies) towards the experiment. This means the experiment will last roughly 23 days.

The experiment does not impact existing users since they will not click the "Start Free Trial" button, and this is very important when determining the risk level of the experiment. Next, even within new visitors to the website (or repeating visitors who haven't signed up), it will not pose as a deterrence to motivated students with a pre-existing desire to complete the course (and hence put in a large number of hours per week). It only affects the decision process of the uncertain people clicking the "Start Free Trial" button, and even then, they might appreciate the honesty provided by the screener, and are unlikely to be offended by the experiment. The experiment does not pose a physical, psychological, emotional, social or financial risk beyond those that they would encounter in normal daily life, and neither does it deal with sensitive data. Thus, the experiment does not exceed minimal risk. For these reasons, the experiment can be run over a large percentage of traffic. That being said, it's pragmatic not to run over all 100% of the traffic to avoid outlier cases and any potential bugs.


## Experiment Analysis

### Sanity Checks

The first part of the analysis is to see if our invariant metrics are unchanged for both the control and experiment groups. The data for the control and experiment can be found [here](https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0).

Getting the totals:

```
Number of cookies:
    Total control pageviews........ 345543
    Total experiment pageviews..... 344660
    Total pageviews................ 690203
    Expected probability...........    0.5

Number of clicks:
    Total control clicks........... 28378
    Total experiment clicks........ 28325
    Total clicks................... 56703
    Expected probability...........   0.5

Click-through-probability:
    Observed CTP (control)......... 0.082126
    Observed CTP (experiment)...... 0.082182
    Pooled CTP..................... 0.082154
```

[Click here](https://i.imgsafe.org/8ebd8ef747.jpg) to see a sample calculation for finding the 95% confidence intervals.

```
Number of cookies:
    Confidence interval............. [0.4988, 0.5012]
    Observed value.................. 0.5006
    Passed?......................... YES

Number of clicks:
    Confidence interval............. [0.4959, 0.5041]
    Observed value.................. 0.5005
    Passed?......................... YES

Click-through-probability:
    Confidence interval (of diff.).. [-0.001296, 0.001296]
    Observed difference............. 0.00005663
    Passed?......................... YES
```

All three Sanity Checks have passed!

### Result Analysis

#### Effect Size Test

Since our experiment has passed the sanity checks, let's see if the evaluation metrics are statistically and practically significant.

Getting the totals:

```
Gross Conversion:
    Total clicks (control).......... 17293
    Total clicks (experiment)....... 17260
    Total enrollments (control).....  3785
    Total enrollments (experiment)..  3423

Net Conversion:
    Total clicks (control).......... 17293
    Total clicks (experiment)....... 17260
    Total payments (control)........  2033
    Total payments (experiment).....  1945
```

[Click here](https://i.imgsafe.org/8ec03781b5.jpg) to see a sample calculation for finding the 95% confidence intervals.

```
Gross Conversion:
    GC rate (control)............... 0.2189
    GC rate (experiment)............ 0.1983
    GC rate diff. (exp. - control).. -0.02055
    Pooled GC rate.................. 0.2086
    Pooled standard deviation....... 0.004372
    ...
    Confidence interval............. [-0.02912, -0.01199]
    Practical significance boundary. 0.01
    ...
    ...
    Statistically significant?...... YES
    Practically significant?........ YES


Net Conversion:
    NC rate (control)............... 0.1176
    NC rate (experiment)............ 0.1127
    NC rate diff. (exp. - control).. -0.004874
    Pooled NC rate.................. 0.1151
    Pooled standard deviation....... 0.003434
    ...
    Confidence interval............. [-0.01160, 0.001857]
    Practical significance boundary. 0.0075
    ...
    ...
    Statistically significant?...... NO (which is okay)
    Practically significant?........ NO
```

For Gross Conversion, we want a statistically significant and practically significant *decrease*, which is what the calculations show. Zero is not a part of the confidence interval, and the upper bound is past the practical significance boundary. So Gross Conversion has PASSED.

For Net Conversion, however, the situation is a bit tricky. We wanted the experiment to *not have* a significant decrease, which means that its definition of practical significance includes staying the same (i.e. no statistically significant difference in net conversion), or an increase. The calculations show that there is no statistical significant difference in net conversion, however the lower bound of the 95% confidence interval is past the boundary for practical significance. This means that any interpretation of the data will be uncertain, and we do not have enough statistical power to draw a strong conclusion.

#### Sign Tests

The two-tailed p-values were calculated using [this online calculator](http://graphpad.com/quickcalcs/binomial2/)

```
Gross Conversion:
    Number of successes.............  4
    Number of trials................ 23
    Probability..................... 0.5
    Two-tailed p-value.............. 0.0026
    ...
    Statistically significant?...... YES

Net Conversion:
    Number of successes............. 10
    Number of trials................ 23
    Probability..................... 0.5
    Two-tailed p-value.............. 0.6776
    ...
    Statistically significant?...... NO
```

#### Summary

The Bonferroni correction was not used because we want *both* the evaluation metrics to pass in order to launch. The Bonferroni correction is useful in reducing Type I errors (i.e. when any metric can pass in order to launch), and not necessarily effective in reducing Type II errors (what we're dealing with).

Both the Effect Size Tests and Sign Tests have produced congruent results, both deeming Gross Conversion to be statistically significant and Net Conversion statistically insignificant. Additionally, the Effect Size Tests have also shown that Gross Conversion is practically significant, but Net Conversion is not because the lower bound of the 95% confidence interval is past the boundary for practical significance.

### Recommendation

The experiment has caused Gross Conversion to significantly decrease both statistically and practically, which means it has succeeded in its first aspect which was to warn students about the course load so that they may reconsider joining the free trial, thereby freeing up coaching resources to students who are more likely to pay for the course. However, we are uncertain whether the experiment has had a significant decrease in Net Conversion or not, and for that reason, I recommend to **NOT LAUNCH** this experiment now, and instead create additional tests to determine whether Net Conversion has significantly decreased or not. Since the confidence interval of Net Conversion does include the negative of the practical significance boundary, launching this experiment might cause a decrease in revenue from students paying for courses, and thus too risky a change. If it is not feasible to run more experiments and a decision must be made, I'd recommend seeking a business analyst to check whether the cost of the possible chance of lower number student payments is made up by the decrease in expenditure for the coaching resources.

## Follow-Up Experiment

It's quite difficult to come up with a unique and transformative experiment for Udacity considering all the good ones have already been done. I particularly like the P0 mini-project idea because, as a student, I adore and applaud the effort put into the Project Review. I think it's what makes Udacity stand out, and if someone entering the free trial has the opportunity to submit a project and experience this, it will increase their chances of paying and completing the course. Along with that, the financial incentives should be effective too.

My idea for an experiment would be to provide an option of attending an online meeting between a representative of any corresponding hiring partner and the student enrolling. With online learning, particularly for those who do not have access to UConnect or cannot join the Nanodegree Plus, I think it's important to establish a social connection to provide the emotional motivation needed to complete the program. By making the meeting with a hiring partner, Udacity can establish a direct line between potential employee and employer right from the go. This should hopefully inspire students, especially entry-level ones and those looking for a career change, to complete the course.     

**Hypothesis**: The Retention will increase because the meeting will show that program is worth the effort and inspire students to complete the course.

**Unit of Diversion**: User-Id. This change only impacts those who have enrolled in the program and subsequently made an account.

**Invariant Metrics**: Number of User-Ids. This should not change over both the control and experiment groups as there is nothing that is affecting their enrollment into the program.

**Evaluation Metrics**: Retention. The meeting can hopefully increase the number of users who continue past the 14-day free trial and make a payment, provided that they have entered the free trial period.

* *The practical significance boundary should be higher than the experiment that was just analyzed because the follow-up experiment will be more expensive and more difficult to organize. In case it is unfeasible business- or logistics-wise, I would change online meeting to personalized e-mails.*
