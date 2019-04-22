---
title: "CodeBook"
author: "Hatem Jasim Hatem"
date: "April 22, 2019"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Analysis Script 

The `run_analysis.R` script performs the data preparation and then followed by the 5 steps required as described in the course project's definition.

1. Download the dataset
+ Dataset downloaded and extracted under the folder called UCI HAR Dataset
```{r}
filename <- "getdata_projectfiles_UCI HAR Dataset.zip"
if (!file.exists("UCI HAR Dataset")) { 
  unzip(filename) 
}
```
2. Read files
```{r}
table_file <- c("UCI HAR Dataset/features.txt", 
                "UCI HAR Dataset/activity_labels.txt", 
                "UCI HAR Dataset/test/subject_test.txt", 
                "UCI HAR Dataset/test/X_test.txt", 
                "UCI HAR Dataset/test/y_test.txt", 
                "UCI HAR Dataset/train/subject_train.txt", 
                "UCI HAR Dataset/train/X_train.txt", 
                "UCI HAR Dataset/train/y_train.txt")
list_table <- lapply(table_file, read.table)
```
3. Assign each data to variables
```{r}
names(list_table) = c("features", "activities", "subject_test", 
                      "X_test", "y_test", "subject_train", "X_train", 
                      "y_train")

```
4. Merges the training and the test sets to create one data set
```{r}
X<- rbind(list_table$X_train, list_table$X_test)
Y<- rbind(list_table$y_train, list_table$y_test)
Subject <- rbind(list_table$subject_train, list_table$subject_test)
Data <- cbind(Subject, X, Y)
names(Data) <- c("Subject", as.character(list_table[["features"]][,2]), 
                 "Activity")
```
5. Extracts only the measurements on the mean and standard deviation for each measurement
```{r}
index = which(grepl("Subject|Activity|mean|std",  names(Data), ignore.case = TRUE))
SubData <- Data[, index]
```
6. Uses descriptive activity names to name the activities in the data set
```{r}
SubData$Activity <- list_table$activities[SubData$Activity, 2]
```
7. Appropriately labels the data set with descriptive variable names

```{r}
names(SubData)<-gsub("Acc", "Accelerometer", names(SubData))
names(SubData)<-gsub("Gyro", "Gyroscope", names(SubData))
names(SubData)<-gsub("BodyBody", "Body", names(SubData))
names(SubData)<-gsub("Mag", "Magnitude", names(SubData))
names(SubData)<-gsub("^t", "Time", names(SubData))
names(SubData)<-gsub("^f", "Frequency", names(SubData))
names(SubData)<-gsub("tBody", "TimeBody", names(SubData))
names(SubData)<-gsub("-mean()", "Mean", names(SubData), ignore.case = TRUE)
names(SubData)<-gsub("-std()", "STD", names(SubData), ignore.case = TRUE)
names(SubData)<-gsub("-freq()", "Frequency", names(SubData), ignore.case = TRUE)
names(SubData)<-gsub("angle", "Angle", names(SubData))
names(SubData)<-gsub("gravity", "Gravity", names(SubData))

```
8. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject
```{r warning = FALSE, message= FALSE}
library(dplyr)
TidyData <- SubData %>%
  group_by(Subject, Activity) %>%
  summarise_all(funs(mean))
write.table(TidyData, "TidyData.txt", row.name=FALSE)
```
```{r}
dim(TidyData)
```


