---
title: Deployment of ASP.NET application to the Azure App Service using VSTS
layout: page
sidebar: vsts2
permalink: /labs/vstsextend/aspdotnet/
folder: /labs/vstsextend/aspdotnet/
---

## Overview

This lab details the deployment of an [ASP.NET](https://www.asp.net/){:target="_blank"} application to the Azure App Service using Visual Studio Team Services (VSTS).
ASP.NET is an open source web framework for building modern web applications and services. ASP.NET creates websites based on HTML5, CSS, and JavaScript that are simple, fast, and can scale to millions of users.

### Prerequisites for the lab

1. **Microsoft Azure Account**: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-us/free/){:target="_blank"}

    * If you are an active Visual Studio Subscriber, you are entitled for a $50-$150 credit per month. You can refer to this [link](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/){:target="_blank"} to find out more information about this including how to activate and start using your monthly Azure credit.

    * If you are not a Visual Studio Subscriber, you can sign up for the FREE [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/){:target="_blank"} program to create a **Azure free account** (includes 1 year of free services, $200 for 1st month).

1. You will need a **Visual Studio Team Services Account**. If you do not have one, you can sign up for free [here](https://www.visualstudio.com/products/visual-studio-team-services-vs){:target="_blank"}

## Setting up the VSTS Project

1. Use the [VSTS Demo Generator](https://vstsdemogenerator.azurewebsites.net/?name=PartsUnlimited){:target="_blank"} to provision the team project on the VSTS account.

   > **VSTS Demo Generator** helps you create team projects on your VSTS account with sample content that include source code, work items,iterations, service endpoints, build and release definitions based on the template you choose during the configuration.

   ![VSTS Demo Generator](images/vstsdemo.png)

1. Once the team project is provisioned, click on the URL to navigate to the team project.

   ![VSTS Demo Generator](images/vstsdemogenproject5.png)

## Exercise 1: Configure Release

1. Click on the **Build & Release** section and then click on the **Releases**. Select the release definition **PartsUnlimitedE2E** and click on the **Edit** button.

   ![Edit Release Definition](images/releaseedit7.png)

1. Select the **Tasks** tab and click on the **Dev** option.

   ![Release Tasks](images/dev8.png)

1. Select the **Azure Resource Group Deployment** task, choose the **Azure subscription**. There are 2 ways of choosing the Azure subscription.
   
    * If your subscription is not listed or if you want to use an existing service principal, click the `Manage` link. 

        1. Click on the `+New Service Connection` button and select the **Azure Resource Manager** option. Provide Connection name, select the Azure Subscription from the list and the click on the Ok button. The Azure credentials will be required to be provided to authorize the connection.

        ![Endpoint](images/endpoint_creation.png)

    * If the subscription is already listed, select the Azure subscription from the list and click `Authorize`.

        ![Authorize](images/authorize.png)

1. Select the desired **Location** for deployment.

    ![Deployment Location](images/devedit9.png)

1. Select the **Azure App Service Deploy** task and select the **Azure subscription** from the dropdown list. In the **Slot** section, provide the slot name as **Dev**.

   ![Deployment Slot](images/dev10.png)

1. Similarly configure the  **Azure subscription** for the **QA** and **Production** environments. Select the **Tasks** tab and click on the **QA** option.

   ![Tasks](images/qaselect10.png)

1. Select the **Azure App Service Deploy** task and pick the **Azure subscription** from the dropdown. Under the **Slot** section enter the slot name as **Staging**.

   ![Staging](images/qaedit11.png)

1. Navigate to the **Tasks** tab and select the **Production** option.

   ![Production](images/prodselect12.png)

1. Select the **Azure App Service Deploy** task, pick the **Azure subscription** from the dropdown and click on the **Save** button to save the release definition.

   ![Azure Configuration](images/prod13.png)

## Exercise 2: Initiate Continuous Integration (CI) and Continuous Deployment (CD)

To automatically initiate the CI-CD, the source code needs to be modified and committed to the source code repository.

1. Navigate to the **Code** hub on the VSTS portal.

   ![Code Hub](images/code14.png)

1. The repository contains an **ASP.NET** application source code provisioned by the VSTS Demo Generator. This application will be deployed to the Azure App Service.

   > The team project already has a Continuous Integration (CI) build configured that gets automatically initiated when the source code modifications are committed to the repository.

1. To edit the source code, open the file **Index.cshtml** by navigating to the below path in the master branch:

   `PartsUnlimited/PartsUnlimited-aspnet45/src/PartsUnlimitedWebsite/Views/Home/Index.cshtml`

   ![Source code path](images/code15.png)

1. Make some small changes to the code. For this example, change the discount percentage of `50%` to `70%` on `line 28` and then click on the **Commit** button to save and commit the changes.

   ![Code Edit](images/code16.png)

1. The code commit will trigger the CI build. Navigate to the **Build and Release** tab to view the progress of the  the CI build initiated automatically due to the code changes.

   ![CI Status](images/buildprog16.png)

   The following tasks are used in the build definition:

   | Tasks used in Build | Usage / Purpose |
   |-------|-------|
   |![Nuget](images/nuget.png) **Nuget Installer**| Nuget installer restores and updates all the package dependencies required to compile the project|
   |![VS Build](images/visual-studio-build.png) **Visual Studio Build**| The Visual Studio Build task is used to invoke the MS build to compile and package the output as a zip file.|
   |![VS Test](images/vstest.png) **Visual Studio Test**| As a part of the build process, all the unit tests included in the project will be executed using the VisualStudio Test task to ensure the code quality. This project contains 16 unit tests|
   |![Copy Files](images/copyfiles.png) **Copy Files**| This task is used to copy the zipped binaries and the ARM template to a staging directory|
   |![Publish Build Artifacts](images/buildartifacts.png) **Publish Build Artifacts**| This task is used to deploy the binaries in the staging directory that were copied in the previous step|

1. To view the live progress of the build, click on the build number to open the build live console.

   ![Build number](images/buildprog17.png)

   ![Buidl Progress](images/buildprog18.png)

1. Once the build is completed, if not clicked earlier, click on the build number to view the build summary including **Test Results**, **Code Coverage** etc.

   ![Build Summary](images/buildsucce19.png)

## Exercise 3: Continuous Delivery

The release pipeline is configured as **Infrastructure as a Code** to deploy the application which is associated to the build and is automatically triggered when the build is successfully completed.

1. To view the release progress, click on the **Build and Release** hub and select the **Release** option.

1. Click on the **PartsUnlimitedE2E** definition, to view the release progress.

   ![Release Progress](images/releaseprog20.png)

1. Click on the **Edit** button to view the release pipeline. There are three configured deployment environments namely the **Dev**, **QA** and **Production**.

   ![Edit Release](images/releaseprog21.png)

1. Select the **Dev** environment to view the tasks configured.

   ![Release Tasks](images/release24.png)

   | Tasks used in Release | Usage / Purpose |
   |-------|-------|
   | **Azure Resource Group Deployment**|The project used in this lab contains the front-end (Azure App Service) and back-end (Azure SQL DB) services. These will be provisioned as [PAAS on Azure](https://azure.microsoft.com/en-in/overview/what-is-paas/){:target="_blank"} using [ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-create-first-template){:target="_blank"} templates. The task will create the above services in a resource group named **ASPDOTNET**|
   | **Azure App Service Deploy**| The task is used to deploy a Web project to the Azure App Service created in the previous step|

1. To view the outcome of the release, click on the **View releases** option.

   ![View Releases](images/releaseprog22.png)

1. To view the release summary for the environment, double click on the environment blocks displayed in the report.

   ![Release Summary](images/releasesuccess23.png)

   ![Release Summary](images/release_summary.png)

1. Login to the [Azure Portal](https://portal.azure.com) and search for a **Resource Group** that you had created for the project. (In this example, it's **ASPDOTNET**)

   ![Azure Resources](images/azure_resources.png)

1. Browse to either the deployed Dev or Staging web application from the resource group to validate that the changes have been deployed successfully.

   ![Overview](images/partsunlimited_overview.png)

## Summary

The **VSTS** simplifies the creation of the continuous integration and continuous delivery pipelines for the application to be deployed to the Azure platform.