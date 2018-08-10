# Batch Analytics

## Overview

Process big data jobs in seconds with Azure Data Lake Analytics. There is no infrastructure to worry about because there are no servers, virtual machines, or clusters to wait for, manage, or tune. Instantly scale the processing power, measured in Azure Data Lake Analytics Units (AU), from one to thousands for each job. You only pay for the processing that you use per job.

U-SQL is a simple, expressive, and extensible language that allows you to write code once and have it automatically parallelized for the scale you need. Process petabytes of data for diverse workload categories such as querying, ETL, analytics, machine learning, machine translation, image processing, and sentiment analysis by leveraging existing libraries written in .NET languages, R, or Python.

In this lab Learn how to 

* use Data Lake Analytics to run big data analysis jobs that scale to massive data sets
* how to create and manage batch, real-time, and interactive analytics jobs, and 
* how to query using the U-SQL language

## Task 1: Create Azure Data Lake Analytics Service

Create Data Lakae Analytics service to mine data stored in Data Lake Store.

Click on **Create a resource**

![Create Datalake Analytics Service_](./media/create_resource.png)

Click on **Data + Analytics**

![Create Datalake Analytics Service_](./media/dataanalytics.png)

Click on **Data Lake Analytics**

![Create Datalake Analytics Service_](./media/01_Create_Data_Lake_Analytics_Service.png)

Pick the Data Lake Store where device telemetry data is being stored from Stream Analytics job

![Pick A Store_](./media/02_Create_Data_Lake_Pick_Store.png)

Use existing resource group and click on Create button

![Create Service_](./media/03_Create_Data_Lake_Analytics_Success.png)

## Task 2: Create Sample Data and Install Extensions

Click on Sample scripts 

![Create Sample Data_](./media/04_Create_Data_Lake_Analytics_Sample_Scripts.png)

Click on sample data missing button to create sample data in Data lake Store 

![Create Sample Data_](./media/05_Create_Data_Lake_Analytics_Sample_Data.png)

You should see successful message after data is copied

![Create Sample Data_](./media/06_Create_Data_Lake_Analytics_Sample_Data_Successful.png)

Install Extensions

![Install Extensions_](./media/07_Create_Data_Lake_Analytics_Install_Extensions.png)

Successful Extension installation

![Install Extensions_](./media/08_Create_Data_Lake_Analytics_Install_Extensions_Success.png)

## Task 3: VS Code Integration

Submit Job using VS Code, Try Samples First to Learn through Data Lake Analytics

Install VS Code Extension for Data Lake Analytics

![Install Extensions_](./media/data-lake-tools-for-vscode-extensions.png)

### Run Samples

Run Samples to learn Data Lake Analytics

![Run Samples_](./media/09_VSCode_Open_Sample_Script.png)

Compile Script

![Compile Script_](./media/10_VSCode_Open_Sample_Script_Compile.png)

Click on List more accounts

![Run Samples]./media/11_VSCode_Open_Sample_Script_Compile_Select_Account.png)

Select a Data Lake Analytics Account

![Select Account_](./media/12_VSCode_Open_Sample_Script_Compile_Account.png)

Select master key

![Select master key_](./media/13_VSCode_Open_Sample_Script_Compile_master.png)

Compile as USQL

![Compile USQL_](./media/14_VSCode_Open_Sample_Script_Compile_usql.png)

USQL script should be compiled

![Compiled USQL_](./media/15_VSCode_Open_Sample_Script_Compile_submit.png)

Submit Job To Run

![Submit Job_](./media/16_VSCode_Open_Sample_Script_Submit_Job.png)

Default priority is 1000 and number default number of nodes to run the script are 5

![Submit Job Success_](./media/17_VSCode_Open_Sample_Script_Submit_Job_success.png)

Job Success with Job Analytics

![Job Success_](./media/18_VSCode_Open_Sample_Script_Submit_Job_success_portal.png)

View Input File

![Veiw Input File_](./media/19_VSCode_Open_Sample_Script_Input_Data.png)

View Output File

![View Output File_](./media/20_VSCode_Open_Sample_Script_Output_Data.png)

## Task 4: Create an Analytics Job against MXChip (or Simulator)

Create an analytics job to convert JSON to CSV using U-SQL and Data Lake Analytics

### Create a new mxchip_analytics.usql file in the project

```sql
REFERENCE ASSEMBLY [Newtonsoft.Json];
REFERENCE ASSEMBLY [Microsoft.Analytics.Samples.Formats];

//Extract the Json string using a default Text extractor.

@json =
    EXTRACT jsonString string FROM @"/workshop/streaming/2018/03/{*}/{*}.json" USING Extractors.Tsv(quoting:false);

//Use the JsonTuple function to get the Json Token of the string so it can be parsed later with Json .NET functions

@jsonify = SELECT Microsoft.Analytics.Samples.Formats.Json.JsonFunctions.JsonTuple(jsonString) AS rec FROM @json;

@columnized = SELECT 
            rec["deviceId"] AS deviceId,
            rec["temperature"] AS temperature,
            rec["humidity"] AS humidity,
            rec["time"] AS time
    FROM @jsonify;



//Output the file to a tool of your choice.

OUTPUT @columnized
TO @"/workshop/output/out.csv"
USING Outputters.Csv();

```

Register two assemblies Newtonsoft and Samples.formats. Download the dlls from /libs folder and register

![U-SQL Analytics_](./media/21_Register_Assembly_Command.png)

Select the dlls from the /libs folder

![U-SQL Analytics_](./media/22_Register_Sample_Formats.png)

### Submit Job

Submit Job to convert all JSON files to CSV files

![U-SQL Analytics_](./media/23_SubmitJob.png)

View Jobs

![U-SQL Analytics_](./media/24_Submitted_Jobs.png)
