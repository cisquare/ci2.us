---
title: Text Mining Shakespeare's First Folio
author: Eric Kammers 
date: '2021-11-05'
slug: text-mining-shakespeare-first-folio
categories:
  - Research
  - Data Analysis
tags:
  - text mining
  - Shakespeare 
  - term document
  - cluster
  - visualization
---

In the age of big data, it is no surprise that the rate data is produced has exponentially increased. In 2013, research company [SINTEF](https://www.sintef.no/en/latest-news/2013/big-data-for-better-or-worse/) cited a claim that 90% of the world's data has been created in just the past two years! While more data is created, this does not necessarily mean inferring conclusions from it has become any easier. For instance, text mining has become a popular form of analysis by formatting text in a quantitative manner. In this post, I wanted to explore some readily available data and the steps it takes to process and format it in an interpretable manner.

> "We are such stuff as dreams are made on; and our little life is rounded with a sleep." - William Shakespeare [The Tempest]

I chose Shakespeare's first folio to analyze for the following reasons: its public availability and to practice analysis in the popular field of natural language processing. For instance, Shakespearean plays are typically placed into one of three categories: comedy, tragedy, or history. What if there is an empirical way to classify Shakespearean plays based on the word usage. In other words, can we use modern computing techniques to identify distinctions between Shakespeare's plays?

# Download
To begin, I downloaded Shakespeare's complete works from Project Gutenberg. The file downloaded contained the 36 plays from Shakespeare's First Folio and his Sonnets. I first read the text document with R line-by-line. When previewing the result, and looking deeper into the text, I noticed there is quite a bit of unnecessary information that will contaminate the analysis, such as the text below.
![unnecessary text](https://user-images.githubusercontent.com/69160694/140597800-1dfd46cb-efc0-466e-a090-c4480ee272f6.png)

# Cleaning & Formatting Data
After combining the lines into one string, I wanted to remove irrelevant information (i.e. any text that is not related to the plays themselves). When taking a closer look into the text file, I noticed that there was a copyright section (see below) between each act of every play. This seemed like a good way to partition the text into chunks for the next steps.
> "<<THIS ELECTRONIC VERSION OF THE COMPLETE WORKS OF WILLIAM SHAKESPEARE IS COPYRIGHT 1990-1993 BY WORLD LIBRARY, INC., AND IS PROVIDED BY PROJECT GUTENBERG ETEXT OF ILLINOIS BENEDICTINE COLLEGE WITH PERMISSION. ELECTRONIC AND MACHINE READABLE COPIES MAY BE DISTRIBUTED SO LONG AS SUCH COPIES (1) ARE FOR YOUR OR OTHERS PERSONAL USE ONLY, AND (2) ARE NOT DISTRIBUTED OR USED COMMERCIALLY. PROHIBITED COMMERCIAL DISTRIBUTION INCLUDES BY ANY SERVICE THAT CHARGES FOR DOWNLOAD TIME OR FOR MEMBERSHIP.>>"

Now that the text has become disaggregated in an organized fashion, I eliminated the sections that did not contain any information regarding the 36 plays of Shakespeare's first folio. The next steps were to organize and label the data to prepare for processing the text.

**The Pattern in the List of Partitioned Text:**
1.	Dramatis Personae (a list containing the play's title and characters)
2.	Act I
3.	Act II
4.	Act III
5.	Act IV
6.	Act V
7.	Repeat 1-6 until there are no more plays

The Dramatis Personae looks like a good way to indicate 1.) which plays are located in the list and 2.) where each play begins in the list. The next steps I took were to identify which chunk of texts contains the words "Dramatis Personae" and to extract the play titles verbatim from the Dramatis Personae. One note I would like to make is that this method was not perfect. After applying the method, I learned that for the play, The Comedy of Errors, the title had been located before the copyright material I had used to indicate the text partitioning. With some manual adjustments, I could accommodate this error.
Next, I organized the list of text into 36 distinct lists based on the play number, resulting in 36 lists of 5 acts each. After grouping the plays, I manually coded each play into one of the three groups: comedy, tragedy, or history. These are a common way Shakespeare's first folio is categorized based on my preliminary research.

![Code of plays](https://user-images.githubusercontent.com/69160694/140598076-382e746e-e60b-4af7-933e-61f511039d5f.png)

After labeling each play with its appropriate category, I regrouped the plays so they follow the order: Comedy, History, Tragedy.

![Ordered code](https://user-images.githubusercontent.com/69160694/140598598-9fe9dcd6-7aff-41d3-abb6-af89a8af43a7.png)

# Term Document Matrix

Now here is where the fun begins! The next steps I consolidated into a function so I could repeat the process on any list of text. The goal was to create a method where we can compare acts of plays or entire plays in a quantitative manner based on the language used in the text. The end result, called a term document matrix (TDM), will be most suitable for this purpose since it quantifies the number of times a word is used in each document and is formatted in an organized fashion for easy comparison. Below is the function I created for processing a list of text into a TDM. First, it takes any vector of texts and formats it into a corpus; this could be text from each play, each act, each scene, etc.

**After Formatted Into Corpus:**
1.	All text is transformed to lowercase.
2.	Punctuation is removed.
3.	Numeric symbols are removed.
4.	Stop words are removed.
5.	Stemming is conducted to reduce conjugated verbs to one tense (e.g. runs and running to run), plurals to singular, and help remove some redundancies. The package, Text Miner I used for this step, uses Porter Stemming.
6.	All extra white space is stripped.
7.	Words are counted and formatted into a term document matrix.

```
text2TDM <- function(text){
  doc.vec <- VectorSource(as.vector(text))
  doc.corpus <- Corpus(doc.vec)
  doc.corpus <- tm_map(doc.corpus, content_transformer(tolower))
  doc.corpus <- tm_map(doc.corpus, removePunctuation)
  doc.corpus <- tm_map(doc.corpus, removeNumbers)
  doc.corpus <- tm_map(doc.corpus, removeWords, stopwords("english"))
  doc.corpus <- tm_map(doc.corpus, stemDocument)
  doc.corpus <- tm_map(doc.corpus, stripWhitespace)
  TDM <- TermDocumentMatrix(doc.corpus)
  return(TDM)
```

For reference, below are the first ten terms in the TDM when comparing from play to play. Note sparsity of the matrix.

![First ten terms in TDM](https://user-images.githubusercontent.com/69160694/140598866-ef5a5198-2b01-46e5-ae02-9867f0a7eded.png)

# Preparing Data for Clustering
My expertise in natural language processing is fairly limited. For the next steps, I'm not 100% certain are commonly used in practice. I wanted to normalize the term counts of each document. This intention was to ensure that I could compare documents without the bias of document length. For example, if a document used 10,000 words but another used 50,000 words this could impact my final comparison. To resolve this document length bias, I divided each column of my matrix (documents) by the total number of words in the document. This resulted in the proportion each word was used in the document ranging in a number between 0 and 1. After normalization, I wanted to eliminate sparse terms from the matrix. If a word is unique to a document (e.g. a character's name), this will not provide any significance when comparing between other documents. Therefore, if a term is only found in one document I removed the word post normalization.

# Clustering and Visualizing
The next steps I took were to conduct Principal Component Analysis on the Term Document Matrix to help visualize the plays in a simpler dimensional space. Currently, there are 8958 words we are comparing across 36 plays. I want to see if we can visualize the plays in a 2-dimensional or 3-dimensional space. By creating new basis vectors from Singular Value Decomposition, I can now look at the data from a new perspective where variables are guaranteed to be uncorrelated. Below are bar plots of the new data vectors represented in six dimensions.

![Bar plots](https://user-images.githubusercontent.com/69160694/140599087-b5977bf6-776d-4594-bbf3-3568c0ab211f.png)

When taking a look at the first two dimensions, it appears that they will sufficiently capture the distinction between the play types. Next, let's plot them together to see if there are any distinct groups.

![2d](https://user-images.githubusercontent.com/69160694/140599378-1e8d684e-3481-4927-b9a4-7271389b5e1c.png)

Interestingly, there are three groupings presented when we plot the first two dimensions. Besides some outliers, there can be an agreed distinction between the three groups. The upper right includes primarily comedies, colored red. The lower right includes primarily tragedies, colored blue. Finally, the centralized left group includes primarily histories, colored green. Which plays are in each group? Additionally, which plays are the outliers of each group and why? Below I took the steps to apply the partitioning around medoids, an algorithm for clustering that is similar to k-means, to identify three groups based on the plot above. As expected, the groups are similar to the ones observed in the plot above.

![partitioning](https://user-images.githubusercontent.com/69160694/140599828-5ee3cbe0-7e20-4ec7-9995-b2108dcefd9f.png)

Now that the groups have been identified, let's see what plays fall into each group.

|Cluster 1 | Primarily Comedies, Upper Right
|:---------------------:|:-------|
|As You Like It |Comedy|
|The Comedy Of Errors|	Comedy|
|Love's Labour's Lost|	Comedy|
|The Merchant Of Venice|	Comedy|
|The Merry Wives Of Windsor|	Comedy|
|A Midsummer Night's Dream|	Comedy|
|Much Ado About Nothing|	Comedy|
The Taming Of The Shrew	|Comedy|
|Twelfth Night; Or, What You Will|	Comedy|
|The Two Gentlemen Of Verona|	Comedy|
|The First Part Of King Henry IV|	History|
|The Second Part Of King Henry IV|	History|
|The Tragedy Of Romeo And Juliet|	Tragedy|

*Two histories and one tragedy are outliers.*
 
|Cluster 2 | Primarily Histories, Center Left|
|:---------------------:|:-------|
|The Life Of King Henry V|	History|
|The First Part Of Henry VI|	History|
|The Second Part Of King Henry VI|	History|
|The Third Part Of King Henry VI|	History|
|King John|	History|
|King Richard II|	History|
|King Richard III|	History|
|The Tragedy Of Titus Andronicus|	Tragedy|

*One tragedy is an outlier.*
 
|Cluster 3 | Primarily Tragedies, Lower Right|
|:---------------------:|:-------|
|All's Well That Ends Well|	Comedy|
|Measure For Measure|	Comedy|
|The Tempest|	Comedy|
|The Winter's Tale|	Comedy|
|King Henry VIII|	History|
|The Tragedy Of Antony And Cleopatra|	Tragedy|
|The Tragedy Of Coriolanus|	Tragedy|
|Cymbeline|	Tragedy|
|The Tragedy Of Hamlet, Prince Of Denmark|	Tragedy|
|The Tragedy Of Julius Caesar|	Tragedy|
|The Tragedy Of King Lear|	Tragedy|
|The Tragedy Of Macbeth|	Tragedy|
|The Tragedy Of Othello, Moor Of Venice|	Tragedy|
|The Life Of Timon Of Athens|	Tragedy|
|The History Of Troilus And Cressida|	Tragedy|

*Four comedies and one history are outliers.*

# Words in Each Cluster

Next let's see which word roots are in every play of each cluster.

**Cluster 1:**
![Cluster1:1-32](https://user-images.githubusercontent.com/69160694/140600364-7bfbe16d-39a3-4bf9-bb03-a105146c9da9.png)

**Cluster 2:**
![temp5](https://user-images.githubusercontent.com/69160694/140600429-456cb228-e47c-44f9-a45a-9cfc0db23348.png)

**Cluster 3:**
![Cluster3](https://user-images.githubusercontent.com/69160694/140600412-799ab760-80b6-4daa-be9e-1c58680e4c0a.png)

Unfortunately, my knowledge of Shakespeare is fairly limited to justify any hypothesis I could make to why some of these plays are grouped this way. However, I think looking into the definition of a Shakespearean comedy, tragedy, and history may help provide some insights. Additionally, researching the individual plays themselves that are outliers of the groups could fill in gaps to why the plays are clustered in this manner.

# Final Thoughts

Besides the categories for the plays mentioned in this post (comedies, tragedies, and histories), I found that some of Shakespeare's plays have other groupings associated with them. For instance, there plays categorized under 'late romances' and 'problem plays' that could provide some explanation to the anomalies. Late romances include: The Tempest, The Winter's Tale, Cymbeline, and Pericles. Problem plays include: All's Well That Ends Well, Measure For Measure, The Merchant of Venice, Timon of Athens, Troilus and Cressida, and The Winter's Tale. Another few remarks to make regarding the play categories are that the groupings: comedy, tragedy, and history were not created by Shakespeare but by scholars. Furthermore, the category history appears arbitrary when seeking a global perspective. All the history plays are based on English history, while there are other plays concerning Roman history that were never placed in this category.

# Future Work

Another area I would find interesting regarding text mining and Shakespeare is the act structure of a Shakespearean play. All Shakespeare's plays are composed of a five act structure. What if there is some significant signature linking the language used in each act of a Shakespearean play (i.e. is there a way to identify if a text provided is from Act I of any Shakespearean play solely on the language used?).
My code for the figures and analysis can be found [here](https://github.com/ekmmrs/Text-Mining-Shakespeare-s-First-Folio/blob/master/Shakespeare_Text_Mining_Post.Rmd).








