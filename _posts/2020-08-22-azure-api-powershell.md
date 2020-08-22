---
layout: post
title: Azure REST API and PowerShell
---
There are many ways to manage resources on Azure and there are also many tools one can use. For example you can make use of the [Azure Portal](https://portal.azure.com), PowerShell with the [Az extensions](https://www.powershellgallery.com/packages/Az/4.5.0), [Azure CLI](https://docs.microsoft.com/nl-nl/cli/azure/install-azure-cli?view=azure-cli-latest) or make use of the [Azure REST API](https://docs.microsoft.com/en-us/rest/api/azure/) to manage resources onm Azure.  
On this page I will describe how you can use PowerShell to make a call to the REST API of Azure. To make the API call you will need the URL for the API call, the method (POST, GET, etc.) you want to use, the body of the message and a header which is used for authentication towards the API.

### Authentication

To authenticate to the REST API, we need the id of a user or service principal ($clientid), the corresponding secret ($secretid) and the id of the tenant ($tenantid). The service principal or user should also have assigned the permissions needed to perform the tasks.  
When we have the $clientid, $secretid and $tenantid, we can authenticate against the REST API.  

#### Construct the URI
First we need to construct the URI to which we are going to authenticate.  
This URI is constucted the follow way:

$tokenUri = "https://login.microsoftonline.com/$tenantid/oauth2/token"  

#### Construct the body
Now we can construct the body. The body will be sent to the URI to which we will authenticate.  
The body is constructed the follow way:

$tokenBody = @{  
    client_id     = $clientId  
    resource         = "https://management.azure.com/"  
    client_secret = $clientSecret  
    grant_type    = "client_credentials"  
}  

#### Get OAuth 2.0 Token
It is time to get the OAuth token.  
To perform actions on Azure by using the REST API, the token will be send with the request to the REST API to perform actions on Azure.  
First we sent a request to get a token:

$tokenRequest = Invoke-WebRequest -Method Post -Uri $tokenUri -ContentType "application/x-www-form-urlencoded" -Body $tokenBody -UseBasicParsing  

After that, we can extract the token:  
$token = ($tokenRequest.Content | ConvertFrom-Json).access_token  

### Request all resource groups

Now that we have the token, we can request information from Azure through the Azure REST API. The most difficult parts are how to get the URI to use and how to construct the body. A good place to start is the official documentation regarding the [Azure REST API](https://docs.microsoft.com/en-us/rest/api/azure/) and look for the Azure resource you want to perform an action on.
For instance the information to get a list of all resource groups, is locatied [here](https://docs.microsoft.com/en-us/rest/api/resources/resourcegroups/list).  

Based on the documentation, the URI we should use to retrieve a list of all the resource groups that are available in the subscription, we should use the follow URI:  
$uri = "https://management.azure.com/subscriptions/$subscriptionid/resourcegroups?api-version=2020-06-01"  
Where $subscriptionid is the id of the subscription off which we want to retrieve the resource groups.

Now that we have both the URI and the OAuth token, we can perform a REST API call to retrieve all the resource groups of the subscription:  
$resourcegroups = Invoke-RestMethod -Method GET -Uri $uri -Headers @{Authorization = "Bearer $token" }

### Create a resource group

Besides retreiving a list of resource groups, we can also create a new resource group through the Azure REST API.
Again, we need the [URI](https://docs.microsoft.com/en-us/rest/api/resources/resourcegroups/createorupdate):
$uri = "https://management.azure.com/subscriptions/$subscriptionId/resourcegroups/${resourcegroupname}?api-version=2020-06-01"

We have to assign a value to the variable $resourcegroupname:
$resourcegroupname = "myfirstresourcegroup2208"

We have to construct the header for the request:
$Headers=@{
  'authorization'="Bearer $token"
  'host'="management.azure.com"
}

And we need to construct the body for the REST API request. The body contains the properties for the resource group, for example:
$body='{
    "location": "westeurope",
     "tags": {
        "owner": "Me",
        "location": "My City"
    }
 }'

At last we can put all teh pieces together and perform the request to the Azure REST API:
$output = Invoke-RestMethod  -Uri $uri -Headers $Headers -ContentType "application/json" -Method PUT -Body $body

The resource group may be created noe, but we are not sure. The first thibg we can verify is if the REST API call is succesfull or not. We can verify this by viewing at the value of $output.properties
When the output is "succeeded", that  means that de REST API call was succesfull. It does not mean that the resource group is created.
To verify if the resource group is created we can do the REST API call, like before:
$uri = "https://management.azure.com/subscriptions/$subscriptionid/resourcegroups?api-version=2020-06-01"
(Invoke-RestMethod -Method GET -Uri $uri -Headers @{Authorization = "Bearer $token" }).value | Where-Object {$_.name -eq $resourcegroupname}
We should now get output containing the name and additional properties of the resource group we did just create.
