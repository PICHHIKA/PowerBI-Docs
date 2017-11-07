---
title: How to migrate Power BI Embedded workspace collection content to Power BI
description: Learn how to migrate from Power BI Embedded to the Power BI service and leverage advances for embedding in apps.
services: powerbi
documentationcenter: ''
author: guyinacube
manager: erikre
backup: ''
editor: ''
tags: ''
qualityfocus: no
qualitydate: ''

ms.service: powerbi
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: powerbi
ms.date: 07/21/2017
ms.author: asaxton

---
# How to migrate Power BI Embedded workspace collection content to Power BI
Learn how to migrate from Power BI Embedded to the Power BI service and leverage advances for embedding in apps.

Microsoft recently [announced Power BI Premium](https://powerbi.microsoft.com/blog/microsoft-accelerates-modern-bi-adoption-with-power-bi-premium/), a new capacity-based licensing model that increases flexibility for how users access, share and distribute content. The offering also delivers additional scalability and performance to the Power BI service.

With the introduction of Power BI Premium, Power BI Embedded and the Power BI service are converging to advance how Power BI content is embedded in apps. This means you will have one API surface, a consistent set of capabilities and access to the latest Power BI features – such as dashboards, gateways and app workspaces – when embedding your content. Moving forward you’ll be able to start with Power BI Desktop and move to deployment with Power BI Premium, which will be generally available late in the second quarter of 2017.

The current Power BI Embedded service will continue to be available for a limited time following general availability of the converged offering: customers under an Enterprise Agreement will have access to through the expiration of their existing agreements; customers that acquired Power BI Embedded through Direct or CSP channels will enjoy access for one year from General Availability of Power BI Premium.  This article will provide some guidance for migrating from the Azure service to the Power BI service and what to expect for changes in your application.

> [!IMPORTANT]
> While the migration will take a dependency on the Power BI service, there is not a dependency on Power BI for the users of your application when using an **embed token**. They do not need to sign up for Power BI to view the embedded content in your application. You can use this embedding approach to service non-Power BI users.
> 
> 

![](media/powerbi-developer-migrate-from-powerbi-embedded/powerbi-embed-flow.png)

## Prepare for the migration
There are a few things you need to do to prepare for migrating from Power BI Embedded Azure service over to the Power BI service. You will need a tenant available, along with a user that has a Power BI Pro license.

1. Make sure you have access to an Azure Active Directory (Azure AD) tenant.
   
    You will need to determine what tenant setup to use.
   
   * Use your existing corporate Power BI tenant?
   * Use a separate tenant for your application?
   * Use a separate tenant for each customer?
     
     If you decide to create a new tenant for your application, or each customer, see [Create an Azure Active Directory tenant](powerbi-developer-create-an-azure-active-directory-tenant.md) or [How to get an Azure Active Directory tenant](https://docs.microsoft.com/azure/active-directory/develop/active-directory-howto-tenant).
2. Create a user within this new tenant that will act as your application "master" account. That account needs to sign up for Power BI and needs to have a Power BI Pro license assigned to it.

## Accounts within Azure AD
The following accounts will need to exist within your tenant.

> [!NOTE]
> These accounts will need to have Power BI Pro licenses in order to use App workspaces.
> 
> 

1. A tenant admin user.
   
    It is recommended that this user be a member of all App workspaces created for the purpose of embedding.
2. Accounts for analysts that will create content.
   
    These users should be assigned to App workspaces as needed.
3. An application *master* user account, or service account.
   
    The applications backend will store the credentials for this account and use it for acquiring an Azure AD token for use with the Power BI REST APIs. This account will be used to generate the embed token for the application. This account also needs to be an admin of the App workspaces created for embedding.
   
   > [!NOTE]
   > This is just a regular user account in your organziation that will be used for the purposes of embedding.
   > 
   > 

## App registration and permissions
You will need to register an application within Azure AD and grant certain permissions.

### Register an application
You will need to register your application with Azure AD in order to make REST API calls. This includes going to the Azure portal to apply additional configuration in addition to the Power BI app registration page. For more information, see [Register an Azure AD app to embed Power BI content](powerbi-developer-register-app.md).

You should register the application using the application **master** account.

## Create App workspaces (Required)
You can take advantage of App workspaces to provide better isoliation if your application is servicing multiple customers. Dashboards and reports would be isolated between your customers. You could then use a Power BI account per App workspace to further isolate application experiences between your customers.

> [!IMPORTANT]
> You cannot use a personal workspace to take advantage of embedding to non-Power BI users.
> 
> 

You will need a user that has a Pro license in order to create an app workspace within Power BI. The Power BI user that creates the App workspace will be an admin of that workspace by default.

> [!NOTE]
> The application *master* account needs to be an admin of the workspace.
> 
> 

## Content migration
Migrating your content from your workspace collections to the Power BI service can be done in parallel to your current solution and doesn’t require any downtime.

A **migration tool** is available for you to use in order to assist with copying content from Power BI Embedded to the Power BI service. Especially if you have a lot of content. For more information, see [Power BI Embedded migration tool](powerbi-developer-migrate-tool.md).

Content migration relies mainly on two APIs.

1. Download PBIX - this API can download PBIX files which were uploaded to Power BI after October 2016.
2. Import PBIX - this API uploads any PBIX to Power BI.

For some related code snippets, see [Code snippets for migrating content from Power BI Embedded](powerbi-developer-migrate-code-snippets.md).

### Report types
There are several types of reports, each requiring a somewhat different migration flow.

#### Cached dataset & report
Cached datasets refer to PBIX files that had imported data as opposed to a live connection or DirectQuery connection.

**Flow**

1. Call Download PBIX API from PaaS workspace.
2. Save PBIX.
3. Call Import PBIX to SaaS workspace.

#### DirectQuery dataset & report
**Flow**

1. Call GET https://api.powerbi.com/v1.0/collections/{collection_id}/workspaces/{wid}/datasets/{dataset_id}/Default.GetBoundGatewayDataSources and save connection string received.
2. Call Download PBIX API from PaaS workspace.
3. Save PBIX.
4. Call Import PBIX to SaaS workspace.
5. Update connection string by calling - POST  https://api.powerbi.com/v1.0/myorg/datasets/{dataset_id}/Default.SetAllConnections
6. Get GW id and datasource id by calling - GET https://api.powerbi.com/v1.0/myorg/datasets/{dataset_id}/Default.GetBoundGatewayDataSources
7. Update user's credentials by calling - PATCH https://api.powerbi.com/v1.0/myorg/gateways/{gateway_id}/datasources/{datasource_id}

#### Old dataset & reports
These are datasets/reports created before October 2016. Download PBIX doesn't support PBIXs which were uploaded before October 2016

**Flow**

1. Get PBIX from your development environment (your internal source control).
2. Call Import PBIX to SaaS workspace.

#### Push Dataset & report
Download PBIX doesn't support *Push API* datasets. Push API dataset data can't be ported from PaaS to SaaS.

**Flow**

1. Call "Create dataset" API with dataset Json to create dataset in SaaS workspace.
2. Rebuild report for the created dataset*.

It is possible using some workarounds to migrate the push api report from PaaS to SaaS by trying the following.

1. Uploading some dummy PBIX to PaaS workspace.
2. Clone the push api report and bind it to the dummy PBIX from step 1.
3. Download push API report with the dummy PBIX.
4. Upload dummy PBIX to your SaaS workspace.
5. Create push dataset in your SaaS workspace.
6. Rebind report to push api dataset.

## Create and upload new reports
In addition to the content you migrated from the Power BI Embedded Azure service, you can create your reports and datasets using Power BI Desktop and then publish those reports to an app workspace. The end user publishing the reports need to have a Power BI Pro license in order to publish to an app workspace.

## Rebuild your application
1. You will need to modify your application to use the Power BI REST APIs and the report location inside powerbi.com.
2. Rebuild your AuthN/AuthZ authentication using the *master* account for your application. You can take advantage of using an [embed token](https://msdn.microsoft.com/library/mt784614.aspx) to allow this user to act on behalf of other users.
3. Embed your reports from powerbi.com into your application.

## Map your users to a Power BI user
Within your application, you will map users that you manage within the application to a *master* Power BI credential for the purposes of your application. The credentials for this Power BI *master* account will be stored within your application and be used to creating embed tokens.

## What to do when you are ready for production
When you are ready to move to production, you will need to do the following.

* If you are using a separate tenant for development, then you will need to make sure your app workspaces, along with dashboards and reports, are available in your production environment. You will also need to make sure that you created the application in Azure AD for your production tenant and assigned the proper app permissions as indicated in Step 1.
* Purchase a capacity that fits your needs. You can use the [Embedded analytics capacity planning whitepaper](https://aka.ms/pbiewhitepaper) to help understand what you may need. When you are ready to purchase, you can do so within the [Office 365 admin center](https://portal.office.com/adminportal/home#/catalog).
  
  > [AZURE.INFORMATION] For information on how to purchase Power BI Premium, see [How to purchase Power BI Premium](powerbi-admin-premium-purchase.md).
  > 
  > 
* Edit the App workspace and assign it to a Premium capacity under advanced.
  
    ![](media/powerbi-developer-migrate-from-powerbi-embedded/powerbi-embedded-premium-capacity.png)
* Deploy your updated application to production and begin embedding reports from the Power BI service.

## After migration
You should do some cleanup within Azure.

* Remove all workspaces off of the deployed solution within the Azure service of Power BI Embedded.
* Delete any Workspace Collections that exist within Azure.

## Next steps
[Embedding with Power BI](powerbi-developer-embedding.md)  
[Power BI Embedded migration tool](powerbi-developer-migrate-tool.md)  
[Code snippets for migrating content from Power BI Embedded](powerbi-developer-migrate-code-snippets.md)  
[How to embed your Power BI dashboards, reports and tiles](powerbi-developer-embedding-content.md)  
[Power BI Premium - what is it?](powerbi-premium.md)  
[JavaScript API Git repo](https://github.com/Microsoft/PowerBI-JavaScript)  
[Power BI C# Git repo](https://github.com/Microsoft/PowerBI-CSharp)  
[JavaScript embed sample](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[Embedded analytics capacity planning whitepaper](https://aka.ms/pbiewhitepaper)  
[Power BI Premium whitepaper](https://aka.ms/pbipremiumwhitepaper)  

More questions? [Try asking the Power BI Community](http://community.powerbi.com/)
