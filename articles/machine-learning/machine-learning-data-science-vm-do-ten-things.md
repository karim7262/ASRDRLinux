<properties
	pageTitle="Ten things you can do on the Data science Virtual Machine  | Microsoft Azure"
	description="Perform various data exploration and modeling task on the Data science Virtual Machine."
	services="machine-learning"
	documentationCenter=""
	authors="bradsev"
	manager="paulettm"
	editor="cgronlun"  />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/14/2016"
	ms.author="gokuma;weig;bradsev" />

# Ten things you can do on the Data science Virtual Machine

The Microsoft Data Science Virtual Machine (DSVM) is a powerful data science development environment that enables you to perform various data exploration and modeling tasks. The environment comes already built and bundled with several popular data analytics tools that make it easy to get started quickly with your analysis for On-premises, Cloud or hybrid deployments. The DSVM works closely with many Azure services and is able to read and process data that is already stored on Azure, in Azure SQL Data Warehouse, Azure Data Lake, Azure Storage, or in DocumentDB. It can also leverage other analytics tools such as Azure Machine Learning and Azure Data Factory.


In this article we walk you through how to use your DSVM to perform various data science tasks and interact with other Azure services. Here is some  of the things you can do on the DSVM:

1. Explore data and develop models locally on the DSVM using Microsoft R Server, Python
2. Use a Jupyter notebook to experiment with your data on a browser using Python 2, Python 3, Microsoft R an enterprise ready version of R designed for scalability and performance
3. Operationalize models built using R and Python on Azure Machine Learning so client applications can access your models using a simple web services interface
4. Administer your Azure resources using  Azure Portal or Powershell
5. Extend your storage space and share large scale datasets / code across your whole team by creating an Azure File Storage as a mountable drive on your DSVM
6. Share code with your team using Github and access your repository using the pre-installed Git clients - Git Bash, Git GUI.
7. Access various Azure data and analytics services like Azure blob storage, Azure Data Lake, Azure HDInsight (Hadoop), Azure DocumentDB, Azure SQL Data Warehouse & databases
8. Build reports and dashboard using the Power BI Desktop pre-installed on the DSVM and deploy them on the cloud
9. Dynamically scale your DSVM to meet your project needs
10. Install additional tools on your virtual machine   


>[AZURE.NOTE] Additional usage charges will apply for many of the additional data storage and analytics services listed in this article. Please refer to the [Azure Pricing](https://azure.microsoft.com/pricing/) page for details.


**Prerequisites**

- You will need an Azure subscription. You can sign-up for a free trial [here](https://azure.microsoft.com/free/).

- Instructions for provisioning a Data Science Virtual Machine on the Azure Portal are available at [Creating a virtual machine](https://portal.azure.com/#create/microsoft-ads.standard-data-science-vmstandard-data-science-vm).

## 1. Explore data and develop models using Microsoft R Server or Python

You can use languages like R and Python to do your data analytics right on the DSVM.

For R, you can use an IDE called "Revolution R Enterprise 8.0" that can be found on the start menu or the desktop. Microsoft has provided additional libraries on top of the Open source/CRAN-R to enable scalable analytics and the ability to analyze data larger than the memory size allowed by doing parallel chunked analysis. You can also install an R IDE of your choice like [RStudio](https://www.rstudio.com/products/rstudio-desktop/).

For Python, you can use an IDE like Visual Studio Community Edition which has the Python Tools for Visual Studio (PTVS) extension pre-installed. By default, only a basic Python 2.7 is configured on PTVS (without any analytics library like SciKit, Pandas). In order to enable Anaconda Python 2.7 and 3.5, you need to do the following:

* Create custom environments for each version by navigating to **Tools** -> **Python Tools** -> **Python Environments** and then clicking "**+ Custom**" in the Visual Studio 2015 Community Edition
* Give a description and set the environment prefix paths as *c:\anaconda* for Anaconda Python 2.7 OR *c:\anaconda\envs\py35* for Anaconda Python 3.5
* Click **Auto Detect** and then **Apply** to save the environment.

Here is what the custom environment setup looks like in Visual Studio.

![PTVS Setup](./media/machine-learning-data-science-vm-do-ten-things/PTVSSetup.png)

See the [PTVS documentation](https://github.com/Microsoft/PTVS/wiki/Selecting-and-Installing-Python-Interpreters#hey-i-already-have-an-interpreter-on-my-machine-but-ptvs-doesnt-seem-to-know-about-it) for additional details on how to create Python Environments.

Now you are set up to create a new Python project. Navigate to **File** -> **New** -> **Project** -> **Python** and select the type of Python application you are building. You can set the Python environment for the current project to the desired version (Anaconda 2.7 or 3.5): right click the **Python environment**, select **Add/Remove Python Environments**, and then select the desired environment to associate with the project. You can find more information about working with PTVS on the product [documentation](https://github.com/Microsoft/PTVS/wiki) page.

## 2. Using a Jupyter Notebook to explore and model your data with Python or R

The Jupyter Notebook is a powerful environment that provides a browser-based "IDE" for data exploration and modeling. You can use Python 2, Python 3 or R (both Open Source and the Microsoft R Server) in a Jupyter Notebook.

To launch the Jupyter Notebook click on the start menu icon / desktop icon titled **Jupyter Notebook**. On the DSVM you can also browse to "https://localhost:9999/" to access the Jupiter Notebook. If it prompts you for a password, use instructions provided in the ***How to create a strong password on the Jupyter notebook server*** section of the 
[Provision the Microsoft Data Science Virtual Machine](machine-learning-data-science-provision-vm.md) topic to create a strong password to access the Jupyter notebook. 

Once you have opened the notebook, you will see a directory that contains a few example notebooks that are pre-packaged into the DSVM. Now you can:

- click on the notebook to see the code.
- execute each cell by pressing **SHIFT-ENTER**.
- run the entire notebook by clicking on **Cell** -> **Run**
- create a new notebook by clicking on the Jupyter Icon (left top corner) and then clicking **New** button on the right and then choosing the notebook language (also known as kernels).   


>[AZURE.NOTE] Currently we support Python 2.7, Python 3.5 and R. The R kernel supports programming in both Open source R as well as the enterprise scalable Microsoft R Server.   


Once you are in the notebook you can explore your data, build the model, test the model using your choice of libraries.


## 3. Build models using R or Python and Operationalize them using Azure Machine Learning

Once you have built and validated your model the next step is usually to deploy it into production. This allows your client applications to invoke the model predictions on a real time or on a batch mode basis. Azure Machine Learning provides a mechanism to operationalize a model built in either R or Python.

When you operationalize your model in Azure Machine Learning, a web service is exposed that allows clients to make REST calls that pass in input parameters and receive predictions from the model as outputs.   


>[AZURE.NOTE] If you have not yet signed up for AzureML, you can obtain a free workspace or a standard workspace by visiting the [AzureML Studio](https://studio.azureml.net/) home page and clicking on "Get Started".   


### Build and Operationalize Python models

Here is a snippet of code developed in a Python Jupyter Notebook that builds a simple model using the SciKit-learn library.

	#IRIS classification
	from sklearn import datasets
	from sklearn import svm
	clf = svm.SVC()
	iris = datasets.load_iris()
	X, y = iris.data, iris.target
	clf.fit(X, y)

The method used to deploy your python models to Azure Machine Learning wraps the prediction of the model into a function and decorates it with attributes provided by the pre-installed Azure Machine Learning python library that denote your Azure Machine Learning workspace ID, API Key, and the input and return parameters.  

	from azureml import services
	@services.publish(workspaceid, auth_token)
	@services.types(sep_l = float, sep_w = float, pet_l=float, pet_w=float)
	@services.returns(int) #0, or 1, or 2
	def predictIris(sep_l, sep_w, pet_l, pet_w):
 	inputArray = [sep_l, sep_w, pet_l, pet_w]
	return clf.predict(inputArray)

A client can now make calls to the web service. There are convenience wrappers that construct the REST API requests. Here is a sample code to consume the web service.

	# Consume through web service URL and keys
	from azureml import services
	@services.service(url, api_key)
	@services.types(sep_l = float, sep_w = float, pet_l=float, pet_w=float)
	@services.returns(float)
	def IrisPredictor(sep_l, sep_w, pet_l, pet_w):
    pass

	IrisPredictor(3,2,3,4)


>[AZURE.NOTE] The Azure Machine Learning library is only supported on Python 2.7 currently.   


### Build and Operationalize R models

You can deploy R models built on the Data Science Virtual Machine or elsewhere onto Azure Machine Learning in a manner that is similar to how it is done for Python. Her are the steps:

- create a settings.json file as below to provide your workspace ID and auth token.
- write a wrapper for the model's predict function.
- call ```publishWebService``` in the Azure Machine Learning library to pass in the function wrapper.  

Here is the procedure and code snippets that can be used to setup, build, publish, and consume a model as a web service in Azure Machine Learning.

#### Setup

1.  Install the AzureML R package by typing ```install.packages("AzureML")``` in Revolution R Enterprise 8.0 IDE or your R IDE.
2.  Download RTools from [here](https://cran.r-project.org/bin/windows/Rtools/). You need the zip utility in the path (and named zip.exe) to operationalize your R package into AzureML.
3.  Create a settings.json file under a directory called ```.azureml``` under your home directory and enter the parameters from your Azure ML workspace:

settings.json File structure:

	{"workspace":{
	"id"                  : "ENTER YOUR AZUREML WORKSPACE ID",
	"authorization_token" : "ENTER YOUR AZUREML AUTH TOKEN"
	}}


#### Build a model in R and publish it in Azure ML

	library(AzureML)
	ws <- workspace(config="~/.azureml/settings.json")

	if(!require("lme4")) install.packages("lme4")
	library(lme4)
	set.seed(1)
	train <- sleepstudy[sample(nrow(sleepstudy), 120),]
	m <- lm(Reaction ~ Days + Subject, data = train)

	# Define a prediction function to publish based on the model:
	sleepyPredict <- function(newdata){
  		predict(m, newdata=newdata)
	}

	ep <- publishWebService(ws, fun = sleepyPredict, name="sleepy lm", inputSchema = sleepstudy, data.frame=TRUE)

#### Consume the model deployed in Azure ML

To consume the model from a client application, we use the Azure Machine Learning library to look up the published web service by name using the `services` API call to determine the endpoint. Then you just call the `consume` function and pass in the data frame to be predicted.
The following code is used to consume the model published as an Azure Machine Learning web service.


	library(AzureML)
	library(lme4)
	ws <- workspace(config="~/.azureml/settings.json")

	s <-  services(ws, name = "sleepy lm")
	s <- tail(s, 1) # use the last published function, in case of duplicate function names

	ep <- endpoints(ws, s)

	# OK, try this out, and compare with raw data
	ans = consume(ep, sleepstudy)$ans

More information about the Azure Machine Learning R library can be found [here](https://cran.r-project.org/web/packages/AzureML/AzureML.pdf).


## 4. Administer your Azure resources using Azure Portal or Powershell

The DSVM not only allows you to build your analytics solution locally on the virtual machine, but also allows you to access services on Microsoft's Azure cloud. Azure provides several compute, storage, data analytics services and other services that you can administer and access from your DSVM.

To administer your Azure subscription and cloud resources you can use your browser and point to the
[Azure Portal](https://portal.azure.com). You can also use Azure Powershell to administer your Azure subscription and resources via a script.
You can run Azure Powershell from a shortcut on the desktop or from the start menu titled "Microsoft Azure Powershell". Refer to
[Microsoft Azure Powershell documentation](../powershell-azure-resource-manager.md) for more information on how you can administer your Azure subscription and resources using Windows Powershell scripts.


## 5. Extend your storage space with a shared file system

Data scientists can share large datasets, code or other resources within the team. The DSVM itself has about 70GB of space available. To extend your storage, you can use the Azure File Service and either mount it on the DSVM or access it via a REST API.   


>[AZURE.NOTE] The maximum space of the Azure File Service share is 5TB and individual file size limit is 1TB.   


You can use Azure Powershell to create an Azure File Service share. Here is the script to run under Azure PowerShell to create a Azure File service share.

	# Authenticate to Azure.
	Login-AzureRmAccount
	# Select your subscription
	Get-AzureRmSubscription ???SubscriptionName "<your subscription name>" | Select-AzureRmSubscription
	# Create a new resource group.
	New-AzureRmResourceGroup -Name <dsvmdatarg>
	# Create a new storage account. You can reuse existing storage account if you wish.
	New-AzureRmStorageAccount -Name <mydatadisk> -ResourceGroupName <dsvmdatarg> -Location "<Azure Data Center Name For eg. South Central US>" -Type "Standard_LRS"
	# Set your current working storage account
	Set-AzureRmCurrentStorageAccount ???ResourceGroupName "<dsvmdatarg>" ???StorageAccountName <mydatadisk>

	# Create a Azure File Service Share
	$s = New-AzureStorageShare <<teamsharename>>
	# Create a directory under the FIle share. You can give it any name
	New-AzureStorageDirectory -Share $s -Path <directory name>
	# List the share to confirm that everything worked
	Get-AzureStorageFile -Share $s


Now that you have created an Azure file share, you can mount it in any virtual machine in Azure. It is highly recommended that the VM is in same Azure data center as the storage account to avoid latency and data transfer charges. Here is the commands to mount the drive on the DSVM that you can run on Azure Powershell.


	# Get storage key of the storage account that has the Azure file share from Azure portal. Store it securely on the VM to avoid prompted in next command.
	cmdkey /add:<<mydatadisk>>.file.core.windows.net /user:<<mydatadisk>> /pass:<storage key>

	# Mount the Azure file share as Z: drive on the VM. You can chose another drive letter if you wish
	net use z:  \\<mydatadisk>.file.core.windows.net\<<teamsharename>>


Now you can access this drive as you would any normal drive on the VM.

## 6. Share code with your team using Github

Github is a code repository where you can find a lot of sample code and sources for different tools using various technologies shared by the developer community. It uses Git as the technology to track and store versions of the code files. Github is also a platform where you can create your own repository to store your team's shared code and documentation, implement version control and also control who have access to view and contribute code. Please visit the [Github help pages](https://help.github.com/) for more information on using Git. You can use Github as one of the ways to collaborate with your team, use code developed by the community and contribute code back to the community.

The DSVM already comes loaded with client tools on both command line as well GUI to access Github repository. The command line tool to work with Git and Github is called Git Bash. Visual Studio installed on the DSVM has the Git extensions. You can find start-up icons for these tools on the start menu and the desktop.

To download code from a Github repository you will use the ```git clone``` command. For example to download data science repository published by Microsoft into the current directory you can run the following command once you are in ```git-bash```.

	git clone https://github.com/Azure/Azure-MachineLearning-DataScience.git

In Visual Studio, you can do the same clone operation. The screen-shot below shows how to access Git and Github tools in Visual Studio.

![Git in Visual Studio](./media/machine-learning-data-science-vm-do-ten-things/VSGit.PNG)


You can find more information on using Git to work with your Github repository from several resources available on github.com. The [cheat sheet](https://training.github.com/kit/downloads/github-git-cheat-sheet.pdf) is a useful reference.


## 7. Access various Azure data and analytics services

### Azure Blob

Azure blob is a reliable, economical cloud storage for data big and small. Let us look at how you can move data to Azure Blob and access data stored in a Azure Blob.

**Prerequisite**

- **Create your Azure Blob storage account from [Azure Portal](https://portal.azure.com).**

![Create_Azure_Blob](./media/machine-learning-data-science-vm-do-ten-things/Create_Azure_Blob.PNG)


- Confirm that the pre-installed command line AzCopy tool is found at ```C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy.exe```. You can add the directory containing the azcopy.exe to your PATH environment variable to avoid typing the full command path when running this tool. For more info on AzCopy tool please refer to [AzCopy documentation](../storage/storage-use-azcopy.md)

- Start the Azure Storage Explorer from a icon on the VM desktop. You can also find it in the directory ***C:\Program Files (x86)\Neudesic\Azure Storage Explorer 6***.  


![AzureStorageExplorer_v4](./media/machine-learning-data-science-vm-do-ten-things/AzureStorageExplorer_v4.png)


**Move data from VM to Azure Blob: AzCopy**

To move data between your local files and blob storage, you can use AzCopy in command line or PowerShell:

	AzCopy /Source:C:\myfolder /Dest:https://<mystorageaccount>.blob.core.windows.net/<mycontainer> /DestKey:<storage account key> /Pattern:abc.txt

Replace **C:\myfolder** to the path where your file is stored, **mystorageaccount** to your blob storage account name, **mycontainer** to the container name, **storage account key** to your blob storage access key. You can find your storage account credentials in [Azure Portal](https://portal.azure.com).

![StorageAccountCredential_v2](./media/machine-learning-data-science-vm-do-ten-things/StorageAccountCredential_v2.png)

Run AzCopy command in PowerShell or from a command prompt. Here is some example usage of AzCopy command:


	# Copy *.sql from local machine to a Azure Blob
	"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:"c:\Aaqs\Data Science Scripts" /Dest:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /DestKey:[ENTER STORAGE KEY] /S /Pattern:*.sql

	# Copy back all files from Azure Blob container to Local machine

	"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Dest:"c:\Aaqs\Data Science Scripts\temp" /Source:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /SourceKey:[ENTER STORAGE KEY] /S



Once you run your AzCopy command to copy to an Azure blob you see your file shows up in Azure Storage Explorer shortly.

![AzCopy_run_finshed_Storage_Explorer_v3](./media/machine-learning-data-science-vm-do-ten-things/AzCopy_run_finshed_Storage_Explorer_v3.png)


**Move data from VM to Azure Blob: Azure Storage Explorer**

You can also upload data from the local file in your VM using Azure Storage Explorer:


![](./media/machine-learning-data-science-vm-do-ten-things/AzureStorageExplorer_upload_v2.png)


**Read data from Azure Blob: AML reader module**

In Azure Machine Learning Studio you can use an **Import Data module** to read data from your blob.


![AML_ReaderBlob_Module_v3](./media/machine-learning-data-science-vm-do-ten-things/AML_ReaderBlob_Module_v3.png)


**Read data from Azure Blob: Python ODBC**

You can use **BlobService** library to read data directly from blob in a Jupyter Notebook or Python program.

First, import required packages:

	import pandas as pd
	from pandas import Series, DataFrame
	import numpy as np
	import matplotlib.pyplot as plt
	from time import time
	import pyodbc
	import os
	from azure.storage.blob import BlobService
	import tables
	import time
	import zipfile
	import random

Then plug in your Azure Blob account credentials and read data from Blob:

    CONTAINERNAME = 'xxx'
	STORAGEACCOUNTNAME = 'xxxx'
	STORAGEACCOUNTKEY = 'xxxxxxxxxxxxxxxx'
	BLOBNAME = 'nyctaxidataset/nyctaxitrip/trip_data_1.csv'
	localfilename = 'trip_data_1.csv'
	LOCALDIRECTORY = os.getcwd()
	LOCALFILE =  os.path.join(LOCALDIRECTORY, localfilename)

	#download from blob
	t1 = time.time()
	blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
	blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILE)
	t2 = time.time()
	print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))

	#unzipping downloaded files if needed
	#with zipfile.ZipFile(ZIPPEDLOCALFILE, "r") as z:
	#    z.extractall(LOCALDIRECTORY)

	df1 = pd.read_csv(LOCALFILE, header=0)
	df1.columns = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime','passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude']
	print 'the size of the data is: %d rows and  %d columns' % df1.shape

The data is read in as a data frame:

![IPNB_data_readin](./media/machine-learning-data-science-vm-do-ten-things/IPNB_data_readin.PNG)


### Azure Data Lake

Azure Data Lake Storage is a hyper-scale repository for big data analytics workloads and compatible with Hadoop Distributed File System (HDFS), It works with both the Hadoop ecosystem and the Azure Data Lake Analytics. We show how you can move data into the Azure Data Lake Store and run analytics using Azure Data Lake Analytics.

**Prerequisite**

- Create your Azure Data Lake Analytics in [Azure Portal](https://portal.azure.com).

![Azure_Data_Lake_Create_v2](./media/machine-learning-data-science-vm-do-ten-things/Azure_Data_Lake_Create_v2.png)


- The  **Azure Data Lake Tools** in **Visual Studio** found at this  [link](https://www.microsoft.com/download/details.aspx?id=49504) is already installed on the Visual Studio Community Edition which is on the virtual machione. After starting Visual Studio and logging in your Azure subscription, you will see your Azure Data Analytics account and storage in the left panel of Visual Studio.

![Azure_Data_Lake_PlugIn_v2](./media/machine-learning-data-science-vm-do-ten-things/Azure_Data_Lake_PlugIn_v2.PNG)


**Move data from VM to Data Lake: Azure Data Lake Explorer**

You can use **Azure Data Lake Explorer** to upload data from the local files in your Virtual Machine to Data Lake storage.

![Azure_Data_Lake_UploadData](./media/machine-learning-data-science-vm-do-ten-things/Azure_Data_Lake_UploadData.PNG)

You can also build a data pipeline to productionize your data movement to or from Azure Data Lake using the [Azure Data Factory(ADF)](https://azure.microsoft.com/services/data-factory/). We refer you to this [article](https://azure.microsoft.com/blog/creating-big-data-pipelines-using-azure-data-lake-and-azure-data-factory/) to guide you through the steps to build the data pipelines.

**Read data from Azure Blob to Data Lake: U-SQL**

If your data resides in Azure Blob storage, you can directly read data from Azure storage blob in U-SQL query. Before composing your U-SQL query, make sure your blob storage account is linked to your Azure Data Lake. Go to **Azure Portal**, find your Azure Data Lake Analytics dashboard, click **Add Data Source**, select storage type to **Azure Storage** and plug in your Azure Storage Account Name and Key. Then you will be able to reference the data stored in the storage account.

![](./media/machine-learning-data-science-vm-do-ten-things/Link_Blob_to_ADLA_v2.PNG)


In Visual Studio, you can read data from blob storage, do some data manipulation, feature engineering, and output the resulting data to either Azure Data Lake or Azure Blob Storage. When you reference the data in blob storage, use **wasb://**; when you reference the data in Azure Data Lake, use **swbhdfs://**

![](./media/machine-learning-data-science-vm-do-ten-things/USQL_Read_Blob_v2.PNG)

You may use the following U-SQL queries in Visual Studio:

	@a =
	    EXTRACT medallion string,
	            hack_license string,
	            vendor_id string,
	            rate_code string,
	            store_and_fwd_flag string,
	            pickup_datetime string,
	            dropoff_datetime string,
	            passenger_count int,
	            trip_time_in_secs double,
	            trip_distance double,
	            pickup_longitude string,
	            pickup_latitude string,
	            dropoff_longitude string,
	            dropoff_latitude string

	    FROM "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Input Data File Name>"
	    USING Extractors.Csv();

	@b =
	    SELECT vendor_id,
	    COUNT(medallion) AS cnt_medallion,
	    SUM(passenger_count) AS cnt_passenger,
	    AVG(trip_distance) AS avg_trip_dist,
	    MIN(trip_distance) AS min_trip_dist,
	    MAX(trip_distance) AS max_trip_dist,
	    AVG(trip_time_in_secs) AS avg_trip_time
	    FROM @a
	    GROUP BY vendor_id;

	OUTPUT @b   
	TO "swebhdfs://<Azure Data Lake Storage Account Name>.azuredatalakestore.net/<Folder Name>/<Output Data File Name>"
	USING Outputters.Csv();

	OUTPUT @b   
	TO "wasb://<Container name>@<Azure Blob Storage Account Name>.blob.core.windows.net/<Output Data File Name>"
	USING Outputters.Csv();



After your query is submitted to the server, a diagram showing the status of your job will be displayed.

![](./media/machine-learning-data-science-vm-do-ten-things/USQL_Job_Status.PNG)


**Query data in Data Lake: U-SQL**

After the dataset is ingested into Azure Data Lake, you can use [U-SQL language](../data-lake-analytics/data-lake-analytics-u-sql-get-started.md) to query and explore the data. U-SQL language is similar to T-SQL, but combines some features from C# so that users can write customized modules, User Defined Functions, and etc. You can use the scripts in the previous step.

After the query is submitted to server, tripdata_summary.CSV can be found shortly in **Azure Data Lake Explorer**, you may preview the data by right click the file.

![](./media/machine-learning-data-science-vm-do-ten-things/USQL_create_summary.png)

To see the file information:

![](./media/machine-learning-data-science-vm-do-ten-things/USQL_tripdata_summary.png)


### HDInsight Hadoop Clusters

Azure HDInsight is a managed Apache Hadoop, Spark, HBase, and Storm service on the cloud. You can work easily with Azure HDInsight clusters from the data science virtual machine.

**Prerequisite**

- Create your Azure Blob storage account from [Azure Portal](https://portal.azure.com). This storage account is used to store data for HDInsight clusters.

![](./media/machine-learning-data-science-vm-do-ten-things/Create_Azure_Blob.PNG)

- Customize Azure HDInsight Hadoop Clusters from [Azure Portal](machine-learning-data-science-customize-hadoop-cluster.md)

  - You must link the storage account created with your HDInsight cluster when it is created. This storage account is used for accessing data that can be processed within the cluster.

![](./media/machine-learning-data-science-vm-do-ten-things/Create_HDI_v4.PNG)

  - You must enable **Remote Access** to the head node of the cluster after it is created. Remember the remote access credentials you specify here (different from those specified for the cluster at its creation): you will need them below.

![](./media/machine-learning-data-science-vm-do-ten-things/Create_HDI_dashboard_v3.PNG)

  - Create an Azure ML workspace. Your Machine Learning Experiments will be stored in this ML workspace. Select the highlighted options in Portal as shown in the screenshot below.

![](./media/machine-learning-data-science-vm-do-ten-things/Create_ML_Space.PNG)


  - Then enter the parameters for your Azure ML workspace

![](./media/machine-learning-data-science-vm-do-ten-things/Create_ML_Space_step2_v2.PNG)

  - Upload data using IPython Notebook. First import required packages, plug in credentials, create a db in your storage account, then load data to HDI clusters.


		#Import required Packages
		import pyodbc
		import time as time
		import json
		import os
		import urllib
		import urllib2
		import warnings
		import re
		import pandas as pd
		import matplotlib.pyplot as plt
		from azure.storage.blob import BlobService
		warnings.filterwarnings("ignore", category=UserWarning, module='urllib2')


		#Create the connection to Hive using ODBC
		SERVER_NAME='xxx.azurehdinsight.net'
		DATABASE_NAME='nyctaxidb'
		USERID='xxx'
		PASSWORD='xxxx'
		DB_DRIVER='Microsoft Hive ODBC Driver'
		driver = 'DRIVER={' + DB_DRIVER + '}'
		server = 'Host=' + SERVER_NAME + ';Port=443'
		database = 'Schema=' + DATABASE_NAME
		hiveserv = 'HiveServerType=2'
		auth = 'AuthMech=6'
		uid = 'UID=' + USERID
		pwd = 'PWD=' + PASSWORD
		CONNECTION_STRING = ';'.join([driver,server,database,hiveserv,auth,uid,pwd])
		connection = pyodbc.connect(CONNECTION_STRING, autocommit=True)
		cursor=connection.cursor()


		#Create Hive database and tables
		queryString = "create database if not exists nyctaxidb;"
		cursor.execute(queryString)

		queryString = """
		                create external table if not exists nyctaxidb.trip
		                (
		                    medallion string,
		                    hack_license string,
		                    vendor_id string,
		                    rate_code string,
		                    store_and_fwd_flag string,
		                    pickup_datetime string,
		                    dropoff_datetime string,
		                    passenger_count int,
		                    trip_time_in_secs double,
		                    trip_distance double,
		                    pickup_longitude double,
		                    pickup_latitude double,
		                    dropoff_longitude double,
		                    dropoff_latitude double)  
		                PARTITIONED BY (month int)
		                ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\\n'
		                STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/trip' TBLPROPERTIES('skip.header.line.count'='1');
		            """
		cursor.execute(queryString)

		queryString = """
		                create external table if not exists nyctaxidb.fare
		                (
		                    medallion string,
		                    hack_license string,
		                    vendor_id string,
		                    pickup_datetime string,
		                    payment_type string,
		                    fare_amount double,
		                    surcharge double,
		                    mta_tax double,
		                    tip_amount double,
		                    tolls_amount double,
		                    total_amount double)
		                PARTITIONED BY (month int)
		                ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\\n'
		                STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/fare' TBLPROPERTIES('skip.header.line.count'='1');
		            """
		cursor.execute(queryString)


		#Upload data from blob storage to HDI cluster
		for i in range(1,13):
		    queryString = "LOAD DATA INPATH 'wasb:///nyctaxitripraw2/trip_data_%d.csv' INTO TABLE nyctaxidb2.trip PARTITION (month=%d);"%(i,i)
		    cursor.execute(queryString)
		    queryString = "LOAD DATA INPATH 'wasb:///nyctaxifareraw2/trip_fare_%d.csv' INTO TABLE nyctaxidb2.fare PARTITION (month=%d);"%(i,i)  
		    cursor.execute(queryString)


- Alternately,  you can follow this [walkthrough](machine-learning-data-science-process-hive-walkthrough.md) to upload NYC Taxi data to HDI cluster. Major steps include:

	- AzCopy: download zipped CSV's from public blob to your local folder
	- AzCopy: upload unzipped CSV's from local folder to HDI cluster
	- Log into the head node of Hadoop cluster and prepare for exploratory data analysis

After the data is loaded to HDI cluster, you can check your data in Azure Storage Explorer. And you have a database nyctaxidb created in HDI cluster.

![](./media/machine-learning-data-science-vm-do-ten-things/Upload_Data_to_HDI_cluster_Azure_Explorer.PNG)


**Data exploration: Hive Queries in Python**

Since the data is in Hadoop cluster, you can use the pyodbc package to connect to Hadoop Clusters and query database using Hive to do exploration and feature engineering. You can view the existing tables we created in the prerequisite step.

	queryString = """
	    show tables in nyctaxidb2;
	    """
	pd.read_sql(queryString,connection)


![](./media/machine-learning-data-science-vm-do-ten-things/Python_View_Existing_Tables_Hive_v3.PNG)

Let's look at the number of records in each month and the frequencies of tipped or not in the trip table:

	queryString = """
	    select month, count(*) from nyctaxidb.trip group by month;
	    """
	results = pd.read_sql(queryString,connection)

	%matplotlib inline

	results.columns = ['month', 'trip_count']
	df = results.copy()
	df.index = df['month']
	df['trip_count'].plot(kind='bar')


![](./media/machine-learning-data-science-vm-do-ten-things/Exploration_Number_Records_by_Month_v3.PNG)


	queryString = """
	    SELECT tipped, COUNT(*) AS tip_freq
	    FROM
	    (
	        SELECT if(tip_amount > 0, 1, 0) as tipped, tip_amount
	        FROM nyctaxidb.fare
	    )tc
	    GROUP BY tipped;
	    """
	results = pd.read_sql(queryString,connection)

	results.columns = ['tipped', 'trip_count']
	df = results.copy()
	df.index = df['tipped']
	df['trip_count'].plot(kind='bar')


![](./media/machine-learning-data-science-vm-do-ten-things/Exploration_Frequency_tip_or_not_v3.PNG)

We can also compute the distance between pickup location and dropoff location and then compare it to the trip distance.

	queryString = """
	                select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, trip_distance, trip_time_in_secs,
	                    3959*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
	                    *radians(180)/180/2),2)-cos(pickup_latitude*radians(180)/180)
	                    *cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2)))
	                    /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*radians(180)/180/2),2)
	                    +cos(pickup_latitude*radians(180)/180)*cos(dropoff_latitude*radians(180)/180)*
	                    pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2))) as direct_distance
	                    from nyctaxidb.trip
	                    where month=1
	                        and pickup_longitude between -90 and -30
	                        and pickup_latitude between 30 and 90
	                        and dropoff_longitude between -90 and -30
	                        and dropoff_latitude between 30 and 90;
	            """
	results = pd.read_sql(queryString,connection)
	results.head(5)


![](./media/machine-learning-data-science-vm-do-ten-things/Exploration_compute_pickup_dropoff_distance_v2.PNG)

	results.columns = ['pickup_longitude', 'pickup_latitude', 'dropoff_longitude',
	                   'dropoff_latitude', 'trip_distance', 'trip_time_in_secs', 'direct_distance']
	df = results.loc[results['trip_distance']<=100] #remove outliers
	df = df.loc[df['direct_distance']<=100] #remove outliers
	plt.scatter(df['direct_distance'], df['trip_distance'])


![](./media/machine-learning-data-science-vm-do-ten-things/Exploration_direct_distance_trip_distance_v2.PNG)


Now let's prepare a down-sampled (1%) set of data for modeling. We can use this data in AML reader module.


		queryString = """
	    create  table if not exists nyctaxi_downsampled_dataset_testNEW (
	    medallion string,
	    hack_license string,
	    vendor_id string,
	    rate_code string,
	    store_and_fwd_flag string,
	    pickup_datetime string,
	    dropoff_datetime string,
	    pickup_hour string,
	    pickup_week string,
	    weekday string,
	    passenger_count int,
	    trip_time_in_secs double,
	    trip_distance double,
	    pickup_longitude double,
	    pickup_latitude double,
	    dropoff_longitude double,
	    dropoff_latitude double,
	    direct_distance double,
	    payment_type string,
	    fare_amount double,
	    surcharge double,
	    mta_tax double,
	    tip_amount double,
	    tolls_amount double,
	    total_amount double,
	    tipped string,
	    tip_class string
	    )
	    row format delimited fields terminated by ','
	    lines terminated by '\\n'
	    stored as textfile;
		"""
		cursor.execute(queryString)

		--- now insert contents of the join into the above internal table

		queryString = """
	    insert overwrite table nyctaxi_downsampled_dataset_testNEW
	    select
	    t.medallion,
	    t.hack_license,
	    t.vendor_id,
	    t.rate_code,
	    t.store_and_fwd_flag,
	    t.pickup_datetime,
	    t.dropoff_datetime,
	    hour(t.pickup_datetime) as pickup_hour,
	    weekofyear(t.pickup_datetime) as pickup_week,
	    from_unixtime(unix_timestamp(t.pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),'u') as weekday,
	    t.passenger_count,
	    t.trip_time_in_secs,
	    t.trip_distance,
	    t.pickup_longitude,
	    t.pickup_latitude,
	    t.dropoff_longitude,
	    t.dropoff_latitude,
	    t.direct_distance,
	    f.payment_type,
	    f.fare_amount,
	    f.surcharge,
	    f.mta_tax,
	    f.tip_amount,
	    f.tolls_amount,
	    f.total_amount,
	    if(tip_amount>0,1,0) as tipped,
	    if(tip_amount=0,0,
	    if(tip_amount>0 and tip_amount<=5,1,
	    if(tip_amount>5 and tip_amount<=10,2,
	    if(tip_amount>10 and tip_amount<=20,3,4)))) as tip_class
	    from
	    (
	    select
	    medallion,
	    hack_license,
	    vendor_id,
	    rate_code,
	    store_and_fwd_flag,
	    pickup_datetime,
	    dropoff_datetime,
	    passenger_count,
	    trip_time_in_secs,
	    trip_distance,
	    pickup_longitude,
	    pickup_latitude,
	    dropoff_longitude,
	    dropoff_latitude,
	    3959*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
	    radians(180)/180/2),2)-cos(pickup_latitude*radians(180)/180)
	    *cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2)))
	    /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*radians(180)/180/2),2)
	    +cos(pickup_latitude*radians(180)/180)*cos(dropoff_latitude*radians(180)/180)*pow(sin((dropoff_longitude-pickup_longitude)*radians(180)/180/2),2))) as direct_distance,
	    rand() as sample_key

	    from trip
	    where pickup_latitude between 30 and 90
	        and pickup_longitude between -90 and -30
	        and dropoff_latitude between 30 and 90
	        and dropoff_longitude between -90 and -30
	    )t
	    join
	    (
	    select
	    medallion,
	    hack_license,
	    vendor_id,
	    pickup_datetime,
	    payment_type,
	    fare_amount,
	    surcharge,
	    mta_tax,
	    tip_amount,
	    tolls_amount,
	    total_amount
	    from fare
	    )f
	    on t.medallion=f.medallion and t.hack_license=f.hack_license and t.pickup_datetime=f.pickup_datetime
	    where t.sample_key<=0.01
		"""
		cursor.execute(queryString)

After a while, you can see the data has been loaded in Hadoop clusters:

	queryString = """
	    select * from nyctaxi_downsampled_dataset limit 10;
	    """
	cursor.execute(queryString)
	pd.read_sql(queryString,connection)


![](./media/machine-learning-data-science-vm-do-ten-things/DownSample_Data_For_Modeling_v2.PNG)


**Read data from HDI using AML: reader module**

You may also use the **reader** module in AML studio to access the database in Hadoop cluster. Plug in the credentials of your HDI clusters and Azure Storage Account and you will be able to build machine learning models using database in HDI clusters.

![](./media/machine-learning-data-science-vm-do-ten-things/AML_Reader_Hive.PNG)

The scored dataset can then be viewed:

![](./media/machine-learning-data-science-vm-do-ten-things/AML_Model_Results.PNG)


### Azure SQL Data Warehouse & databases

Azure SQL Data Warehouse is an elastic data warehouse as a service with enterprise-class SQL Server experience.

You can provision your Azure SQL Data Warehouse by following the instructions provided in this [article](../sql-data-warehouse/sql-data-warehouse-get-started-provision.md). Once you provision your Azure SQL Data Warehouse, you can use this [walkthrough](machine-learning-data-science-process-sqldw-walkthrough.md) to do data upload, exploration and modeling using data within the SQL Data Warehouse.

#### Azure DocumentDB

Azure DocumentDB is a NoSQL database in the cloud. It allows you to work with documents like JSON and allows you to store and query the documents.

You need to do the following per-requisites steps to access DocumentDB from the DSVM.

1. Install DocumentDB Python SDK (Run ```pip install pydocumentdb``` from command prompt)
1. Create DocumentDB account and Document DB database from [Azure portal](https://portal.azure.com)
1. Download "DocumentDB Migration Tool" from [here](http://www.microsoft.com/downloads/details.aspx?FamilyID=cda7703a-2774-4c07-adcc-ad02ddc1a44d) and extract to a directory of your choice
1. Import JSON data (volcano data) stored on a [public blob](https://cahandson.blob.core.windows.net/samples/volcano.json) into DocumentDB with following command parameters to the migration tool (dtui.exe from the directory where you installed the DocumentDB Migration Tool). Enter the source and target location parameters from below.

	/s:JsonFile /s.Files:https://cahandson.blob.core.windows.net/samples/volcano.json /t:DocumentDBBulk /t.ConnectionString:AccountEndpoint=https://[DocDBAccountName].documents.azure.com:443/;AccountKey=[[KEY];Database=volcano /t.Collection:volcano1

Once you import the data, you can go to Jupyter and open the notebook titled *DocumentDBSample* which contains python code to access DocumentDB and do some basic querying. You can learn more about DocumentDB by visiting the service [documentation page](https://azure.microsoft.com/documentation/learning-paths/documentdb/)


## 8. Build reports and dashboard using the Power BI Desktop

Let us visualize the Volcano JSON file we saw in the DocumentDB example above in Power BI to gain visual insights into the data. Detailed steps are available in the [Power BI article](../documentdb/documentdb-powerbi-visualize.md). The high level steps are below :

1. Open Power BI Desktop and do "Get Data". Specify the URL as: https://cahandson.blob.core.windows.net/samples/volcano.json
2. You should see the JSON records imported as a list
3. Convert the list to a table so PowerBI can work with the same
4. Expand the columns by clicking on the expand icon (the one with the "left arrow and a right arrow" icon on the right of the column)
5. Notice that location is a "Record" field. Expand the record and select only the coordinates. Coordinate is a list column
6. Add a new column to convert the list coordinate column into a comma separate LatLong column contatenating the two elements in the coordinate list field using the formula ```Text.From([coordinates]{1})&","&Text.From([coordinates]{0})```.
7. Finally convert the ```Elevation``` column to Decimal and select the **Close** and **Apply**.

Instead of steps above, you can paste the following code that scripts out the steps above in the Advanced Editor in PowerBI that allows you to write the data transformations in a query language.


	let
	    Source = Json.Document(Web.Contents("https://cahandson.blob.core.windows.net/samples/volcano.json")),
	    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
	    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}, {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}),
	    #"Expanded Location" = Table.ExpandRecordColumn(#"Expanded Column1", "Location", {"coordinates"}, {"coordinates"}),
	    #"Added Custom" = Table.AddColumn(#"Expanded Location", "LatLong", each Text.From([coordinates]{1})&","&Text.From([coordinates]{0})),
	    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Elevation", type number}})
	in
	    #"Changed Type"



You now have the data in your Power BI data model. Your Power BI desktop should look as shown below.

![](./media/machine-learning-data-science-vm-do-ten-things/PowerBIVolcanoData.png)

You can start building reports and visualizations using the data model. You can follow the steps in this [Power BI article](../documentdb/documentdb-powerbi-visualize.md#build-the-reports) to build a report. The end result will be a report that looks like the following.

![Power BI Desktop Report View - Power BI connector](./media/machine-learning-data-science-vm-do-ten-things/power_bi_connector_pbireportview2.png)

## 9. Dynamically scale your DSVM to meet your project needs

You can scale up and down the DSVM to meet your project needs. If you don't need to use the VM in the evening or weekends, you can just shutdown the VM from the [Azure Portal](https://portal.azure.com).

>[AZURE.NOTE]  You will incur compute charges if you use just the Operating system shutdown button on the VM.  

If you need to handle some large scale analysis and need more CPU and/or memory and/or disk capacity you can find a large choice of VM sizes in terms of CPU cores, memory capacity and disk types (including Solid state drives) that meet your compute and budgetary needs. The full list of VMs along with their hourly compute pricing is available on the [Azure Virtual Machines Pricing](https://azure.microsoft.com/pricing/details/virtual-machines/) page.

Similarly, if your needs for VM processing capacity reduces (for example: you moved a major workload to a Hadoop or a Spark cluster), you can scale down the cluster from the [Azure Portal](https://portal.azure.com) and going to the settings of your VM instance. Here is a screenshot.


![](./media/machine-learning-data-science-vm-do-ten-things/VMScaling.PNG)


## 10. Install additional tools on your virtual machine

We have packaged several tools that we believe will be able to address many of the common data analytics needs and that should save you time by avoiding having to install and configure your environments one by one and save you money by paying only for resources that you use.

You can leverage other Azure data and analytics services profiled in this article to enhance your analytics environment. We understand that in some cases your needs may require additional tools, including some proprietary third party tools. You have full administrative access on the virtual machine to install new tools you need. You can also install additional packages in Python and R that are not pre-installed. For Python  you can use either ```conda``` or ```pip```. For R you can use the ```install.packages()``` in the R console or use the IDE and choose "**Packages** -> **Install Packages...**".

## Summary
These are just some of the things you can do on the Microsoft Data Science Virtual Machine. There are many more things you can do to make it an effective analytics environment.
