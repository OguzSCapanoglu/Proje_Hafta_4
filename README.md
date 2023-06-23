---
title: | 
  | Assignment 4: Collaborating Together
  | Introduction to Applied Data Science
  | 2022-2023
author: |
  | Oğuz Şahin Çapanoğlu
  | o.s.capanoglu@students.uu.nl
  | http://www.github.com/OguzSCapanoglu

date: April 2023
urlcolor: purple
linkcolor: purple
output: 
  pdf_document
---

```{r setup, include = FALSE}
knitr::opts_chunk$set(warning = FALSE, message = FALSE, error=TRUE)
```

## Assignment 4: Collaborating Together 

### Part 1: Contributing to another student's Github repository

In this assignment, you will create a Github repository, containing this document and the .pdf output, which analyzes a dataset individually using some of the tools we have developed. 

This time, make sure to not only put your name and student e-mail in your Rmarkdown header, but also your Github account, as I have done myself. 

However, you will also pair up with a class mate and contribute to each others' Github repository. Each student is supposed to contribute to another student's work by writing a short interpretation of 1 or 2 sentences at the designated place (this place is marked with **designated place**) in the other student's assignment. 

This interpretation will not be graded, but a Github shows the contributors to a certain repository. This way, we can see whether you have contributed to a repository of a class mate. 

**Question 1.1**: Fill in the __github username__ of the class mate to whose repository you have contributed. 

lenasincar

### Part 2: Analyzing various linear models

In this part, we will summarize a dataset and create a couple of customized tables. Then, we will compare a couple of linear models to each other, and see which linear model fits the data the best, and yields the most interesting results.

We will use a dataset called `GrowthSW` from the `AER` package. This is a dataset containing 65 observations on 6 variables and investigates the determinants of economic growth. First, we will try to summarize the data using the `modelsummary` package. 

```{r, warning=FALSE, message=FALSE}
library(AER)
data(GrowthSW)
```

One of the variables in the dataset is `revolutions`, the number of revolutions, insurrections and coup d'etats in country $i$ from 1965 to 1995.

**Question 2.1**: Using the function `datasummary`, summarize the mean, median, sd, min, and max of the variables `growth`, and `rgdp60` between two groups: countries with `revolutions` equal to 0, and countries with more than 0 revolutions. Call this variable `treat`. Make sure to also write the resulting data set to memory. Hint: you can check some examples [here](https://vincentarelbundock.github.io/modelsummary/articles/datasummary.html#datasummary).

```{r}
library(modelsummary); library(tidyverse)

new_data <- GrowthSW %>%
  mutate(treat = if_else(GrowthSW$revolutions > 0, "Revolutions", "Stable"))

datasummary(formula = new_data$growth + new_data$rgdp60 ~ new_data$treat * (mean + sd + median + min + max), data=GrowthSW)
```

**Designated place**: type one or two sentences describing this table of a fellow student below. For example, comment on the mean and median growth of both groups. Then stage, commit and push it to their github repository. 
For the growth variable, while comparing the means, we see from the table that the countries who has revolution has smaller values than the stable ones. This explains that stable countries have a better economic growth rate. 


### Part 3: Make a table summarizing reressions using modelsummary and kable

In question 2, we have seen that growth rates differ markedly between countries that experienced at least one revolution/episode of political stability and countries that did not. 

**Question 3.1**: Try to make this more precise this by performing a t-test on the variable growth according to the group variable you have created in the previous question. 

```{r}
t.test(new_data$growth ~ new_data$treat)

"t = -1.8531, df = 61.015, p-value = 0.06871"
```

**Question 3.2**: What is the $p$-value of the test, and what does that mean? Write down your answer below.

This implies that the T-test supports the null hypothesis, as a P value larger than 1 - confidence interval is statistically insignificant.

We can also control for other factors by including them in a linear model, for example:

$$
\text{growth}_i = \beta_0 + \beta_1 \cdot \text{treat}_i + \beta_2 \cdot \text{rgdp60}_i + \beta_3 \cdot \text{tradeshare}_i + \beta_4 \cdot \text{education}_i + \epsilon_i
$$

**Question 3.3**: What do you think the purpose of including the variable `rgdp60` is? Look at `?GrowthSW` to find out what the variables mean. 

It seems to be the GDP per capita, of the year 1960.

We now want to estimate a stepwise model. Stepwise means that we first estimate a univariate regression $\text{growth}_i = \beta_0 + \beta_1 \cdot \text{treat}_i + \epsilon_i$, and in each subsequent model, we add one control variable. 

**Question 3.4**: Write four models, titled `model1`, `model2`, `model3`, `model4` (using the `lm` function) to memory. Hint: you can also use the `update` function to add variables to an already existing specification.

```{r}
model1 <- lm(data = new_data, growth ~ new_data$treat)
model2 <- update(model1,. ~ . +new_data$rgdp60)
model3 <- update(model2,. ~ . +new_data$tradeshare)
model4 <- update(model3,. ~ . +new_data$education)


```

Now, we put the models in a list, and see what `modelsummary` gives us:

```{r}
list(model1, model2, model3, model4) |> 
  modelsummary(stars=TRUE, gof_map = c("nobs", "r.squared"))




```

**Question 3.5**: Edit the code chunk above to remove many statistics from the table, but keep only the number of observations $N$, and the $R^2$ statistic. 

**Question 3.6**: According to this analysis, what is the main driver of economic growth? Why?

The answer is education, as it creates the largest spike between the R^2 values.

**Question 3.7**: In the code chunk below, edit the table such that the cells (including standard errors) corresponding to the variable `treat` have a red background and white text. Make sure to load the `kableExtra` library beforehand.

```{r}
library(kableExtra)
list(model1, model2, model3, model4) |>
  modelsummary(stars=TRUE, gof_map = c("nobs", "r.squared")) %>%
  row_spec(3:4, backround="red", color="white")

```

**Question 3.8**: Write a piece of code that exports this table (without the formatting) to a Word document. 

```{r}
modelsummary(list(model1, model2, model3, model4), gof_map = c("nobs","r.squared"), title = "Regression table", output = "table_1.docx")
```

## The End
