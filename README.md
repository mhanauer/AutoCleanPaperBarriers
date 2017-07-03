---
title: "SCDMarkIRR"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here we are loading and cleaning the data
```{r}
setwd("~/Desktop")
matt = read.csv("HandCodesMattMolly.csv", header = TRUE)
head(matt)
matt = matt[c(3:4)]
head(matt)
library(psych)
cohen.kappa(matt)

```
Agreement above chance.  Do not use weighted, but this assumes there is some order to the nomial values.  So we are penalized more when I answer 1 and she answers 6 even though 

Now we are running the actual program.

```{r}
library(devtools)
install_github("iqss-research/VA-package")
install_github("iqss-research/ReadMeV1")
library(ReadMe)
setwd("~/Desktop/QualData")
District1 = read.csv("District1.csv", header = TRUE)
District2 = read.csv("District2.csv", header = TRUE)

District2 = District2[c("Q3")]
District2 = District2[-c(1:2), ]
District2 = as.matrix(District2)

District1 = District1[c("Q3")]
District1 = District1[-c(1:2), ]
District1 = as.matrix(District1)

both = rbind(District1, District2)

write.csv(both, "both.csv")
both = read.csv("both.csv", header= TRUE, na.strings = c("", "NA"))
both = na.omit(both)
id = 1:nrow(both)
both = cbind(id, both)
both = both[c(1,3)]
write.csv(both, "both.csv", row.names = FALSE)
```
Now we need to place each of the responses into their own txt file.  Cannot be the same as before, because the order is different. 

Now we are creating the control.txt file, because you already created the txt file with the responses
```{r}
library(devtools)
source_url("https://gist.github.com/benmarwick/9266072/raw/csv2txts.R")
csv2txt("~/Desktop/QualAuto", labels = 1)

set.seed(1)
both1 <- replicate(401, rnorm(1, 0, 1))  

names(both1) <- paste0( "/Users/Authorname/Desktop/QualAuto/",1:ncol(both1), ".txt")
library(reshape2)
both1 = melt(both1)
both1 = both1$variable
colnames(both1) = c("filename")
filename = both1$filename
```
Now we need to create the training and truth data sets.  We are grabbing matt's codes from the matt data set and that will be the "truth" data set.  Then we will stack on the "" for the rest of the values
```{r}

truth1 = truth[c(2)]
truth1 = data.frame(truth = truth1)
truth2 = data.frame(truth = rep("", 401-100))
truth = rbind(truth1, truth2)

trainingset1 = data.frame(trainingset = rep(1,100))
trainingset2 = data.frame(trainingset = rep(0, 401-100))
trainingset = rbind(trainingset1, trainingset2)
control = cbind(filename, truth, trainingset)
write.table(control, "control.txt", row.names = FALSE, sep = ",")

```
Now we run the program
```{r}

setwd("~/Desktop/QualAuto")
undergrad.results = undergrad(sep = ',')
undergrad.preprocess <- preprocess(undergrad.results)
readme.results <- readme(undergrad.preprocess, boot.se = TRUE)
readme.results$CSMF.se
readme.results$est.CSMF

```

