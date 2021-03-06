<properties 
pageTitle="Common startup tasks for Cloud Services | Microsoft Azure" 
description="Provides some examples of common startup tasks you may want to perform in your cloud services web role or worker role." 
services="cloud-services" 
documentationCenter="" 
authors="Thraka" 
manager="timlt" 
editor=""/>
<tags 
ms.service="cloud-services" 
ms.workload="tbd" 
ms.tgt_pltfrm="na" 
ms.devlang="na" 
ms.topic="article" 
ms.date="06/22/2016" 
ms.author="adegeo"/>

# Common Cloud Service startup tasks

This article provides some examples of common startup tasks you may want to perform in your cloud service. You can use startup tasks to perform operations before a role starts. Operations that you might want to perform include installing a component, registering COM components, setting registry keys, or starting a long running process. 

See [this article](cloud-services-startup-tasks.md) to understand how startup tasks work, and specifically how to create the entries that define a startup task.

Many of the tasks here use the 

>[AZURE.NOTE] Startup tasks are not applicable to Virtual Machines, only to Cloud Service Web and Worker roles.


## Define environment variables before a role starts

If you need environment variables defined for a specific task, use the [Environment] element inside of the [Task] element.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple">
                <Environment>
                    <Variable name="MyEnvironmentVariable" value="MyVariableValue" />
                </Environment>
            </Task>
        </Startup>
    </WebRole>
</ServiceDefinition>
```

Variables can also use a [valid Azure XPath value](cloud-services-role-config-xpath.md) to reference something about the deployment. Instead of using the `value` attribute, define a [RoleInstanceValue] child element.

```xml
<Variable name="PathToStartupStorage">
    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
</Variable>
```


## Configure IIS startup with AppCmd.exe

The [AppCmd.exe](https://technet.microsoft.com/library/jj635852.aspx) command line tool can be used to manage IIS settings at startup on Azure. *AppCmd.exe* provides convenient, command line access to configuration settings for use in startup tasks on Azure. Using *AppCmd.exe*, Website settings can be added, modified, or removed for applications and sites.

However, there are a few things to watch out for in the use of *AppCmd.exe* as a startup task:

- Startup tasks can be run more than once between reboots. This can happen if the role recycles, for instance.
- Some *AppCmd.exe* actions can generate errors if they are performed more than once. Attempting to add a section to *Web.config* twice could generate an error.
- Startup tasks fail if they return a non-zero exit code or **errorlevel**. This can happen if *AppCmd.exe* generates an error.

For the reasons listed, it is often wise to check the **errorlevel** after calling *AppCmd.exe*, which is easy to do if you wrap the call to *AppCmd.exe* with a *.cmd* file. If you detect a known **errorlevel** response, you can ignore it, otherwise return it. This is demonstrated in the example below.

The errorlevels returned by *AppCmd.exe* are listed in the winerror.h file, and can also be seen on [MSDN](https://msdn.microsoft.com/library/windows/desktop/ms681382.aspx).

### Example

This example adds a compression section and a compression entry for JSON to the *Web.config* file, with error handling and logging.

The relevant sections of the [ServiceDefinition.csdef] file are shown here, which include setting the [executionContext](https://msdn.microsoft.com/library/azure/gg557552.aspx#Task) attribute to `elevated` to give *AppCmd.exe* sufficient permissions to change the settings in the *Web.config* file:

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="Startup.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

The Startup.cmd batch file uses AppCmd.exe to add a compression section and a compression entry for JSON to the *Web.config* file. The expected **errorlevel** of 183 is set to zero using the VERIFY.EXE command line program. Unexpected errorlevels are logged to StartupErrorLog.txt.

    REM   *** Add a compression section to the Web.config file. ***
    %windir%\system32\inetsrv\appcmd set config /section:urlCompression /doDynamicCompression:True /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
    
    REM   ERRORLEVEL 183 occurs when trying to add a section that already exists. This error is expected if this
    REM   batch file were executed twice. This can occur and must be accounted for in a Azure startup
    REM   task. To handle this situation, set the ERRORLEVEL to zero by using the Verify command. The Verify
    REM   command will safely set the ERRORLEVEL to zero.
    IF %ERRORLEVEL% EQU 183 DO VERIFY > NUL
    
    REM   If the ERRORLEVEL is not zero at this point, some other error occurred.
    IF %ERRORLEVEL% NEQ 0 (
        ECHO Error adding a compression section to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
        GOTO ErrorExit
    )
    
    REM   *** Add compression for json. ***
    %windir%\system32\inetsrv\appcmd set config  -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json; charset=utf-8',enabled='True']" /commit:apphost >> "%TEMP%\StartupLog.txt" 2>&1
    IF %ERRORLEVEL% EQU 183 VERIFY > NUL
    IF %ERRORLEVEL% NEQ 0 (
        ECHO Error adding the JSON compression type to the Web.config file. >> "%TEMP%\StartupLog.txt" 2>&1
        GOTO ErrorExit
    )
    
    REM   *** Exit batch file. ***
    EXIT /b 0
    
    REM   *** Log error and exit ***
    :ErrorExit
    REM   Report the date, time, and ERRORLEVEL of the error.
    DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
    TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO An error occurred during startup. ERRORLEVEL = %ERRORLEVEL% >> "%TEMP%\StartupLog.txt" 2>&1
    EXIT %ERRORLEVEL%


## Add firewall rules

In Azure, there are effectively two firewalls. The first firewall controls connections between the virtual machine and the outside world. This is controlled by the [EndPoints] element in the [ServiceDefinition.csdef] file.

The second firewall controls connections between the virtual machine and the processes within that virtual machine. This is controlled by the `netsh advfirewall firewall` command line tool, and is the focus of this article.

Azure creates firewall rules for the processes started within your roles. For example, when you start a service or program, Azure automatically creates the necessary firewall rules to allow that service to communicate with the Internet. However, if you create a service that is started by a process outside your role (for example, a COM+ service, or a program that starts by using the Windows Scheduler), you will need to manually create a firewall rule to allow access to that service. These firewall rules can be created by using a startup task.

A startup task that creates a firewall rule must have an [executionContext][Task] of **elevated**. Add the following startup task to the [ServiceDefinition.csdef] file.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="AddFirewallRules.cmd" executionContext="elevated" taskType="simple" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

To add the firewall rule, you must use the appropriate `netsh advfirewall firewall` commands in your startup batch file. In this example, the startup task requires security and encryption for TCP port 80.

    REM   Add a firewall rule in a startup task.
    
    REM   Add an inbound rule requiring security and encryption for TCP port 80 traffic.
    netsh advfirewall firewall add rule name="Require Encryption for Inbound TCP/80" protocol=TCP dir=in localport=80 security=authdynenc action=allow >> "%TEMP%\StartupLog.txt" 2>&1
    
    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%


## Block a specific IP address

You can restrict an Azure web role access to a set of specified IP addresses by modifying your IIS **web.config** file and creating a command file which unlocks the **ipSecurity** section of the **ApplicationHost.config** file.

First, create a command file that runs when your role starts which unlocks the **ipSecurity** section of the **ApplicationHost.config** file. Create a new folder at the root level of your web role called **startup** and, within this folder, create a batch file called **startup.cmd**. Set the properties of this file to **Copy Always** to ensure that it will be deployed.

Add the following startup task to the [ServiceDefinition.csdef] file.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        <Startup>
            <Task commandLine="startup.cmd" executionContext="elevated" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

Add this command to the **startup.cmd** file:

    %windir%\system32\inetsrv\AppCmd.exe unlock config -section:system.webServer/security/ipSecurity

This causes the **startup.cmd** batch file to be run every time the web role is initialized, ensuring that the required **ipSecurity** section is unlocked.

Finally, modify the [system.webServer section](http://www.iis.net/configreference/system.webserver/security/ipsecurity#005) your web role???s **web.config** file to add a list of IP addresses that are granted access, as shown in the following example:

This sample config **allows** all IPs to access the server except the two defined

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are granted access-->
    <ipSecurity>
        <!--The following IP addresses are denied access-->
        <add allowed="false" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="false" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

This sample config **denies** all IPs from accessing the server except for the two defined.

```xml
<system.webServer>
    <security>
    <!--Unlisted IP addresses are denied access-->
    <ipSecurity allowUnlisted="false">
        <!--The following IP addresses are granted access-->
        <add allowed="true" ipAddress="192.168.100.1" subnetMask="255.255.0.0" />
        <add allowed="true" ipAddress="192.168.100.2" subnetMask="255.255.0.0" />
    </ipSecurity>
    </security>
</system.webServer>
```

## Create a PowerShell startup task

Windows PowerShell scripts cannot be called directly from the [ServiceDefinition.csdef] file, but they can be invoked from within a startup batch file.

PowerShell, by default, will not run an unsigned script. Unless you sign your scripts, you need to configure Windows PowerShell to run unsigned scripts. To run unsigned scripts, **ExecutionPolicy** must be set to **Unrestricted**. The **ExecutionPolicy** setting that you use is based on the version of Windows PowerShell.

    REM   Run an unsigned PowerShell script and log the output
    PowerShell -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
        
    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%


If you're using a Guest OS that is runs PowerShell 2.0 or 1.0 you can force version 2 to run, and if unavailable, use version 1.

    REM   Attempt to set the execution policy by using PowerShell version 2.0 syntax.
    PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

    REM   If PowerShell version 2.0 isn't available. Set the execution policy by using the PowerShell
    IF %ERRORLEVEL% EQU -393216 (

       PowerShell -Command "Set-ExecutionPolicy Unrestricted" >> "%TEMP%\StartupLog.txt" 2>&1
       PowerShell .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1
    )

    REM   If an error occurred, return the errorlevel.
    EXIT /B %errorlevel%

## Create files in local storage from a startup task

You can use a local storage resource to store files created by your startup task which will be accessed later by your application.

To create the local storage resource, add a [LocalResources] section to the [ServiceDefinition.csdef] file and then add the [LocalStorage] child element. Give the local storage resource a unique name and an appropriate size for your startup task.

To use a local storage resource in your startup task, you need to create an environment variable to reference the local storage resource location. Then the Startup task and the application will be able to read and write files to the local storage resource.

The relevant sections of the **ServiceDefinition.csdef** file are shown here:

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">
        ...
        
        <LocalResources>
          <LocalStorage name="StartupLocalStorage" sizeInMB="5"/>
        </LocalResources>
        
        ...
        
        <Runtime>
            <Environment>
                <Variable name="PathToStartupStorage">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='StartupLocalStorage']/@path" />
                </Variable>
            </Environment>
        </Runtime>
        
        ...
        
        <Startup>
          <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" />
        </Startup>
    </WebRole>
</ServiceDefinition>
```

As an example, this **Startup.cmd** batch file uses the **PathToStartupStorage** environment variable to create the file **MyTest.txt** on the local storage location.

    REM   Create a simple text file.

    ECHO This text will go into the MyTest.txt file which will be in the    >  "%PathToStartupStorage%\MyTest.txt"
    ECHO path pointed to by the PathToStartupStorage environment variable.  >> "%PathToStartupStorage%\MyTest.txt"
    ECHO The contents of the PathToStartupStorage environment variable is   >> "%PathToStartupStorage%\MyTest.txt"
    ECHO "%PathToStartupStorage%".                                          >> "%PathToStartupStorage%\MyTest.txt"

    REM   Exit the batch file with ERRORLEVEL 0.

    EXIT /b 0

You can access local storage from the Azure SDK by using the [GetLocalResource](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.getlocalresource.aspx) method. Standard file read and write operations will then work to read and write the contents of the local storage resource. 

```csharp
string localStoragePath = Microsoft.WindowsAzure.ServiceRuntime.RoleEnvironment.GetLocalResource("StartupLocalStorage").RootPath;

string fileContent = System.IO.File.ReadAllText(System.IO.Path.Combine(localStoragePath, "MyTest.txt"));
```


## Differentiate between running in the Emulator and the Cloud

You can have your startup task perform different steps when it is operating in the cloud compared to when it is in the compute emulator. For example, you may want to use a fresh copy of your SQL data only when running in the emulator. Or you may want to do some sort of performance optimizations for the cloud that you don't need to do when running in the emulator.

This ability to perform different actions on the compute emulator and the cloud can be accomplished by creating an environment variable in the [ServiceDefinition.csdef] file, then testing the environment variable in your startup task.

To create the environment variable, add the [Variable]/[RoleInstanceValue] element and create an XPath value of `/RoleEnvironment/Deployment/@emulated`. The value of the **%ComputeEmulatorRunning%** environment variable will be `"true"` when running on the compute emulator, and `"false"` when running on the cloud.


```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
    <WebRole name="WebRole1">

        ...
        
        <Runtime>
            <Environment>
                <Variable name="ComputeEmulatorRunning">
                    <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
                </Variable>
            </Environment>
        </Runtime>

    </WebRole>
</ServiceDefinition>
```

Any task run can now use the **%ComputeEmulatorRunning%** environment variable to perform different actions based on whether or not the role is running in the cloud or the emulator. Here is a .cmd shell script that checks for that environment variable.

    REM   Check if this task is running on the compute emulator.

    IF "%ComputeEmulatorRunning%" == "true" (
        REM   This task is running on the compute emulator. Perform tasks that must be run only in the compute emulator.
        
    ) ELSE (
        REM   This task is running on the cloud. Perform tasks that must be run only in the cloud.
        
    )


## Detect that your task has already run

The role may recycle without a reboot causing your startup tasks to run again. There is flag to indicate a task has already run on the hosting VM. You may have some tasks where it doesn't matter that they run multiple times. However you may run into a situation where you need to prevent a task from running more than once.

The simplest way to detect that a task has already run is to create a file in the **%TEMP%** folder when the task is successful and look for it at the start of the task. Here is a sample cmd shell script that does that for you.

    REM   If Task1_Success.txt exists, then Application 1 is already installed.
    IF EXIST "%RoleRoot%\Task1_Success.txt" (
      ECHO Application 1 is already installed. Exiting. >> "%TEMP%\StartupLog.txt" 2>&1
      GOTO Finish
    )

    REM   Run your real exe task
    ECHO Running XYZ >> "%TEMP%\StartupLog.txt" 2>&1
    "%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

    IF %ERRORLEVEL% EQU 0 (
      REM   The application installed without error. Create a file to indicate that the task
      REM   does not need to be run again.

      ECHO This line will create a file to indicate that Application 1 installed correctly. > "%RoleRoot%\Task1_Success.txt"
      
    ) ELSE (
      REM   An error occurred. Log the error and exit with the error code.

      DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
      TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
      ECHO  An error occurred running task 1. Errorlevel = %ERRORLEVEL%. >> "%TEMP%\StartupLog.txt" 2>&1

      EXIT %ERRORLEVEL%
    )

    :Finish

    REM   Exit normally.
    EXIT /B 0

## Task best practices
Here are some best practices you should follow when configuring task for your web or worker role.

### Always log startup activities

Visual Studio does not provide a debugger to step through batch files, so it's good to get as much data on the operation of batch files as possible. Logging the output of batch files, both **stdout** and **stderr**, can give you important information when trying to debug and fix batch files. To log both **stdout** and **stderr** to the StartupLog.txt file in the directory pointed to by the **%TEMP%** environment variable, add the text `>>  "%TEMP%\\StartupLog.txt" 2>&1` to the end of specific lines you want to log. For example, to execute setup.exe in the **%PathToApp1Install%** directory:

    "%PathToApp1Install%\setup.exe" >> "%TEMP%\StartupLog.txt" 2>&1

If you want to log all output from the startup task without adding `>> "%TEMP%\StartupLog.txt" 2>&1`to the end of each line, two startup batch files are needed. The first batch file will call the second batch file with redirection to log all of the activities of the second batch file. This is necessary for proper redirection to occur.

The following shows how to redirect all output from a startup batch file. In this example, the ServerDefinition.csdef file creates a startup task that calls Startup1.cmd. Startup1.cmd calls Startup2.cmd, redirecting all output to %TEMP%\\StartupLog.txt.

ServiceDefinition.cmd:

```xml
<Startup>
    <Task commandLine="Startup1.cmd" executionContext="limited" taskType="simple" />
</Startup>
```

Startup1.cmd:

    REM   Startup1.cmd calls the main startup batch file, Startup2.cmd, redirecting all output
    REM   to the StartupLog.txt log file.

    REM   Log the startup date and time.
    ECHO Startup1.cmd: >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO Current date and time: >> "%TEMP%\StartupLog.txt" 2>&1
    DATE /T >> "%TEMP%\StartupLog.txt" 2>&1
    TIME /T >> "%TEMP%\StartupLog.txt" 2>&1
    ECHO Starting up Startup2.cmd. >> "%TEMP%\StartupLog.txt" 2>&1

    REM   Call the Startup2.cmd batch file, redirecting all output to the StartupLog.txt log file.
    START /B /WAIT Startup2.cmd >> "%TEMP%\StartupLog.txt" 2>&1

    REM   Log the completion of Startup1.cmd.
    ECHO Returned to Startup1.cmd. >> "%TEMP%\StartupLog.txt" 2>&1

    IF ERRORLEVEL EQU 0 (
       REM   No errors occurred. Exit Startup1.cmd normally.
       EXIT /B 0
    ) ELSE (
       REM   Log the error.
       ECHO An error occurred. The ERRORLEVEL = %ERRORLEVEL%.  >> "%TEMP%\StartupLog.txt" 2>&1
       EXIT %ERRORLEVEL%
    )

Startup2.cmd:

    REM   This is the batch file where the startup steps should be performed. Because of the
    REM   way Startup2.cmd was called, all commands and their outputs will be stored in the
    REM   StartupLog.txt file in the directory pointed to by the TEMP environment variable.

    REM   If an error occurs, the following command will pass the ERRORLEVEL back to the
    REM   calling batch file.
    EXIT /B %ERRORLEVEL%

### Set executionContext appropriately for startup tasks

Set privileges appropriately for the startup task. Sometimes startup tasks must run with elevated privileges even though the role runs with normal privileges.

The [executionContext][Task] attribute sets the privilege level of the startup task. Using `executionContext="limited"` means the startup task will have the same privilege level as the role. Using `executionContext="elevated"` means the startup task will have administrator privileges, which allows the startup task to perform administrator tasks without giving administrator privileges to your role.

An example of a startup task that requires elevated privileges is a startup task that uses **AppCmd.exe** to configure IIS. **AppCmd.exe** requires `executionContext="elevated"`.

### Use the appropriate taskType

The [taskType][Task] attribute determines the way the startup task will be executed. There are three values: **simple**, **background**, and **foreground**. The background and foreground tasks are started asynchronously, and then the simple tasks are executed synchronously one at a time.

With **simple** startup tasks, you can set the order in which the tasks will occur by the order in which the tasks are listed in the ServiceDefinition.csdef file. If a **simple** task ends with a non-zero exit code, then the startup procedure will stop and the role will not start.

The difference between **background** startup tasks and **foreground** startup tasks is that **foreground** tasks will keep the role running until the **foreground** task ends. This also means that if the **foreground** task hangs or crashes, the role will not recycle until the **foreground** task is forced closed. For this reason, **background** tasks are recommended for asynchronous startup tasks unless you need that feature of the **foreground** task.

### End batch files with EXIT /B 0

The role will only start if the **errorlevel** from each of your simple startup task is zero. Not all programs set the **errorlevel** (exit code) correctly, so the batch file should end with an `EXIT /B 0` if everything ran correctly.

A missing `EXIT /B 0` at the end of a startup batch file is a common cause of roles that do not start.

### Expect startup tasks to run more than once

Not all role recycles include a reboot, but all role recycles include running all startup tasks. This means that startup tasks must be able to run multiple times between reboots without any problems. This is discussed [above](#detect-that-your-task-has-already-run).

### Use local storage to store files that must be accessed in the role

If you want to copy or create a file during your startup task that is then accessible to your role, then that file must be placed in local storage. See the [section](#create-files-in-local-storage-from-a-startup-task) above.

## Next steps

Review the cloud [service model and package](cloud-services-model-and-package.md)

Learn more about how [Tasks](cloud-services-startup-tasks.md) work.

[Create and deploy](cloud-services-how-to-create-deploy-portal.md) your cloud service package.


[ServiceDefinition.csdef]: cloud-services-model-and-package.md#csdef
[Task]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Task
[Runtime]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Runtime
[Startup]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Startup
[Runtime]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Runtime
[Environment]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Environment
[Variable]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Variable
[RoleInstanceValue]: https://msdn.microsoft.com/library/azure/gg557552.aspx#RoleInstanceValue
[RoleEnvironment]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.aspx
[Endpoints]: https://msdn.microsoft.com/library/azure/gg557552.aspx#Endpoints
[LocalStorage]: https://msdn.microsoft.com/library/azure/gg557552.aspx#LocalStorage
[LocalResources]: https://msdn.microsoft.com/library/azure/gg557552.aspx#LocalResources
[RoleInstanceValue]: https://msdn.microsoft.com/library/azure/gg557552.aspx#RoleInstanceValue
