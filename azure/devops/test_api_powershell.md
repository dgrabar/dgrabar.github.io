---
layout: page
title: Test endpoint APIs in Azure release pipeline task using PowerShell task
description: desc
---


Sometimes you need to test some API in order to know whether to procceed with further tasks in your pipeline or not.
Here's an example of how to do it by using Powershell:



## Using PowerShell task in Azure release pipeline to test an existing API

With Azure DevOps you can create PowerShell task to be run as part of your release pipeline as shown:

![Azure PowerShell task](images/test_api_powershell.png)


## PowerShell script

Run the following script as part of PowerShell script job:

    $res = Invoke-RestMethod -Uri http://localhost:8059/webservice/home/someapi
    
    If ($response.Content -eq "null")
    {
        Write-Host "Server returned null"
    }
    else
    {
        if($res.IsSuccessful)
        {
            write "OK, world is fine"
        }
        else
        {
            Write-Error $res.Error
        }
    }

Ok, let's decompose this.

First, we create temp variable where backup files will be outputted:

    $folder=powershell get-date -format "{dd-MMM-yyyy__HH_mm}"

Here, we are using Powershell `get-date` function and we will format date to format acceptable for folder name on Windows drive. 


Then we are creating IIS config backup using native `appcmd` app with dynamically named subfolder inside `..\inetsrv` folder. 

    Invoke-Expression "& $env:windir\system32\inetsrv\appcmd.exe add backup ""$folder-IIS"""

Note the way we are getting local `Windows` path by using PowerShell supported PATH: `$env:windir`.

Afterwards, we copy backup folder to the destination backup folder using `xcopy` command:

    xcopy "C:\Windows\System32\inetsrv\backup\$folder-IIS\*" "d:\Deployment\$folder\IIS-config" /i /s /y

The crucial step is the usage of existing `msdeploy` app which is used for various deplyoments scenarios, in this case backing up the whole site:

    Invoke-Expression "& 'C:\Program Files (x86)\IIS\Microsoft Web Deploy V3\msdeploy.exe' --% -verb:sync -source:iisapp=""Yourwebsite/YourWebservice"" -dest:package=D:\Deployment\$folder\site.zip"

If you don't have `msdeploy` app installed, you can download it from [here](https://www.iis.net/downloads/microsoft/web-deploy).

## Checking the output of PowerShell task result

Once this job finishes, following should be seen inside destination backup folder:

![Output of PowerShell task](images/backup_task_output.png)
