---
title: Deploy an ASP.NET Core application to Azure App Service using VSTS
layout: page
sidebar: vsts2
permalink: /labs/vstsextend/dotnetcore/
folder: /labs/vstsextend/dotnetcore/
---

## Overview

This lab shows how to deploy an ASP.NET Core application to Azure App Service with Visual Studio Team Services.

[ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/) is a cross-platform, high-performance, open-source framework for building modern, cloud-based, Internet-connected applications. With ASP.NET Core, you can:

- Build web apps and services, IoT apps, and mobile backends.
- Use your favorite development tools on Windows, macOS, and Linux.
- Deploy to the cloud or on-premises
- Run on .NET Core or .NET Framework.

## Prerequisites for the lab

1. **Microsoft Azure Account**: You will need a valid and active Azure account for the Azure labs. If you do not have one, you can sign up for a [free trial](https://azure.microsoft.com/en-us/free/){:target="_blank"}

    - If you are an active Visual Studio Subscriber, you are entitled for a $50-$150 credit per month. You can refer to this [link](https://azure.microsoft.com/en-us/pricing/member-offers/msdn-benefits-details/){:target="_blank"} to find out more information about this including how to activate and start using your monthly Azure credit.

    - If you are not a Visual Studio Subscriber, you can sign up for the FREE [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/){:target="_blank"} program to create a **Azure free account** (includes 1 year of free services, $200 for 1st month).

1. You will need a **Visual Studio Team Services Account**. If you do not have one, you can sign up for free [here](https://www.visualstudio.com/products/visual-studio-team-services-vs){:target="_blank"}

1. You will need a **Personal Access Token** to set up your project using the **VSTS Demo Generator**. Please see this [article](https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate){:target="_blank"} for instructions to create your token.

    {% include note.html content= "You should treat Personal Access Tokens like passwords. It is recommended that you save them somewhere safe so that you can re-use them for future requests." %}

## Setting up the VSTS Project

1. Use the [VSTS Demo Generator](https://vstsdemogenerator.azurewebsites.net/?name=MyHealthClinic){:target="_blank"} to provision the team project on the VSTS account.

   > **VSTS Demo Generator** helps you create team projects on your VSTS account with sample content that include source code, work items,iterations, service endpoints, build and release definitions based on the template you choose during the configuration.

   ![VSTS Demo Generator](images/vstsdemo.png)

1. Once the team project is provisioned, click on the URL to navigate to the team project.

   ![VSTS Demo Generator](images/vstsdemo2.png)

## Exercise 1: Configure Release

1. Go to **Releases** under **Build & Release** tab, edit the release definition **MyHealthClinicE2E**.

   ![create_release](images/releaseedit3.png)

1. Select **Tasks** tab and click **Dev**.

   ![release_2](images/release4.png)

1. Select the **Azure Resource Group Deployment** task, choose the **Azure subscription**. There are 2 ways of choosing the Azure subscription.

    1. If your subscription is not listed or if you want to use an existing service principal, click the Manage link.

        Click on the **+New Service Connection** button and select the **Azure Resource Manager** option. Provide Connection name, select the Azure Subscription from the list and the click on the Ok button. The Azure credentials will be required to be provided to authorize the connection.

        ![Endpoint](images/endpoint5.png)

    2. If the subscription is already listed, select the Azure subscription from the list and click Authorize.

        ![Endpoint](images/endpoint6.png)


1. Select the desired **Location** for deployment.

   ![release_3](images/location7.png)

1. Select the **Azure App Service Deploy** task and pick **Azure subscription** from the dropdown list. In the **Slot** section, provide the slot name as **Dev**.

   ![release_4](images/dev8.png)

1. Similarly update **Azure subscription** for **QA** and **Production** environments. Go to **Tasks** and select **QA**.

   ![qa](images/qa9.png)

1. Under **Azure App Service Deploy** task, update **Azure subscription** from the dropdown. Under the **Slot** section enter the slot name as **Staging**.

   ![qa_1](images/qa10.png)

1. Go to **Tasks** and select **Production**.

   ![prod](images/prod11.png)

1. Under **Azure App Service Deploy** task, update **Azure subscription** from the dropdown and click **Save** to save the release definition.

   ![prod_1](images/prod12.png)

## Exercise 2: Triggering a Continuous Integration (CI) build

1. Go to the **Code** hub.

   ![code_hub](images/Code13.png)

1. We have a **DOTNETCORE** app code provisioned by the demo generator system. We will deploy this to Azure app service.

1. We have a Continuous Integration (CI) build setup to run upon a code commit. Let's make a simple change to the code to trigger the CI build.

1. Open the file **Index.cshtml** by navigating to the below path and click on **Edit**.

   > **MyHealthClinic/src/MyHealth.Web/Views/Home/Index.cshtml**

   ![code_edit](images/Code14.png)

1. Edit the code. For this example, let's change **line 33** to change the text from **Login** to **Private Area** and click on **Commit** to save and commit the changes.

   ![update_code](images/Code15.png)

1. The code commit will trigger the CI build. Go to the **Build** tab to see the CI build running in progress. 

   ![build_in_progress](images/buildprog18.png)

   While the build is in progress, let's explore the build definition. The tasks that is used in the build definition are listed in the table below.

   |Tasks| Usage|
   |-----|------|
   |![dotnetcore](images/dotnetcore.png) **Restore**| dotnet command-line tool restores all the package dependencies like **ASP.NET Core Identity, ASP.NET Core session** etc. required to build this project|
   |![npm](images/npm.png) **npm Install**| npm task installs the npm packages (javascript dependencies) like **babelify, browserify** etc. required to build this project|
   |![bower](images/bower.png) **Bower Install**|We will use this task to manage components that contain HTML, CSS, JavaScript, fonts and even image files. Example: jquery, angular, webcomponentsjs etc|
   |![gulp](images/gulp.png) **Gulp**| gulp task compiles sass files, uglify and compress js files|
   |![dotnetcore](images/dotnetcore.png) **Build**| We will use dotnet command-line tool to build the project and its dependencies into a set of binaries|
   |![dotnetcore](images/dotnetcore.png) **Test**| dotnet command-line tool wil run unit tests as part of build process to ensure the code quality. This project contains 6 unit tests|
   |![dotnetcore](images/dotnetcore.png) **Publish**| We will use this task to create a package with published content for the web deployment|
   |![vstest](images/vstest.png) **Publish Test Results**| We will use this task to publish the unit test result|
   |![copyfiles](images/copyfiles.png) **Copy Files**| We will copy the zipped file and the ARM template to a staging directory|
   |![publishartifacts](images/publishartifacts.png) **Publish Build Artifacts**| And finally, we will publish the files in the staging directory which were copied in the previous step|

1. Select the build definition `MyHealthClinicE2E` and click on the ellipsis to view the build in progress.

   ![build_number](images/Build16.png)

   ![build_in_progress_2](images/buildprg17.png)

1. Once the build is complete click on the build number, to see the summary which shows **Test Results** as shown.

   ![build_summary](images/buldsuccess19.png)

## Exercise 3: Continuous Delivery

We are using **Infrastructure as a Code** in the release pipeline. We have a release configured to deploy the application which is associated to the build and triggered when the build is successful.

1. Go to **Releases** tab under **Build and Release** hub.

1. Select the **MyHealthClinicE2E** definition, you will see the release in-progress.

   ![release](images/release20.png)

1. While the release is in-progress, let's explore the tasks used. Click **edit** to see the release pipeline. We have three environments **Dev, QA** and **Production**.

   ![pipeline](images/release21.png)

   >Go to the Dev environment, you will see 2 tasks are used. Let us explore the tasks.

   ![release_tasks](images/release22.png)

   - **Azure Resource Group Deployment**: The project used in this lab contains frontend (Azure App Service) and backend (Azure SQL DB) services. We will provision these services as PAAS on Azure using ARM templates. This task will create the above services in a resource group **dotnetcore.**

   - **Azure App Service Deploy**: The task is used to deploy a Web project to the Azure App Service created above.

1. Click on **View releases**.

   ![view_releases](images/release23.png)

1. Double click on each environment to see the release summary and logs.

   ![release_summary1](images/releasesucc24.png)

   ![release_summary](images/release25.png)

1. Login to [Azure Portal](https://portal.azure.com) and search a **Resource Group** with the name **dotnetcore**.

   ![azure_resources](images/azure_resources.png)

1. Navigate to either Dev or Staging web app in the resource group and you will see the application is deployed successfully with the changes.

   ![mhc_web_app](images/mhc_web_app.png)

## Summary

**Visual Studio Team Services** simplifies creation of continuous integration and continuous delivery pipelines for your application to be deployed to Azure.
