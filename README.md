# Tidydata-set
 # Load the needed packages packages &lt;- c("data.table", "reshape2", "dplyr") sapply(packages, require, character.only=TRUE, quietly=TRUE)  # Assumes the Git repository : https://github.com/dholtz/GettingAndCleaningData # has been cloned to a users local machine, and the R, setwd(), has been used  # to set the working directory to the root of this cloned repository. path &lt;- getwd()  # Give warning to set the working directory if not able to find data files. projectDataPath &lt;- file.path(path, "project_data") fileCount &lt;- length(list.files(projectDataPath, recursive=TRUE)) if (fileCount != 28) {   stop("Please use setwd() to the root of the cloned repository.") }  # Read in the 'Subject' data dtTrainingSubjects &lt;- fread(file.path(projectDataPath, "train", "subject_train.txt")) dtTestSubjects  &lt;- fread(file.path(projectDataPath, "test" , "subject_test.txt" ))  # Read in the 'Activity' data dtTrainingActivity &lt;- fread(file.path(projectDataPath, "train", "Y_train.txt")) dtTestActivity  &lt;- fread(file.path(projectDataPath, "test" , "Y_test.txt" ))  # Read in the 'Measurements' data # Switching to standard, read.table to avoid the following possible error: # https://github.com/Rdatatable/data.table/issues/487 # No time to figure out where this, 'works again now' version is dtTrainingMeasures &lt;- data.table(read.table(file.path(projectDataPath, "train", "X_train.txt"))) dtTestMeasures  &lt;- data.table(read.table(file.path(projectDataPath, "test" , "X_test.txt")))  # Row merge the Training and Test Subjects # http://www.statmethods.net/management/merging.html dtSubjects &lt;- rbind(dtTrainingSubjects, dtTestSubjects) # Why setnames() ?? http://stackoverflow.com/questions/10655438/rename-one-named-column-in-r setnames(dtSubjects, "V1", "subject")  # Row merge the Training and Test Activities dtActivities &lt;- rbind(dtTrainingActivity, dtTestActivity) setnames(dtActivities, "V1", "activityNumber")  # Merge the Training and Test 'Measurements' data dtMeasures &lt;- rbind(dtTrainingMeasures, dtTestMeasures)  # Column merge the subjects to activities dtSubjectActivities &lt;- cbind(dtSubjects, dtActivities) dtSubjectAtvitiesWithMeasures &lt;- cbind(dtSubjectActivities, dtMeasures)  # Order all of the combined data by, subject and activity setkey(dtSubjectAtvitiesWithMeasures, subject, activityNumber)  ## Read in the 'features.txt'  ## This file matches up to the columns in the data.table, dtSubjectActivitiesWithMeasures ## with the features/measures. dtAllFeatures &lt;- fread(file.path(projectDataPath, "features.txt")) setnames(dtAllFeatures, c("V1", "V2"), c("measureNumber", "measureName"))  # Use grepl to just get features/measures related to mean and std dtMeanStdMeasures &lt;- dtAllFeatures[grepl("(mean|std)\\(\\)", measureName)] # Create a column to 'index/cross reference' into the 'measure' headers # in dtSubjectActivitiesWithMeasures dtMeanStdMeasures$measureCode &lt;- dtMeanStdMeasures[, paste0("V", measureNumber)]  # Build up the columns to select from the data.table, # dtSubjectActivitiesWithMeasures columnsToSelect &lt;- c(key(dtSubjectAtvitiesWithMeasures), dtMeanStdMeasures$measureCode) # Just take the rows with the columns of interest ( std() and mean() ) dtSubjectActivitesWithMeasuresMeanStd &lt;- subset(dtSubjectAtvitiesWithMeasures,                                                  select = columnsToSelect)  # Read in the activity names and give them more meaningful names dtActivityNames &lt;- fread(file.path(projectDataPath, "activity_labels.txt")) setnames(dtActivityNames, c("V1", "V2"), c("activityNumber", "activityName"))  # Merge the 'meaningful activity names' with the  # dtSubjectActiitiesWithMeasuresMeanStd dtSubjectActivitesWithMeasuresMeanStd &lt;- merge(dtSubjectActivitesWithMeasuresMeanStd,                                                 dtActivityNames, by = "activityNumber",                                                 all.x = TRUE)  # Sort the data.table, dtSubjectActivitesWithMeasuresMeanStd setkey(dtSubjectActivitesWithMeasuresMeanStd, subject, activityNumber, activityName)  # Convert from a wide to narrow data.table using the keys created earlier dtSubjectActivitesWithMeasuresMeanStd &lt;- data.table(melt(dtSubjectActivitesWithMeasuresMeanStd,                                                           id=c("subject", "activityName"),                                                           measure.vars = c(3:68),                                                           variable.name = "measureCode",                                                           value.name="measureValue"))  # Merge measure codes dtSubjectActivitesWithMeasuresMeanStd &lt;- merge(dtSubjectActivitesWithMeasuresMeanStd,                                                 dtMeanStdMeasures[, list(measureNumber, measureCode, measureName)],                                                 by="measureCode", all.x=TRUE)  # Convert activityName and measureName to factors dtSubjectActivitesWithMeasuresMeanStd$activityName &lt;-    factor(dtSubjectActivitesWithMeasuresMeanStd$activityName) dtSubjectActivitesWithMeasuresMeanStd$measureName &lt;-    factor(dtSubjectActivitesWithMeasuresMeanStd$measureName)  # Reshape the data to get the averages  measureAvgerages &lt;- dcast(dtSubjectActivitesWithMeasuresMeanStd,                            subject + activityName ~ measureName,                            mean,                            value.var="measureValue")  # Write the tab delimited file write.table(measureAvgerages, file="tidyData.txt", row.name=FALSE, sep = "\t")   181 
