---
layout      : post
title       : "Mapping Contracts from Logic Apps in Azure API Management using Azure Pipelines"
description : "Today, you will see how to connect your Logic App in APIM using Azure Pipelines"
tags        : [azure-devops, azure-pipelines,logic-apps, apim, azure]
---

# Scenario
Recently, one of my costumers decided to use Azure Logic App in applications to be responsable for some operations, for example, delete data and others.
In the first time, i create all resources using Azure Portal, including the mapping from Logic Apps and APIM. BUT... In my scenarion, we are usign the Azure Pipelines to deploy All resources in Azure, Application and IaC, and now, i have a problem.

![image 01 - Logic App Mapped in APIM](..\..\..\assets\img\apim-la-01.png)

# Problem
I need to run my deployments *only using Azure Pipelines*, that is, i need to run only using scripts or using IaC templates to deploy this.
I looking in documentation, but, i can't find how to do this using ARM Templates, Terraform or CLI, and i can't find the explanation how this integration works.
For example, when you integrate the both using Azure Portal, behind the scenes, the Azure create an internal mapping in the *inbound process*, with internal values without clear explanations, like this:

<script src="https://gist.github.com/julioarruda/9ce0ac2ed8830b7c869cc1e5b23a7d3a.js"></script>

If you see, in the file, whe have two values:
- LogicApp_logicapp-p_86d45f30c6758d1a59fc45e656e85063
- echo-api_create-resource_5f4e32cad47afaa03b0bf061

The first value, is an Backend Name, and the Second, is a Named Value, and this, is a special value, this is a *Logic App Access Key*, we will see how to get this in the future.

## APIM Backend
The first step to mapping Logic Apps with APIM, is a creation of the Backend. Create this backend is simple, in my scenario, i'm using the [Azure Rest API](https://docs.microsoft.com/en-us/rest/api/apimanagement/2019-12-01/backend/createorupdate), and added this in my Azure Pipelines.

I shared with yours in my gist, the script i used to create my Backend, in this script, you need to inform:

- Logic App Name
- APIM Name
- APIM Resource Group
- Subscription Name

This script make a request to Logic App, to request the trigger information. This content is important to create a Backend, and i added this in the body to APIM Backend creation.

After this, my script run the Backend creation in APIM, and output to Azure Pipelines, the Backend Name.

<script src="https://gist.github.com/julioarruda/76c49095d9e0372228ece7d4e8b81840.js"></script>

## Logic App Access Key
To continue this process, we need to get a Logic App Access Key. I found one away to get this, using an ARM Template to get this value.

To run this ARM, you need to set one variable in execution time:
- logicAppName

After the ARM run, you be able to get a value called *sasToken*, and this value, is necessary to creation a Named Value, in next step.
<script src="https://gist.github.com/julioarruda/e04bb81b075b8d769f34e69b2d484222.js"></script>

## APIM Named Value
Now, we need to create a Named Value in APIM. In this step, we need to use the [Azure Rest API](https://docs.microsoft.com/en-us/rest/api/apimanagement/2019-12-01/namedvalue/createorupdate) again.

In this script, we using the Access Key that we got in the last step, to add in the body of the API request. This script output an Azure DevOps variable with the Named Value Name.

<script src="https://gist.github.com/julioarruda/55403362a03a11ce0fb62b2a09c2052a.js"></script>

## Mounting the Inbound Process.
Now, we can be able to create a Inbound Process to Add in your API Method, like the first script in this article, changing the variables from Backend Name and Named Value.
<script src="https://gist.github.com/julioarruda/9ce0ac2ed8830b7c869cc1e5b23a7d3a.js"></script>

## Azure Pipelines
Now, we can now how to create all points in Azure using scripts. Now, we can add the scripts in Azure DevOps pipelines. The first step for this, is add an Azure DevOps Extension from Marketplace.

- [API Management Suite](https://marketplace.visualstudio.com/items?itemName=stephane-eyskens.apim)


This extension, is the most complete in Marketplace today. With this extension, we can deploy API's in APIM from Swagger, from Functions, apply Global Policies and apply Inbound Process in each API method. In this example, we need especifically the task to deploy a Inbound Process in API Method.

![image 02 - APIM Extension Marketplace](..\..\..\assets\img\apim-extension-marketplace.png)

We need another extension, called *ARM Outputs*, you need this to get output values from ARM Templates Executions.

- [ARM Outputs](https://marketplace.visualstudio.com/items?itemName=keesschollaart.arm-outputs)
![image 03 - ARM Output Extension Marketplace](..\..\..\assets\img\arm-output-extension-marketplace.png)


## Azure Pipelines - Process
Now, we can to create our Pipeline to deploy this integration on APIM. This script, is the steps that you need to add in your own pipeline, with adjusts, like Subscrpition ID, Resource Group name, and others.

The Steps are:
- 'Create APIM Backend' (Azure CLI)
- 'Getting LogicApp Access Key' (ARM Template)
- 'Output Access Key' (Output ARM Template)
- 'Create APIM Named Value' (Azure CLI)
- 'API Management - Set or update an operation policy ' (Stephane's Task)

<script src="https://gist.github.com/julioarruda/87ba1cc11a4ac75d8369176aa094deae.js"></script>

# Conclusion
It's a big process, but it's a very simple after you understand this. It's solved my scenario and probably will be help you too. If you have a questions, ask me in the comments. :)