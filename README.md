#This is an R Markdown document that describes the coding steps used to download the #zip file, extract the zip file, load the data into R and relevant processing done to #create a tidy dataset.


#loading required packages
library(dplyr)
library(tidyr)

#downloading the zip file

if(!file.exists("data")){dir.create("data")}
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",
              "./data/data.zip",method = "curl")

#extracting data from zip file
unzip("./data/data.zip",exdir = "./data")

#loading test, train and common datasets into R
x_test <- read.table("./data/UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./data/UCI HAR Dataset/test/Y_test.txt")
subject_test <- read.table("./data/UCI HAR Dataset/test/subject_test.txt")

x_train <- read.table("./data/UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./data/UCI HAR Dataset/train/Y_train.txt")
subject_train <- read.table("./data/UCI HAR Dataset/train/subject_train.txt")

features <- read.table("./data/UCI HAR Dataset/features.txt")
activity_labels <- read.table("./data/UCI HAR Dataset/activity_labels.txt")

#Creating variable names
###assigning feature names to variables in x train and x test 
colnames(x_test) <- t(features[,2])
colnames(x_train) <- t(features[,2])

###assigning variable names to the rest
colnames(y_test) <- c("activity_id")
colnames(y_train) <- c("activity_id")

colnames(subject_test) <- c("subject")
colnames(subject_train) <- c("subject")

colnames(activity_labels) <- c("activity_id","activity")

#binding x-test, y-test and subject ids to create test dataset
test_data <- bind_cols(subject_test,y_test,x_test)

#binding x-train, y-train and subject ids to create training dataset
train_data <- bind_cols(subject_train,y_train,x_train)

#binding train and test datasets
test_data <- mutate(test_data,subject_type="test")
train_data <- mutate(train_data,subject_type = "train")
full_activity_data <- bind_rows(test_data,train_data)

#creating descriptive acitivity labels
full_activity_data <- merge(full_activity_data,activity_labels,
                         by.x = "activity_id",
                         by.y = "activity_id")
###Converting to data frame tbl for better viewability
full_activity_data <- tbl_df(full_activity_data) 

#selecting only the mean/std variables
full_activity_data_1 <- select(full_activity_data,subject,activity, subject_type,
                               contains("mean()"),contains("std()"))

#grouping and summarizing by activity and subject
full_activity_data_2 <- group_by(full_activity_data_1,activity,subject,subject_type)

full_activity_data_3 <- summarise_all(full_activity_data_2,funs(mean))

#Exporting the tidy data
write.table(full_activity_data_3,
            file = "./data/UCI HAR Dataset/avg_features_data.txt",row.names = FALSE,
            sep = "\t")
