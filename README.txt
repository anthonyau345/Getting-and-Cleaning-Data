Peer-graded Assignment: Getting and Cleaning Data Course Project


The repository consists of the following files:
===============================================

- 'README.txt'- it describes how the script in run_analysis.R works

- 'cookbook.txt': it describes the variables in the tidy_dataset.txt.

- 'tidy_dataset.txt': the final dataset after going through steps 1 to 5 as required.


Here are the scripts to prepare and clean the data which resulted in the final dataset
=======================================================================================

Step 1: download the data and assign column names to each variable in original dataset
=====================================================================================
# download package

library(dplyr)

# Set working directory
setwd("~/Data Science/John Hopkins/3 Getting and Cleaning Data/HAPT Data Set")

# Download the dataset

filename<-"HAPT_Data_Set.zip"

fileURL<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileURL, filename, method = "curl")

unzip(filename)

# Assign all column Names

features<-read.table("UCI HAR Dataset/features.txt", col.names = c("Feature_No", "Features"))
activities<-read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("Activity_No", "Activities"))
subject_train<-read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "Subject")
subject_test<-read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "Subject")
y_train<-read.table("UCI HAR Dataset/train/y_train.txt", col.names = "Activity_No")
y_test<-read.table("UCI HAR Dataset/test/y_test.txt", col.names = "Activity_No")
x_train<-read.table("UCI HAR Dataset/train/X_train.txt", col.names = features$Features)
x_test<-read.table("UCI HAR Dataset/test/X_test.txt", col.names = features$Features)

Step 2: Go through all the Data Transformation steps to arrive at the final dataset
===================================================================================

#1. Merges the training and the test sets to create one data set.
#Combine the training dataset
train<-cbind(subject_train, y_train, x_train)
#Combine the testing dataset
test<-cbind(subject_test, y_test, x_test)
#Merge the training and testing dataset
train_test<-rbind(train, test)

#2. Extracts only the measurements on the mean and standard deviation for each measurement.

mean_std<-train_test %>% select("Subject", "Activity_No", contains("mean"), contains("std"))

#3. Uses descriptive activity names to name the activities in the data set

mean_std$Activity_No<-activities[mean_std$Activity_No, 2]

#4. Appropriately labels the data set with descriptive variable names.

names(mean_std)[2]<-"activities"
names(mean_std) <- gsub("^t", "Time", names(mean_std))
names(mean_std) <- gsub("Acc", "Accelerator", names(mean_std))
names(mean_std) <- gsub("Gyro", "Gyroscope", names(mean_std))
names(mean_std) <- gsub("Mag", "Magnitude", names(mean_std))
names(mean_std) <- gsub("^f", "Frequency_Domain_Signals", names(mean_std))
names(mean_std) <- gsub("tBody", "TimeBody", names(mean_std))

#5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

#create the tidy data set with the average of each subject and activity
tidy_dataset<- mean_std %>%
  group_by(Subject, activities) %>%
  summarise_all(funs(mean))

Step 3: Write the final dataset into a text file
================================================
                
#Write the tidy data set into a file

write.table(tidy_dataset, "tidy_dataset.txt", row.names = FALSE)