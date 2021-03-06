# customer-review-analysis

---
author: "Goutham Polamreddy"
date: "28 March 2020"
---

# Introduction

### Problem Statement

  The Objective of the project is to provide product recommendations based on the text analysis of Customer reviews.

    
### Need for the Study
    
  The online marketing space is in constant shift as new technologies, services, and marketing tactics gain popularity and become the new standard. Online store owners are one of the many different segments affected by these constant evolutions. In order for these business owners to survive and thrive, they need to be able to make better decisions faster. This is where web analytics comes into play.
    
  This is a Women’s Clothing E-Commerce dataset revolving around the reviews written by customers. Its nine supportive features offer a great environment to parse out the text through its multiple dimensions.
    
    
### Business Opportunity

- Help Business understand customer behaviour
- Contribute in increasing revenue by suggesting better products to customers.
- Support Business in Customer retention by providing relevant suggestions to customers.


# Data Report

**Data Collection** : This data is collected from reviews that customers provided for different products on a E Commerce website.
A quick glance at data below
```{r echo=FALSE}
setwd("C:\\Users\\DELL\\Desktop\\exploRe")
reviewData = read.csv("Womens Clothing E-Commerce Reviews.csv", header = TRUE, stringsAsFactors = FALSE)
for (i in 6:11) {
  reviewData[,i] = as.factor(reviewData[,i])
}
summary(reviewData[,-c(1,2,4,5)])
```
Apart from the columns above there are two more variables namely **Title** and **Review Text** which holds title and content for the review respectively.

Different Attributes have below meanings:

This dataset includes 23486 rows and 10 feature variables. Each row corresponds to a customer review, and includes the variables:

**Clothing ID**: Integer Categorical variable that refers to the specific piece being reviewed.

**Age**: Positive Integer variable of the reviewers age.

**Title**: String variable for the title of the review.

**Review Text**: String variable for the review body.

**Rating**: Positive Ordinal Integer variable for the product score granted by the customer from 1 Worst, to 5 Best.

**Recommended IND**: Binary variable stating where the customer recommends the product where 1 is recommended, 0 is not recommended.

**Positive Feedback Count**: Positive Integer documenting the number of other customers who found this review positive.

**Division Name**: Categorical name of the product high level division.

**Department Name**: Categorical name of the product department name.

**Class Name**: Categorical name of the product class name.


# Exploratory Data Analysis

### Univariate Analysis

Let's look at the distribution of Age of customers who reviewed the product.

```{r include=FALSE}
library(dplyr)
library(ggplot2)
```

```{r echo=FALSE}
reviewData %>% ggplot(aes(x=Age))+ geom_density(fill="#69b3a2", color="#e9ecef", alpha=0.8)+ ggtitle("Customer Age Distribution")
```

From the graph above, a majority of customers are between ages 30 and 50


Now, looking at the count of different clothing available

```{r echo=FALSE}
  reviewData %>% filter(Department.Name != "") %>% ggplot(aes(x = Department.Name)) + geom_bar(fill="#69b3a2", color="#e9ecef", alpha=0.8)
```

Moving on to analysing text columns. Analysing what the reviews talks about majorly helps to gain few insights into the nature of the data.

```{r include=FALSE}
library(tm)
library(SnowballC)
```


## Missing Value Treatment

Looking at the empty values in both columns( False= empty, True= Non empty)

```{r echo=FALSE}
table(reviewData$Title != "", reviewData$Review.Text != "")
```

The summary below of other variables where the text columns are both empty.

```{r echo=FALSE}
reviewData %>% filter(Title=="" & Review.Text=="") %>% select(-c(X,Clothing.ID, Title, Review.Text)) %>% summary()
```

* Positive feedback count of these rows is all 0, so removing these rows in further analysis.
* Also, removing the 14 empty rows from Division Name and Department Name

Summary of data after cleaning of empty values
```{r echo=FALSE}
reviewData = reviewData[reviewData$Review.Text != "" | reviewData$Title != "",]
reviewData = reviewData[reviewData$Division.Name != "",]
summary(reviewData[,-c(1,2,4,5)])
```

### Creation of New Variables

Since there are missing values in either of Review text or Title, concatenate them both. This helps in doing text analysis on single column.

Variables in analysis after concatenation.

```{r echo=FALSE}
reviewData$Review.Text = paste(reviewData$Title, ".", reviewData$Review.Text)
reviewData$Title = NULL
colnames(reviewData)
```

### Removing unwanted variables

Removing first two columns, which are IDs and doesn't add much to the analysis.

Final variables for analysis.

```{r echo=FALSE}
reviewData= reviewData[,-c(1,2)]
colnames(reviewData)
```


### Outliers

No outliers considered in this Analysis

### Univariate analysis contd.

Analysis of word frequencies in the Review text is done below

Following analysis went into, while doing the analysis:

1. Words are converted to lowercase.
2. Punctuation is removed from the content.
3. Stop words are removed.
4. Stemming is done.

```{r include=FALSE}
reviewCorpus =Corpus(VectorSource(reviewData$Review.Text))
reviewCorpus = tm_map(reviewCorpus, tolower)
reviewCorpus = tm_map(reviewCorpus, removePunctuation)
reviewCorpus = tm_map(reviewCorpus, removeWords, stopwords("en"))
reviewCorpus = tm_map(reviewCorpus, stemDocument)
```


```{r include=FALSE}
library(wordcloud)
library(tidytext)
library(tidyr)
library(reshape2)
```

### Sentiment Analysis

We created a new sentiment which describes whether the review is positive or negative.

Below is plot of word frequency plot with both positive and negative words.

```{r echo=FALSE}
reviewDTM <- DocumentTermMatrix(reviewCorpus)
reviewTDM <- TermDocumentMatrix(reviewCorpus)
reviewTDMTidy = tidy(reviewTDM)
reviewSentiments = reviewTDMTidy %>% inner_join(get_sentiments("bing"), by = c(term = "word"))
```

```{r echo=FALSE}
reviewSentiments %>%
  count(sentiment, term, wt = count) %>%
  filter(n >= 300) %>%
  mutate(n = ifelse(sentiment == "negative", -n, n)) %>%
  mutate(term = reorder(term, n)) %>%
  ggplot(aes(term, n, fill = sentiment)) +
  geom_bar(stat = "identity") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  ylab("Contribution to sentiment")
```


And word cloud below

```{r echo=FALSE}
reviewSentiments %>%
  count(term, sentiment, sort = TRUE) %>%
  acast(term ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("#F8766D", "#00BFC4"),
                   max.words = 40)
```


### Bivariate Analysis


Now looking at the how each variable varies with recommendation Ind

```{r echo=FALSE}
reviewData$Positive.Feedback.Count = as.numeric(reviewData$Positive.Feedback.Count)
ggplot(reviewData) +
 aes(x = Recommended.IND, y = Age) +
 geom_boxplot(fill = "#6baed6") +
 theme_minimal()
```


There is no significant difference in people Ages who recommended and not recommended a product

Next to positive feedback count vs recommendations

```{r echo=FALSE}
ggplot(reviewData) +
 aes(x = Recommended.IND, y = Positive.Feedback.Count) +
 geom_boxplot(fill = "#9ecae1") +
 theme_minimal()
```

From the graph above, there are users who found both reviews where the product is recommended/not recommemended helpful.


### Insights

1. In many postive reviews people used the word "love".
2. In most reviews which are negative people described it with "disappoint"
3. Adding on to points 1 and 2, people recommended products with positive reviews and not for negative reviews.
4. Age has no influence on recommendations.
5. People found reviews helpful both recommended and not recommended.
