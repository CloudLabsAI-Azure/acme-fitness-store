## Lab 8: Automate from Idea to Production (Optional)

Duration: 15 minutes

### Task 1: Setup GitHub Account and Settings

1. From the new browser tab, go to [GitHub](https://github.com/) and log in to your account.
    > **Note:** If you don't have an account for GitHub, please sign up.

1. After the login, go to [https://github.com/CloudLabsAI-Azure/acme-fitness-store](https://github.com/CloudLabsAI-Azure/acme-fitness-store) and click on `Fork`.

   ![](Images/L8-t1-s2.png)
   
1. On the Create a new fork page, click on Create fork.   

### Task 2: Add Secrets to GitHub Actions

1. Now you're going to add the secrets to your repo.

1. From your repo, click on **Settings**.

   ![](Images/lab8.png)

1. Find **Secrets and variables** **(1)** under _Security_ on the left side of menu, and click on **Actions** **(2)**. After that Click on **New repository secret** **(3)**.
  
   ![](Images/secretsandvariables.png)
   
1. Type `AZURE_CREDENTIALS` **(1)** for the Name of the secret, enter the following code under Secret and make sure to replace the values of **ClientId (Application Id)**, **ClientSecret (Secret Key)**, **Subscription_ID** and **TenantId (Directory ID)** **(2)** and then click on **Add Secret** **(3)**.   

     ```json
    {
        "clientId": "Application_ID",
        "clientSecret": "Application_secret",
        "subscriptionId": "Subscription_ID",
        "tenantId": "TENANT_ID",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com/",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```
     
     >**Note:** You can copy the **ClientId (Application Id)**, **ClientSecret (Secret Key)**, **Subscription_ID** and **TenantId (Directory ID)** from the Environment details page > Service principal details.

   ![](Images/Ex8-T2-S4.png)

1. In a similar way, you will add the following secrets to GitHub Actions:

   | Secret Name | Secret value|
   |:----------|:--------|
   |`APP_SERVICE`| Provide app service name as **<inject key="Spring App Name" />**|
   | `RESOURCE_GROUP`| Provide the RG name **<inject key="Resource Group Name" />**|
   | `KEYVAULT`| Provide the Key vault name **<inject key="KeyVault Name" />**|
   | `AZURE_LOCATION` | Provide the region **<inject key="Region" />**|
   | `OIDC_JWK_SET_URI` | use the `JWK_SET_URI` |
   | `OIDC_CLIENT_ID` | use the `CLIENT_ID` |
   | `OIDC_CLIENT_SECRET` | use the `CLIENT_SECRET`|
   | `OIDC_ISSUER_URI` | use the `ISSUER_URI`|

    ![](Images/secrets-count.png)
 
    > **Note**: For the values of `OIDC_JWK_SET_URI`, `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET`, `OIDC_ISSUER_URI`, enter the values you have copied in your text editor in Lab 2.


1. Add the secret `TF_BACKEND_CONFIG` to GitHub Actions with the value (replacing `${STORAGE_ACCOUNT_NAME}` and `${STORAGE_RESOURCE_GROUP}` with **<inject key="Resource Group Name" />**):

   ```text
   resource_group_name  = "${STORAGE_RESOURCE_GROUP}"
   storage_account_name = "${STORAGE_ACCOUNT_NAME}"
   container_name       = "terraform-state-container"
   key                  = "dev.terraform.tfstate"
   ```

### Task 3: Run GitHub Actions

1. From your repo, click on **Actions**.

1. Select **Deploy catalog** (1) under __Actions_ All workflows_ from the left side panel and click on **Run workflow** (2). After that Click on **Run workflow** (3) under _Branch: Azure_.

   ![](Images/L8-t3-s2.png)

1. Each application has a `Deploy` workflow that will redeploy the application when changes are made to that application. An example output from the catalog service is shown below:

   ![Output from the Deploy Catalog workflow](Images/final-result.png)

1. Once the GitHub workflow is completed, navigate back to your web app to see the new **Catalog** option added to the website.


## Lab Ends
