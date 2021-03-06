<properties 
    pageTitle="Import a BACPAC file to create a new Azure SQL database using PowerShell | Microsoft Azure" 
    description="Import a BACPAC file to create a new Azure SQL database using PowerShell" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="data-management" 
    ms.date="07/06/2016"
    ms.author="sstein"/>

# Import a BACPAC file to create a new Azure SQL database using PowerShell

**Single database**

> [AZURE.SELECTOR]
- [Azure Portal](sql-database-import.md)
- [PowerShell](sql-database-import-powershell.md)
- [SSMS](sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)
- [SqlPackage](sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage.md)

This article provides directions for creating an Azure SQL database by importing a BACPAC with PowerShell.

A BACPAC is a .bacpac file that contains a database schema and data. For details, see Backup Package (.bacpac) in [Data-tier Applications](https://msdn.microsoft.com/library/ee210546.aspx).

The database is created from a BACPAC imported from an Azure storage blob container. If you don't have a .bacpac file in Azure storage you can create one by following the steps in [Archive an Azure SQL database to a BACPAC file using PowerShell](sql-database-export-powershell.md).

> [AZURE.NOTE] Azure SQL Database automatically creates and maintains backups for every user database that you can restore. For details, see [SQL Database automated backups](sql-database-automated-backups.md).


To import a SQL database you need the following:

- An Azure subscription. If you need an Azure subscription simply click **FREE TRIAL** at the top of this page, and then come back to finish this article.
- A .bacpac file (BACPAC) of the database you want to import. The BACPAC needs to be in an [Azure Storage account (classic)](../storage/storage-create-storage-account.md) blob container.



[AZURE.INCLUDE [Start your PowerShell session](../../includes/sql-database-powershell.md)]



## Setup the variables for your environment

There are a few variables where you need to replace the example values with the specific values for your database and your storage account.

The server name needs to be a server that currently exists in the subscription selected in the previous step and is the server you want the database to be created in. Note that importing a database directly into an elastic pool is not supported, but you can first import into a single database and then move the database into a pool.

The database name is the name you want for the new database.

    $ResourceGroupName = "resource group name"
    $ServerName = "server name"
    $DatabaseName = "database name"


The following variables are from the storage account where your BACPAC is located. In the [Azure portal](https://portal.azure.com) browse to your storage account to get these values. You can find the primary access key by clicking **All settings** and then **Keys** from your storage account's blade.

The blob name is the name of an existing .bacpac file that you want to create the database from. You need to include the .bacpac extension.

    $StorageName = "storageaccountname"
    $StorageKeyType = "storageKeyType"
    $StorageUri = "http://storageaccountname.blob.core.windows.net/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"


Running the **Get-Credential** cmdlet opens a window asking for your username and password. Enter the admin login and password for the SQL Database server ($ServerName from above), and not the username and password for your Azure account.

    $credential = Get-Credential


## Import the database

This command submits an import database request to the service. Depending on the size of your database the import operation may take some time to complete.

    $importRequest = New-AzureRmSqlDatabaseImport ???ResourceGroupName $ResourceGroupName ???ServerName $ServerName ???DatabaseName $DatabaseName ???StorageKeytype $StorageKeyType ???StorageKey $StorageKey StorageUri $StorageUri ???AdministratorLogin $credential.UserName ???AdministratorPassword $credential.Password ???Edition Standard ???ServiceObjectiveName S0 -DatabaseMxSize 50000
    

## Monitor the progress of the operation

After running **New-AzureRmSqlDatabaseImport** you can check the status of the request. 

Checking status immediately after the request will usually return a status of **Pending**, or **Running** and will provide current percent complete so you can run this multiple times until you see **Status : Completed** in the output. 

Running this command will prompt you for a password. Enter the admin login and password for your SQL server.


    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest .OperationStatusLink
 


## SQL Database PowerShell import script


    $ResourceGroupName = "resourceGroupName"
    $ServerName = "servername"
    $DatabaseName = "databasename"

    $StorageName = "storageaccountname"
    $StorageKeyType = "storageKeyType"
    $StorageUri = "http://storageaccountname.blob.core.windows.net/containerName/filename.bacpac"
    $StorageKey = "primaryaccesskey"

    $credential = Get-Credential

    $importRequest = New-AzureRmSqlDatabaseImport ???ResourceGroupName $ResourceGroupName ???ServerName $ServerName ???DatabaseName $DatabaseName ???StorageKeytype $StorageKeyType ???StorageKey $StorageKey  StorageUri $StorageUri ???AdministratorLogin $credential.UserName ???AdministratorPassword $credential.Password ???Edition Standard ???ServiceObjectiveName S0 -DatabaseMxSize 50000
 
    Get-AzureRmSqlDatabaseImportExportStatus -OperationStatusLink $importRequest .OperationStatusLink

    

## Next steps

- [Connect to SQL Database with SQL Server Management Studio and perform a sample T-SQL query](sql-database-connect-query-ssms.md)




## Additional resources

- [Business Continuity Overview](sql-database-business-continuity.md)
- [Disaster Recovery Drills](sql-database-disaster-recovery-drills.md)
- [SQL Database documentation](https://azure.microsoft.com/documentation/services/sql-database/)
