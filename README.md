This is the repository for the Project submission

1. – This script downloads the data for the url provided.
```if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="./data/Dataset.zip")```

2. – The data needs to be unzipped to be able to be processed. This script unzips the data file
unzip(zipfile="./data/Dataset.zip",exdir="./data")

3. – Once the data is available we need to be able to see the list of files, number of files and the type of files available. This script helps to see the list of files in the directory
path_rf <- file.path("./data" , "UCI HAR Dataset")
files<-list.files(path_rf, recursive=TRUE)
files (This will print the list of files in the folder)

4. – Once we know what are the files required for analysis, the next step is to read the different files (Activity files, Subject files and Feature files). We need to consider the test and training datasets for the analysis.  The activity file gives the list of the activities, the subject gives the list of the participants and the features files give the different measurements taken. We use the read.table function to read the respective files. 
ActivityTest  <- read.table(file.path(path_rf, "test" , "Y_test.txt" ),header = FALSE)
ActivityTrain <- read.table(file.path(path_rf, "train", "Y_train.txt"),header = FALSE)
SubjectTrain <- read.table(file.path(path_rf, "train", "subject_train.txt"),header = FALSE)
SubjectTest  <- read.table(file.path(path_rf, "test" , "subject_test.txt"),header = FALSE)
FeaturesTest  <- read.table(file.path(path_rf, "test" , "X_test.txt" ),header = FALSE)
FeaturesTrain <- read.table(file.path(path_rf, "train", "X_train.txt"),header = FALSE)

5. – We require both the training and test data sets for the analysis. So we use the rbind function to merge the test and training datasets for activity, subjects and features. The script below merges the data row wise using the rbind function. The features dataframe has all the observations for the different measurements.
Subjectdata <- rbind(SubjectTrain, SubjectTest)
Activitydata<- rbind(ActivityTrain, ActivityTest)
Featuresdata<- rbind(FeaturesTrain, FeaturesTest)

6. – The dataframe derived till step5 does not have the appropriate column headers. To be able to carry out the next steps of the analysis the columns need to be renamed appropriately. The script below renames the columns – The subject column indicates the participant number, the activity column gives the number of the activity performed. The ‘features.txt’ file has the list of the measurements taken. The scripts below will add the appropriate column headers.
names(Subjectdata)<-c("subject")
names(Activitydata)<- c("activity")
FeatureNames <- read.table(file.path(path_rf, "features.txt"),head=FALSE)
names(Featuresdata)<- FeatureNames$V2

7. – We need to match the activities performed against the subject and merge with the features data (this gives the actual values for the different measurements). The cbind function helps put the columns together and get a complete tidy dataset.
completedata <- cbind(Subjectdata, Activitydata)
Data <- cbind(Featuresdata, completedata)

8. – The next part of the project is to extract only the ‘mean’ and ‘std’. The script below uses the grep function (it recognizes a pattern) and extracts only the ‘mean’ and ‘std’. 
selectdata<-FeatureNames$V2[grep("mean\\(\\)|std\\(\\)", FeatureNames$V2)]

9. – The mean and std need to be mapped to the subject and the activity. The detail for this is obtained from the first dataframe (Data) created.  
selectedNames<-c(as.character(selectdata), "subject", "activity")
Data<-subset(Data,select=selectedNames)

10. –As the activity details are numeric the actual activity labels need to be added and this is done using the ‘if else’ function. 
for(i in 1:nrow(Data)){
    
    if(Data$activity[i]==as.integer(1)){
        Data$activity[i]<-"WALKING"
    }
    else if(Data$activity[i]==as.integer(2)){
        Data$activity[i]<-"WALKING_UPSTAIRS"
    }
    else if(Data$activity[i]==as.integer(3)){
        Data$activity[i]<-"WALKING_DOWNSTAIRS"
    }
    else if(Data$activity[i]==as.integer(4)){
        Data$activity[i]<-"SITTING"
    }
    else if(Data$activity[i]==as.integer(5)){
        Data$activity[i]<-"STANDING"
    }
    else if(Data$activity[i]==as.integer(6)){
        Data$activity[i]<-"LAYING"
    }
}
head(Data$activity,30)    

11. – the column headers need to be expanded to be more descriptive. The script below uses the gsub function (looks for a pattern and substitutes) and the expanded names for the acronyms in the header are added. The data frame that is obtained is the ‘TIDY DATASET’ with only the required columns extracted, descriptive headers and the activity and subject mapped.
names(Data)<-gsub("^t", "time", names(Data))
names(Data)<-gsub("^f", "frequency", names(Data))
names(Data)<-gsub("Acc", "Accelerometer", names(Data))
names(Data)<-gsub("Gyro", "Gyroscope", names(Data))
names(Data)<-gsub("Mag", "Magnitude", names(Data))
names(Data)<-gsub("BodyBody", "Body", names(Data))

12. – The script below caters to the last part of the project. For this the ‘plyr’ package needs to be installed first. The final dataframe (Data2) is the second, independent tidy data set with the average of each variable for each activity and each subject.
library(plyr);
Data2<-aggregate(. ~subject + activity, Data, mean)
Data2<-Data2[order(Data2$subject,Data2$activity),]
write.table(Data2, file = "tidydata.txt",row.name=FALSE)
