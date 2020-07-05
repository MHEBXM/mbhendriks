---
layout: post
title: Azure DevOps REST API and PowerShell
---
### Introduction

There are multiple approaches you can take to set up, configure and manage Azure DevOps. There are also multiple tools you can use. You can use the Azure Cli with the [Azure DevOps extension](https://github.com/Azure/azure-devops-cli-extension), the VSTeam extension from the [PowerShell Gallery](https://www.powershellgallery.com/packages/VSTeam/5.0.0) or make use of [API calls](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-5.1) to set up, configure and manage Azure DevOps.  
On this page I will describe how you can use PowerShell to make a call to the REST API of Azure DevOps. To make the API call you will need the URL for the API call, the method (POST, GET, etc.) you want to use and a header which is used for the authentication towards the API.

### Authentication

To authenticate to the REST API, one can use a personal access token (PAT). A PAT is created from within Azure DevOps.  
Personal access token is located below the user settings:

![user settings]({{ site.baseurl }}/images/api/pat1.png "user settings")

To create a new personal access token, just choose "new token" and the "create a new personal access token" page will appear. Choose a name for the personal access token, select an organization and assign the right level of access to the personal access token. If the shown level of access is not granular enough, choose "show all scopes" to extend the permissions screen.  
It is common to assign the minimal permissions to the PAT.  
In the screen dump below, the option "all accessible organizations" is selected. One should of course choose his/her organization.

![create PAT]({{ site.baseurl }}/images/api/pat2.png "create PAT")

Once the PAT is generated, the secret that belongs to the PAT is shown. One should store the secret in a safe place, because this will be the only time one can copy the secret.

![PAT secret]({{ site.baseurl }}/images/api/pat3.png "PAT secret")

Although the secret of the PAT cannot be retrieved again, it is possible to change the assigned permissions and scope by editing the PAT.  

Once the PAT has been created, we can set up the authentication header which will be used in the API call.  
To construct the authentication header, we need to set two variables first:  

$organization = "The name of your Azure DevOps organization"  
$pat = "The secret of the PAT in plain text"

Instead of setting both variables hard coded, both should be retrieved from a secure place, like a key vault.  
After setting both variables, the header can be constructed:

$header = @{"Authorization" = "Basic "+[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("$($organization):$($pat)"))}

### URL

The URL to use to make the API call can be found on the documentation pages of de [Azure DevOps REST API](https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-5.1). There are multiple API versions available and also some of them are still in preview.  
If you for instance want list all the builds in a certain project, you could use the following [URL](https://docs.microsoft.com/en-us/rest/api/azure/devops/build/builds/list?view=azure-devops-rest-5.1) and the GET method:

$uri = "https://dev.azure.com/$organization/$project/_apis/build/builds?api-version=5.1"  

Where $organization is the name of the Azure DevOps organization and $project is the name of the project within the Azure DevOps organization.

### Putting it all together

Now it's time to put all the pieces together and perform the API request. For this, we use the "Invoke-RestMethod" command of PowerShell. Based on the variables we created in the previous sections, the command should be:

$pipelines = (Invoke-RestMethod -Method GET -Uri $uri -Header $header).value  

The variable $pipelines is an array which contains the history of all the builds.
