**Big Data Mini Project on USA Crime Analysis**
-------------------------------------------

**Task:**


Write a MapReduce/Pig program to calculate the number of cases investigated under each FBI code

Write a MapReduce/Pig program to calculate the number of cases investigated under Ward 32

Write a MapReduce/Pig program to calculate the number of arrests in theft district wise

Write a MapReduce/Pig program to calculate the number of arrests done between October 2014 and October 2015


**Dataset:**


Introduction:

This dataset contains attributes related to crimes taking place in various areas like type of crime, FBI code related to that criminal case, arrest frequency, location of crime etc.


Google Drive Link: 

https://drive.google.com/file/d/0B1QaXx7tpw3SaUJHOHBZclBXWG8/view?usp=sharing


Dataset Description:

The columns present in the dataset:

ID, Case Number, Date, Block, IUCR, Primary Type, Description, Location Description, Arrest, Domestic, Beat, District, Ward, Community Area, FBICode, X Coordinate, Y Coordinate, Year, Updated On, Latitude, Longitude, Location

![enter image description here](https://user-images.githubusercontent.com/29932053/32462135-5295e12e-c306-11e7-84f6-ef6c34a20c7d.png)


![enter image description here](https://user-images.githubusercontent.com/29932053/32462187-84fdc834-c306-11e7-80c7-42547e6ffcc4.png)


**Solution:**
*Pig program to calculate the number of cases investigated under each FBI code*

Start the pig terminal in local mode

    pig -x local

Loading the dataset into the relation crimeDataset using the Pig CSV parser and specifying that the data is comma separated, not multiline, should be in Unix format and to skip the header

In following load statement, we haven’t specified the datatype of each column, therefore the default type of all columns of the dataset is bytearray

    crimeDataset = LOAD '/path/to/filr/in/local/file/system/Crimes_-_2001_to_present.csv' USING org.apache.pig.piggybank.storage.CSVExcelStorage
    (',','NO_MULTILINE','UNIX','SKIP_INPUT_HEADER') AS (ID,Case_Number,Date,Block,IUCR,Primary_Type,Description,Location_Description,Arrest,Domestic,Beat,Distirct,Ward,Community_Area,FBI_Code,XCordinate,YCordinate,Year,UpdatedOn,Lattitude,Logitute,Location);

Generating the data (each row) of the above loaded relation with the corresponding datatype for each column. We can also convert all columns to chararray datatype

    crimeDataset_Converted = FOREACH crimeDataset GENERATE $0 AS ID:int,$1 AS Case_Number:chararray,$2 AS Date:chararray,$3 AS Block:chararray,$4 AS IUCR:chararray,$5 AS Primary_type:chararray,$6 AS Description:chararray,$7 AS Location_Description:chararray,$8 AS Arrest:chararray,$9 AS Domestic:boolean,$10 AS Beat:int,$11 AS Distirct:int,$12 AS Ward:int,$13 AS Community_Area:int,$14 AS FBI_Code:chararray,$15 AS XCordinate:int,$16 AS YCordinate:int,$17 AS Year:int,$18 AS UpdatedOn:chararray,$19 AS Lattitude:float,$20 AS Longitute:float,$21 AS Location:chararray);



Group all the data in the data-loaded relation crimeDataset by FBICode in the relation groupFBICodes. This will result in groups of record for each FBICode

    groupFBICodes = GROUP crimeDataset BY FBI_Code;

Generating all cases under each FBICode by flattening (removing/un-nest the tuples) each group and counting the records in each flatten group and storing it in the relation casesUnderEachFBICode. Thus, we get the number of cases investigated under each FBI code

    casesUnderEachFBICode = FOREACH groupFBICodes GENERATE FLATTEN(group) AS FBI_Code,COUNT($1)

Displaying the result in the relation casesUnderEachFBICode

    DUMP casesUnderEachFBICode;
**Output:**

![enter image description here](https://user-images.githubusercontent.com/29932053/32502466-cbd95384-c3a8-11e7-953f-f463f50b1154.png)

*Pig program to calculate the number of cases investigated under Ward 32*


Using the relation crimeDataset that was previously loaded with the dataset in the above problem

Filtering the relation crimeDataset so as to get only the records that have 32 as Ward number. This is stored in the relation filterWard

    filterWard = FILTER crimeDataset BY Ward==32;

Grouping all the data of the relation filterWard and generating the count (counting the records) of the grouped data. This is stored in the relation casesUnderWard32

    casesUnderWard32 = FOREACH (GROUP filterWard ALL) GENERATE COUNT_STAR(filterWard);

Displaying the result in the relation casesUnderWard32. Here there are 4592 number of cases investigated under Ward 32


    DUMP casesUnderWard32;


**Output:**

![enter image description here](https://user-images.githubusercontent.com/29932053/32503288-d782e798-c3aa-11e7-9ff7-eda86d607228.png)

*Pig program to calculate the number of arrests in theft district wise*


Using the relation crimeDataset that was previously loaded with the dataset in the above problem

Filtering the relation crimeDataset so as to get only the cases where the Primary_Type was Theft and were arrested for the crime. This is stored in the relation filterTheft

    filterTheft = FILTER crimeDataset BY Primary_Type=='THEFT' AND Arrest==true;

Grouping all the records with arrested thefts by District. This is stored in the relation groupTheftDistrict

    groupTheftDistrict = GROUP filterTheft BY District;

Generating the count of arrests in each District (groupTheftDistrict) by grouping the records and flattening them by District. This is stored in the relation casesUnderTheftArrestDistrict

    casesUnderTheftArrestDistrict = FOREACH groupTheftDistrict GENERATE FLATTEN(group) AS District,COUNT($1);

Displaying the result in the relation casesUnderTheftArrestDistrict

    DUMP casesUnderTheftArrestDistrict;

**Output:**

![enter image description here](https://user-images.githubusercontent.com/29932053/32504124-197584ba-c3ad-11e7-93d8-d0d09a89d2cb.png)

*Pig program to calculate the number of arrests done between October 2014 and October 2015*

Using the relation crimeDataset that was previously loaded with the dataset in the above problem


Filtering the relation crimeDataset so as to get only the cases where an arrest was made for the crime committed. This is stored in the relation filterArrest

    filterArrest = FILTER crimeDataset BY Arrest=='true';

Generating all the columns in the relation filterArrest and extracting the month and year from the Date column for each row in the relation. This is stored in the relation getDate

    getDate = FOREACH filterArrest GENERATE ID .. Location,SUBSTRING(Date,0,2) AS MONTH,SUBSTRING(Date,6,10) AS YEAR;

Filtering the records in the relation getDate by month between October ‘14 and ‘15 and year as 2014 and 2015. This gives us a list of records that were investigated between October 2014 and October 2015. This is stored in the relation checkDate

    checkDate = FILTER getDate BY((YEAR=='2014') AND ((MONTH=='10')OR(MONTH=='11')OR(MONTH=='12')))OR((YEAR=='2015') AND ((MONTH=='01')OR(MONTH=='02')OR(MONTH=='03')OR(MONTH=='04')OR(MONTH=='05')OR(MONTH=='06')OR(MONTH=='07')OR(MONTH=='08')OR(MONTH=='09')OR(MONTH=='10')));

Generating the count of arrests/records in the relation checkDate by grouping all the records in checkDate. This is stored in the relation countDate

    countDate = FOREACH (GROUP checkDate ALL) GENERATE COUNT_STAR(checkDate);

Displaying the result in the relation countDate


    DUMP countDate;


**Output:**
![enter image description here](https://user-images.githubusercontent.com/29932053/32506680-39a371ba-c3b3-11e7-9324-0c5841d9d2d3.png)

