---
title: Customize the user interface of your application using a custom policy in Azure Active Directory B2C | Microsoft Docs
description: Learn about customizing a user interface using a custom policy in Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: mtillman

ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/23/2018
ms.author: davidmu
ms.component: B2C
---
# Customize the user interface of your application using a custom policy in Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

After you complete this article, you will have a sign-up and sign-in custom policy with your brand and appearance. With Azure Active Directory B2C (Azure AD B2C), you get nearly full control of the HTML and CSS content that's presented to users. When you use a custom policy, you configure UI customization in XML instead of using controls in the Azure portal. 

## Prerequisites

Complete the steps in [Get started with custom policies](active-directory-b2c-get-started-custom.md). You should have a working custom policy for sign-up and sign-in with local accounts.

## Page UI customization

By using the page UI customization feature, you can customize the look and feel of any custom policy. You can also maintain brand and visual consistency between your application and Azure AD B2C.

Here's how it works: Azure AD B2C runs code in your customer's browser and uses a modern approach called [Cross-Origin Resource Sharing (CORS)](http://www.w3.org/TR/cors/). First, you specify a URL in the custom policy with customized HTML content. Azure AD B2C merges UI elements with the HTML content that's loaded from your URL and then displays the page to the customer.

## Create your HTML5 content

Create HTML content with your product's brand name in the title.

1. Copy the following HTML snippet. It is well-formed HTML5 with an empty element called *\<div id="api"\>\</div\>* located within the *\<body\>* tags. This element indicates where Azure AD B2C content is to be inserted.

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>My Product Brand Name</title>
   </head>
   <body>
       <div id="api"></div>
   </body>
   </html>
   ```

   >[!NOTE]
   >For security reasons, the use of JavaScript is currently blocked for customization.

2. Paste the copied snippet in a text editor, and then save the file as *customize-ui.html*.

## Create an Azure Blob storage account

>[!NOTE]
> In this article, we use Azure Blob storage to host our content. You can choose to host your content on a web server, but you must [enable CORS on your web server](https://enable-cors.org/server.html).

To host this HTML content in Blob storage, do the following:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. On the **Hub** menu, select **New** > **Storage** > **Storage account**.
3. Enter a unique **Name** for your storage account.
4. **Deployment model** can remain **Resource Manager**.
5. Change **Account Kind** to **Blob storage**.
6. **Performance** can remain **Standard**.
7. **Replication** can remain **RA-GRS**.
8. **Access tier** can remain **Hot**.
9. **Storage service encryption** can remain **Disabled**.
10. Select a **Subscription** for your storage account.
11. Create a **Resource group** or select an existing one.
12. Select the **Geographic location** for your storage account.
13. Click **Create** to create the storage account.  
    After the deployment is completed, the **Storage account** blade opens automatically.

## Create a container

To create a public container in Blob storage, do the following:

1. Click the **Overview** tab.
2. Click **Container**.
3. For **Name**, type **$root**.
4. Set **Access type** to **Blob**.
5. Click **$root** to open the new container.
6. Click **Upload**.
7. Click the folder icon next to **Select a file**.
8. Go to **customize-ui.html**, which you created earlier in the [Page UI customization](#the-page-ui-customization-feature) section.
9. Click **Upload**.
10. Select the customize-ui.html blob that you uploaded.
11. Next to **URL**, click **Copy**.
12. In a browser, paste the copied URL, and go to the site. If the site is inaccessible, make sure the container access type is set to **blob**.

## Configure CORS

Configure Blob storage for Cross-Origin Resource Sharing by doing the following:

1. In the menu, select **CORS**.
2. For **Allowed origins**, enter `your-tenant-name.b2clogin.com`. Replace `your-tenant-name` with the name of your Azure AD B2C tenant. For example, `fabrikam.b2clogin.com`. You need to use all lowercase letters when entering your tenant name.
3. For **Allowed Methods**, select both `GET` and `OPTIONS`.
4. For **Allowed Headers**, enter an asterisk (*).
5. For **Exposed Headers**, enter an asterisk (*).
6. For **Max age**, enter 200.
7. Click **Save**.

## Test CORS

Validate that you're ready by doing the following:

1. Go to the [www.test-cors.org](http://www.test-cors.org/) website, and then paste the URL in the **Remote URL** box.
2. Click **Send Request**.  
    If you receive an error, make sure that your [CORS settings](#configure-cors) are correct. You might also need to clear your browser cache or open an in-private browsing session by pressing Ctrl+Shift+P.

## Modify the extensions file

To configure UI customization, you copy the **ContentDefinition** and its child elements from the base file to the extensions file.

1. Open the base file of your policy. For example, *TrustFrameworkBase.xml*.
2. Search for and copy the entire contents of the **ContentDefinitions** element.
3. Open the extension file. For example, *TrustFrameworkExtensions.xml*. Search for the **BuildingBlocks** element. If the element doesn't exist, add it.
4. Paste the entire contents of the **ContentDefinitions** element that you copied as a child of the **BuildingBlocks** element. 
5. Search for the **ContentDefinition** element that contains `Id="api.signuporsignin"` in the XML that you copied.
6. Change the value of **LoadUri** to the URL of the HTML file that you uploaded to storage. For example, `https://mystore1.azurewebsites.net/b2c/customize-ui.html.
    
    Your custom policy should look like the following:

    ```xml
    <BuildingBlocks>
      <ContentDefinitions>
        <ContentDefinition Id="api.signuporsignin">
          <LoadUri>https://your-storage-account.blob.core.windows.net/your-container/customize-ui.html</LoadUri>
          <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
          <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.0.0</DataUri>
          <Metadata>
            <Item Key="DisplayName">Signin and Signup</Item>
          </Metadata>
        </ContentDefinition>
      </ContentDefinitions>
    </BuildingBlocks>
    ```

7. Save the extensions file.

## Upload your updated custom policy

1. Make sure you're using the directory that contains your Azure AD B2C tenant by clicking the **Directory and subscription filter** in the top menu and choosing the directory that contains your tenant.
3. Choose **All services** in the top-left corner of the Azure portal, and then search for and select **Azure AD B2C**.
4. Select **Identity Experience Framework**.
2. Click **All Policies**.
3. Click **Upload Policy**.
4. Upload the extensions file that you previously changed.

## Test the custom policy by using **Run now**

1. On the **Azure AD B2C** blade, go to **All policies**.
2. Select the custom policy that you uploaded, and click the **Run now** button.
3. You should be able to sign up by using an email address.

## Reference

You can find sample templates for UI customization here:

```
git clone https://github.com/azureadquickstarts/b2c-azureblobstorage-client
```

The sample_templates/wingtip folder contains the following HTML files:

| HTML5 template | Description |
|----------------|-------------|
| *phonefactor.html* | Use this file as a template for a multi-factor authentication page. |
| *resetpassword.html* | Use this file as a template for a forgot password page. |
| *selfasserted.html* | Use this file as a template for a social account sign-up page, a local account sign-up page, or a local account sign-in page. |
| *unified.html* | Use this file as a template for a unified sign-up or sign-in page. |
| *updateprofile.html* | Use this file as a template for a profile update page. |

In the [Modify your sign-up or sign-in custom policy section](#modify-your-sign-up-or-sign-in-custom-policy), you configured the content definition for `api.idpselections`. The full set of content definition IDs that are recognized by the Azure AD B2C identity experience framework and their descriptions are in the following table:

| Content definition ID | Description | 
|-----------------------|-------------|
| *api.error* | **Error page**. This page is displayed when an exception or an error is encountered. |
| *api.idpselections* | **Identity provider selection page**. This page contains a list of identity providers that the user can choose from during sign-in. These options are either enterprise identity providers, social identity providers such as Facebook and Google+, or local accounts. |
| *api.idpselections.signup* | **Identity provider selection for sign-up**. This page contains a list of identity providers that the user can choose from during sign-up. These options are either enterprise identity providers, social identity providers such as Facebook and Google+, or local accounts. |
| *api.localaccountpasswordreset* | **Forgot password page**. This page contains a form that the user must complete to initiate a password reset.  |
| *api.localaccountsignin* | **Local account sign-in page**. This page contains a sign-in form for signing in with a local account that is based on an email address or a user name. The form can contain a text input box and password entry box. |
| *api.localaccountsignup* | **Local account sign-up page**. This page contains a sign-up form for signing up for a local account that is based on an email address or a user name. The form can contain various input controls, such as a text input box, a password entry box, a radio button, single-select drop-down boxes, and multi-select check boxes. |
| *api.phonefactor* | **Multi-factor authentication page**. On this page, users can verify their phone numbers (by using text or voice) during sign-up or sign-in. |
| *api.selfasserted* | **Social account sign-up page**. This page contains a sign-up form that users must complete when they sign up by using an existing account from a social identity provider such as Facebook or Google+. This page is similar to the preceding social account sign-up page, except for the password entry fields. |
| *api.selfasserted.profileupdate* | **Profile update page**. This page contains a form that users can use to update their profile. This page is similar to the social account sign-up page, except for the password entry fields. |
| *api.signuporsignin* | **Unified sign-up or sign-in page**. This page handles both the sign-up and sign-in of users, who can use enterprise identity providers, social identity providers such as Facebook or Google+, or local accounts.  |

## Next steps

For additional information about UI elements that can be customized, see [reference guide for UI customization for built-in policies](active-directory-b2c-reference-ui-customization.md).
