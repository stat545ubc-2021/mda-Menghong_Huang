Mini Data-Analysis Deliverable 3
================

# Welcome to your last milestone in your mini data analysis project!

In Milestone 1, you explored your data and came up with research
questions. In Milestone 2, you obtained some results by making summary
tables and graphs.

In this (3rd) milestone, you’ll be sharpening some of the results you
obtained from your previous milestone by:

-   Manipulating special data types in R: factors and/or dates and
    times.
-   Fitting a model object to your data, and extract a result.
-   Reading and writing data as separate files.

**NOTE**: The main purpose of the mini data analysis is to integrate
what you learn in class in an analysis. Although each milestone provides
a framework for you to conduct your analysis, it’s possible that you
might find the instructions too rigid for your data set. If this is the
case, you may deviate from the instructions – just make sure you’re
demonstrating a wide range of tools and techniques taught in this class.

## Instructions

**To complete this milestone**, edit [this very `.Rmd`
file](https://raw.githubusercontent.com/UBC-STAT/stat545.stat.ubc.ca/master/content/mini-project/mini-project-3.Rmd)
directly. Fill in the sections that are tagged with
`<!--- start your work here--->`.

**To submit this milestone**, make sure to knit this `.Rmd` file to an
`.md` file by changing the YAML output settings from
`output: html_document` to `output: github_document`. Commit and push
all of your work to your mini-analysis GitHub repository, and tag a
release on GitHub. Then, submit a link to your tagged release on canvas.

**Points**: This milestone is worth 40 points (compared to the usual 30
points): 30 for your analysis, and 10 for your entire mini-analysis
GitHub repository. Details follow.

**Research Questions**: In Milestone 2, you chose two research questions
to focus on. Wherever realistic, your work in this milestone should
relate to these research questions whenever we ask for justification
behind your work. In the case that some tasks in this milestone don’t
align well with one of your research questions, feel free to discuss
your results in the context of a different research question.

# Setup

Begin by loading your data and the tidyverse package below:

``` r
library(datateachr) # <- might contain the data you picked!
library(tidyverse)
library(broom)
library(here)
```

From Milestone 2, you chose two research questions. What were they? Put
them here.

<!-------------------------- Start your work below ---------------------------->

-   The `vancouver_tree` dataset is what I choose for all the milestones
    of my mini-data analysis project. The final two research questions
    are:

1.  *What is the relationship between the age of trees with `diameter`
    or `height_range_id`?*

2.  *How `plant_area` influence the diameter or the height of trees?*

<!----------------------------------------------------------------------------->

# Exercise 1: Special Data Types (10)

For this exercise, you’ll be choosing two of the three tasks below –
both tasks that you choose are worth 5 points each.

But first, tasks 1 and 2 below ask you to modify a plot you made in a
previous milestone. The plot you choose should involve plotting across
at least three groups (whether by facetting, or using an aesthetic like
colour). Place this plot below (you’re allowed to modify the plot if
you’d like). If you don’t have such a plot, you’ll need to make one.
Place the code for your plot below.

<!-------------------------- Start your work below ---------------------------->

-   The original plot from milestone 2 indicates the diameter range for
    each height_range_id of trees. The factor is ordered increasingly by
    height levels.

``` r
vancouver_trees %>%
  filter(plant_area==10,genus_name=="PRUNUS",diameter!=0)  %>%
  ggplot(aes(factor(height_range_id),diameter)) +
  geom_boxplot()+
  geom_jitter(alpha=0.1,colour="blue")+
  scale_y_log10()+
  xlab("Height Range Id")+
  ylab("Diameter_logScaled")
```

![](mini-project-3_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

<!----------------------------------------------------------------------------->

Now, choose two of the following tasks.

1.  Produce a new plot that reorders a factor in your original plot,
    using the `forcats` package (3 points). Then, in a sentence or two,
    briefly explain why you chose this ordering (1 point here for
    demonstrating understanding of the reordering, and 1 point for
    demonstrating some justification for the reordering, which could be
    subtle or speculative.)

2.  Produce a new plot that groups some factor levels together into an
    “other” category (or something similar), using the `forcats` package
    (3 points). Then, in a sentence or two, briefly explain why you
    chose this grouping (1 point here for demonstrating understanding of
    the grouping, and 1 point for demonstrating some justification for
    the grouping, which could be subtle or speculative.)

3.  If your data has some sort of time-based column like a date (but
    something more granular than just a year):

    1.  Make a new column that uses a function from the `lubridate` or
        `tsibble` package to modify your original time-based column. (3
        points)
        -   Note that you might first have to *make* a time-based column
            using a function like `ymd()`, but this doesn’t count.
        -   Examples of something you might do here: extract the day of
            the year from a date, or extract the weekday, or let 24
            hours elapse on your dates.
    2.  Then, in a sentence or two, explain how your new column might be
        useful in exploring a research question. (1 point for
        demonstrating understanding of the function you used, and 1
        point for your justification, which could be subtle or
        speculative).
        -   For example, you could say something like “Investigating the
            day of the week might be insightful because penguins don’t
            work on weekends, and so may respond differently”.

<!-------------------------- Start your work below ---------------------------->

**Task 1**: Reorder the `height_range_id` in decreasing order according
to the count of trees in each height range.

-   In this way, the plot can give more emphasis on the trees with most
    common heights and their corresponding diameter range. Thurs, as
    shown below, it can be seen that most of trees in height range 3 and
    2 and generally higher trees have bigger diameters. Height range
    5\~7 don’t follow this trend because of there are few trees in these
    heights. Thurs, it also prevents from drawing wrong conclusion
    causing by lack of data.

``` r
exer1_data<- vancouver_trees %>%
  filter(plant_area==10,genus_name=="PRUNUS",diameter!=0)  %>%
  mutate(height_range_id=factor(height_range_id))
#head(exer1_data)

exer1_data %>%
  mutate(height_range_id=fct_rev(fct_infreq(height_range_id))) %>%
  ggplot(aes(fct_infreq(height_range_id),diameter)) +
  geom_boxplot()+
  geom_jitter(alpha=0.1,colour="blue")+
  scale_y_log10()+
  xlab("Height Range Id")+
  ylab("Diameter_logScaled")
```

![](mini-project-3_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

<!----------------------------------------------------------------------------->
<!-------------------------- Start your work below ---------------------------->

**Task 3**: `date_planted` is a date-based column.use **lubridate**
package to convert date information to the age of trees. So that we can
get numerical data of age to further study the age-related relationship
with diameter or height.

-   Firstly, load **lubridate** package

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

-   Use the **interval**,**today** and **duration** function from
    lubridate to calculate the age based on the interval between date of
    tree planted and the date of today.

``` r
exer1_data %>%
  filter(!is.na(date_planted)) %>%
  mutate(age=floor(interval(date_planted, today())/duration(n=1, unit="years")))%>%
  head()
```

    ## # A tibble: 6 × 21
    ##   tree_id civic_number std_street   genus_name species_name cultivar_name
    ##     <dbl>        <dbl> <chr>        <chr>      <chr>        <chr>        
    ## 1  155971         1795 E 49TH AV    PRUNUS     CERASIFERA   ATROPURPUREUM
    ## 2  158745         3536 TRAFALGAR ST PRUNUS     X YEDOENSIS  AKEBONO      
    ## 3  158868         1798 E 47TH AV    PRUNUS     CERASIFERA   ATROPURPUREUM
    ## 4  160408         5667 MANSON ST    PRUNUS     CERASIFERA   ATROPURPUREUM
    ## 5  160613         2808 W 36TH AV    PRUNUS     CERASIFERA   NIGRA        
    ## 6  160776         1071 E 50TH AV    PRUNUS     CERASIFERA   ATROPURPUREUM
    ## # … with 15 more variables: common_name <chr>, assigned <chr>,
    ## #   root_barrier <chr>, plant_area <chr>, on_street_block <dbl>,
    ## #   on_street <chr>, neighbourhood_name <chr>, street_side_name <chr>,
    ## #   height_range_id <fct>, diameter <dbl>, curb <chr>, date_planted <date>,
    ## #   longitude <dbl>, latitude <dbl>, age <dbl>

<!----------------------------------------------------------------------------->

# Exercise 2: Modelling

## 2.0 (no points)

Pick a research question, and pick a variable of interest (we’ll call it
“Y”) that’s relevant to the research question. Indicate these.

<!-------------------------- Start your work below ---------------------------->

**Research Question**: *What is the relationship between the age of
trees with `diameter` or `height_range_id`?*

**Variable of interest**: `diameter`

<!----------------------------------------------------------------------------->

## 2.1 (5 points)

Fit a model or run a hypothesis test that provides insight on this
variable with respect to the research question. Store the model object
as a variable, and print its output to screen. We’ll omit having to
justify your choice, because we don’t expect you to know about model
specifics in STAT 545.

-   **Note**: It’s OK if you don’t know how these models/tests work.
    Here are some examples of things you can do here, but the sky’s the
    limit.
    -   You could fit a model that makes predictions on Y using another
        variable, by using the `lm()` function.
    -   You could test whether the mean of Y equals 0 using `t.test()`,
        or maybe the mean across two groups are different using
        `t.test()`, or maybe the mean across multiple groups are
        different using `anova()` (you may have to pivot your data for
        the latter two).
    -   You could use `lm()` to test for significance of regression.

<!-------------------------- Start your work below ---------------------------->

-   Fit a linear model on `diameter` over `age` to identify the
    relationship between them.

``` r
## sample `ACER` trees and calculate their ages for exercise 2 study ##
exer2_data<-vancouver_trees %>%
  filter(!is.na(date_planted))  %>%
  filter(genus_name=="ACER",diameter!=0) %>%
  mutate(age=interval(date_planted, today())/duration(n=1, unit="years"))

## fit linear model on diameter over age ##
lm_fitted <- lm(diameter ~ age, exer2_data)
print(lm_fitted)
```

    ## 
    ## Call:
    ## lm(formula = diameter ~ age, data = exer2_data)
    ## 
    ## Coefficients:
    ## (Intercept)          age  
    ##      1.1392       0.2574

``` r
## check the model fitting information in observation level ##
model_info <- augment(lm_fitted)
head(model_info)
```

    ## # A tibble: 6 × 8
    ##   diameter   age .fitted .resid     .hat .sigma    .cooksd .std.resid
    ##      <dbl> <dbl>   <dbl>  <dbl>    <dbl>  <dbl>      <dbl>      <dbl>
    ## 1      9    27.9    8.31  0.689 0.000157   4.00 0.00000234      0.172
    ## 2     15    27.9    8.31  6.69  0.000157   4.00 0.000220        1.67 
    ## 3     14    27.9    8.31  5.69  0.000157   4.00 0.000159        1.42 
    ## 4     16    27.9    8.31  7.69  0.000157   4.00 0.000291        1.92 
    ## 5     18    27.9    8.31  9.69  0.000157   4.00 0.000462        2.42 
    ## 6     10.2  27.9    8.31  1.94  0.000157   4.00 0.0000185       0.486

<!----------------------------------------------------------------------------->

## 2.2 (5 points)

Produce something relevant from your fitted model: either predictions on
Y, or a single value like a regression coefficient or a p-value.

-   Be sure to indicate in writing what you chose to produce.
-   Your code should either output a tibble (in which case you should
    indicate the column that contains the thing you’re looking for), or
    the thing you’re looking for itself.
-   Obtain your results using the `broom` package if possible. If your
    model is not compatible with the broom function you’re needing, then
    you can obtain your results by some other means, but first indicate
    which broom function is not compatible.

<!-------------------------- Start your work below ---------------------------->

Use `tidy` function in `broom` package to summarize the coeffients,
p.value and other key components of linear model in a tibble. the
coefficient of age and intercept are shown in **estimate** column and
the last column shows the p.value.

``` r
tidy(lm_fitted)
```

    ## # A tibble: 2 × 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)    1.14    0.0731       15.6 1.85e-54
    ## 2 age            0.257   0.00389      66.2 0

<!----------------------------------------------------------------------------->

# Exercise 3: Reading and writing data

Get set up for this exercise by making a folder called `output` in the
top level of your project folder / repository. You’ll be saving things
there.

## 3.1 (5 points)

Take a summary table that you made from Milestone 2 (Exercise 1.2), and
write it as a csv file in your `output` folder. Use the `here::here()`
function.

-   **Robustness criteria**: You should be able to move your Mini
    Project repository / project folder to some other location on your
    computer, or move this very Rmd file to another location within your
    project repository / folder, and your code should still work.
-   **Reproducibility criteria**: You should be able to delete the csv
    file, and remake it simply by knitting this Rmd file.

<!-------------------------- Start your work below ---------------------------->

``` r
dir.create(here::here("output"))
```

``` r
## A summary table from Milestone 2 ##
pa_summ<-vancouver_trees %>%
  group_by(plant_area) %>%
  mutate(totalTree_pa=n()) %>%
  group_by(plant_area,totalTree_pa,genus_name) %>%
  summarise(n=n(),mean(diameter),min(diameter),max(diameter),median(height_range_id),min(height_range_id),max(height_range_id),.groups="drop") %>%
  arrange(desc(totalTree_pa),desc(n))

## write csv ##
write_csv(pa_summ, here::here("output", "plant_area_summary.csv"))
dir(here::here("output"))
```

    ## [1] "plant_area_summary.csv"

<!----------------------------------------------------------------------------->

## 3.2 (5 points)

Write your model object from Exercise 2 to an R binary file (an RDS),
and load it again. Be sure to save the binary file in your `output`
folder. Use the functions `saveRDS()` and `readRDS()`.

-   The same robustness and reproducibility criteria as in 3.1 apply
    here.

<!-------------------------- Start your work below ---------------------------->

``` r
## save to a .rds file ##
saveRDS(lm_fitted,here::here("output","model.rds"))

## load model.rds ##
readRDS(here::here("output","model.rds"))
```

    ## 
    ## Call:
    ## lm(formula = diameter ~ age, data = exer2_data)
    ## 
    ## Coefficients:
    ## (Intercept)          age  
    ##      1.1392       0.2574

<!----------------------------------------------------------------------------->

# Tidy Repository

Now that this is your last milestone, your entire project repository
should be organized. Here are the criteria we’re looking for.

## Main README (3 points)

There should be a file named `README.md` at the top level of your
repository. Its contents should automatically appear when you visit the
repository on GitHub.

Minimum contents of the README file:

-   In a sentence or two, explains what this repository is, so that
    future-you or someone else stumbling on your repository can be
    oriented to the repository.
-   In a sentence or two (or more??), briefly explains how to engage
    with the repository. You can assume the person reading knows the
    material from STAT 545A. Basically, if a visitor to your repository
    wants to explore your project, what should they know?

Once you get in the habit of making README files, and seeing more README
files in other projects, you’ll wonder how you ever got by without them!
They are tremendously helpful.

## File and Folder structure (3 points)

You should have at least four folders in the top level of your
repository: one for each milestone, and one output folder. If there are
any other folders, these are explained in the main README.

Each milestone document is contained in its respective folder, and
nowhere else.

Every level-1 folder (that is, the ones stored in the top level, like
“Milestone1” and “output”) has a `README` file, explaining in a sentence
or two what is in the folder, in plain language (it’s enough to say
something like “This folder contains the source for Milestone 1”).

## Output (2 points)

All output is recent and relevant:

-   All Rmd files have been `knit`ted to their output, and all data
    files saved from Exercise 3 above appear in the `output` folder.
-   All of these output files are up-to-date – that is, they haven’t
    fallen behind after the source (Rmd) files have been updated.
-   There should be no relic output files. For example, if you were
    knitting an Rmd to html, but then changed the output to be only a
    markdown file, then the html file is a relic and should be deleted.

Our recommendation: delete all output files, and re-knit each
milestone’s Rmd file, so that everything is up to date and relevant.

PS: there’s a way where you can run all project code using a single
command, instead of clicking “knit” three times. More on this in STAT
545B!

## Error-free code (1 point)

This Milestone 3 document knits error-free. (We’ve already graded this
aspect for Milestone 1 and 2)

## Tagged release (1 point)

You’ve tagged a release for Milestone 3. (We’ve already graded this
aspect for Milestone 1 and 2)
