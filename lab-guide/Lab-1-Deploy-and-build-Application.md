## Lab 1:  Deploy and Build Applications

Duration: 40 minutes

In this lab, you will learn how to build and deploy both frontend and backend Spring applications to Azure Spring Apps. In order to develop a high-level grasp of how to deploy and operate the same, you will first attempt to set up a very basic hello-world Spring Boot app. After that, you will configure Spring Cloud Gateway, deploy the frontend and backend apps of ACME-FITNESS (the demo application you will use in this lab), and verify that you can access the frontend as well as the backend. Additionally, you will change the Spring Cloud Gateway rules for these backend apps and set them up to communicate with the Application Configuration Service and the Service Registry.

### Task 1: Deploy Infrastructure Stack

In this task, you will try to deploy a very simple hello-world Spring Boot app to get a high-level understanding of how to deploy an ASA-E app and access it.

1. If you are not logged in already, click on the Azure portal shortcut that is available on the desktop and log in with the below Azure credentials.
    
    * **Azure Username/Email**: <inject key="AzureAdUserEmail"></inject> 
    * **Azure Password**: <inject key="AzureAdUserPassword"></inject>
    
2. Now, click on the start button, search for git and open **Git Bash**.  

     ![](Images/gitbash.png)                          

3. Once the Git Bash is open, please continue with the next step.

4. Run the following command to remove previous versions and install the latest Azure Spring Apps Enterprise tier extension.

```shell
  az extension remove --name spring-cloud
  az extension add --name spring
```
    
5. To change the directory to the sample app repository in your shell, run the following command in the Bash shell pane.

```shell
  cd source-code/acme-fitness-store/azure-spring-apps-enterprise/scripts
```
    
6. Run the following command to create a bash script with environment variables by making a copy of the supplied template.

```shell
  cp ./azure/setup-env-variables-template.sh ./azure/setup-env-variables.sh
```

7. To open the `./scripts/setup-env-variables.sh` file, run the following command.

```shell
  vi setup-env-variables.sh
```
   >**Note**: If you face any issues while editing the file please check at the bottom of the file if it is in edit mode or not, if not in edit mode please hit on **i** key of your keyboard to go to the edit mode. once the updates are done to save the file please do **ctrl+c** and write **:wq!**  this will save the file or if you dont want to save the file do **ctrl_c** and write **:q!**.

8. Update the following variables in the setup-env-variables.sh file by replacing the following values and **Save** it using **Ctrl+S** key and **Close** the file.

   * SubscriptionID: **<inject key="Subscription Id" enableCopy="true"/>**
   * Spring App Name: **<inject key="Spring App Name" enableCopy="true"/>**

```shell
  export SUBSCRIPTION=subscription-id                 # Replace it with your subscription-id 
  export RESOURCE_GROUP=Modernize-java-apps        
  export SPRING_APPS_SERVICE=azure-spring-apps-SUFFIX   # Replace suffix with the provided DeploymentID
  export LOG_ANALYTICS_WORKSPACE=acme-log-analytic  
  export REGION=eastus                          
```
   >**Note:** Leave default values for RESOURCE_GROUP, LOG_ANALYTICS_WORKSPACE and REGION.
   
   ![](Images/mja-setup-env-variables.png)

9. Run the following command to move back to the acme-fitness-store directory and then set up the environment.
  
```shell
  chmod +x ./setup-env-variables.sh
  source ./setup-env-variables.sh
``` 
  
10. Run the following command to log in to Azure.

```shell
  az login
```   
   
   > **Note:** Once you run the command, you will be redirected to the default browser. Enter the following:
   > - **Azure username:** <inject key="AzureAdUserEmail"></inject>  
   > - **Password:** <inject key="AzureAdUserPassword"></inject> 
   > 
   > Close the tab when you see the successful login message and proceed with the next command.


11. Run the following commands to get the list of subscriptions and to set your subscription.

    * Replace ${SUBSCRIPTION} with the SubscriptionID: **<inject key="Subscription Id" enableCopy="true"/>**

```shell
   az account list -o table
   az account set --subscription ${SUBSCRIPTION}
```  
    
   ![](Images/mjv2-4.png)
   
12. Now, run the following command to set your default resource group name and cluster name.

```shell
  az configure --defaults \
  group=${RESOURCE_GROUP} \
  location=${REGION} \
  spring=${SPRING_APPS_SERVICE}
```
    
   > **Note:** Make sure you are in the **scripts** directory.
13. Run the following command to create the instance of Azure Spring Apps Enterprise.

```shell
az spring create --name ${SPRING_APPS_SERVICE} \
    --resource-group ${RESOURCE_GROUP} \
    --location ${REGION} \
    --sku Enterprise \
    --enable-application-configuration-service \
    --enable-service-registry \
    --enable-gateway \
    --enable-api-portal \
    --enable-alv \
    --enable-app-acc \
    --build-pool-size S2 
```
   > **Note:** Creating the instance will take around **2-3** minutes.

### Task 2: Configure Log Analytics for Azure Spring Apps

1. Create a Log Analytics Workspace to be used for your Azure Spring Apps service.

> Note: This step can be skipped if using an existing workspace

```shell
az configure --defaults \
    group=${RESOURCE_GROUP} \
    location=${REGION} \
    spring=${SPRING_APPS_SERVICE}
```

2. Retrieve the resource ID for the recently create Azure Spring Apps Service and Log Analytics Workspace.

```shell
export LOG_ANALYTICS_RESOURCE_ID=$(az monitor log-analytics workspace show \
    --resource-group ${RESOURCE_GROUP} \
    --workspace-name ${LOG_ANALYTICS_WORKSPACE} \
    --query id \
    -o tsv)

export SPRING_APPS_RESOURCE_ID=$(az spring show \
    --name ${SPRING_APPS_SERVICE} \
    --resource-group ${RESOURCE_GROUP} \
    --query id \
    -o tsv)
```

3. Configure diagnostic settings for the Azure Spring Apps Service.

```shell
az monitor diagnostic-settings create --name "send-logs-and-metrics-to-log-analytics" \
    --resource ${SPRING_APPS_RESOURCE_ID} \
    --workspace ${LOG_ANALYTICS_RESOURCE_ID} \
    --logs '[
         {
           "category": "ApplicationConsole",
           "enabled": true,
           "retentionPolicy": {
             "enabled": false,
             "days": 0
           }
         },
         {
            "category": "SystemLogs",
            "enabled": true,
            "retentionPolicy": {
              "enabled": false,
              "days": 0
            }
          },
         {
            "category": "IngressLogs",
            "enabled": true,
            "retentionPolicy": {
              "enabled": false,
              "days": 0
             }
           }
       ]' \
       --metrics '[
         {
           "category": "AllMetrics",
           "enabled": true,
           "retentionPolicy": {
             "enabled": false,
             "days": 0
           }
         }
       ]'
```

> Note: For Git Bash users, this command may fail when resource IDs are misinterpreted as file paths because they begin with `/`. 
> 
> If the above command fails, try setting MSYS_NO_PATHCONV using:
> 
> `run this command on the terminal **export MSYS_NO_PATHCONV=1** and rerun the diagonstic configure command.

### Task 3: Configure Application Configuration Service

1. Create a configuration repository for Application Configuration Service using the Azure CLI.

```shell
az spring application-configuration-service git repo add --name acme-fitness-store-config \
    --label main \
    --patterns "catalog/default,catalog/key-vault,identity/default,identity/key-vault,payment/default" \
    --uri "https://github.com/Azure-Samples/acme-fitness-store-config"
```

### Task 4: Configure Tanzu Build Service


1. Make sure you are operating from the ./scripts folder

```shell
pwd
```
> Should show something like:

```
./source-code/acme-fitness-store/azure-spring-apps-enterprise/scripts
```

2. Create a custom builder in Tanzu Build Service using the Azure CLI.

```shell
az spring build-service builder create -n ${CUSTOM_BUILDER} \
    --builder-file ../resources/json/tbs/builder.json \
    --no-wait
```

### Task 5: Create applications in Azure Spring Apps

1. Create an application for each service.

```shell
az spring app create --name ${CART_SERVICE_APP} --instance-count 1 --memory 1Gi &
az spring app create --name ${ORDER_SERVICE_APP} --instance-count 1 --memory 1Gi &
az spring app create --name ${PAYMENT_SERVICE_APP} --instance-count 1 --memory 1Gi &
az spring app create --name ${CATALOG_SERVICE_APP} --instance-count 1 --memory 1Gi &
```

2. Then, create an app for the Front End.

```shell
az spring app create --name ${FRONTEND_APP} --instance-count 1 --memory 1Gi
```

> At this time, wait until control is passed back to your console before proceeding. Please check the Portal to make sure ALL services (4 Services & Frontend App) are created.  Should look something like:



### Task 6: Bind to Application Configuration Service

1. Several applications require configuration from Application Configuration Service, so create the bindings.

```shell
az spring application-configuration-service bind --app ${PAYMENT_SERVICE_APP}
az spring application-configuration-service bind --app ${CATALOG_SERVICE_APP}
```

### Task 7: Bind to Service Registry

1. Several application require service discovery using Service Registry, so create the bindings.

```shell
az spring service-registry bind --app ${PAYMENT_SERVICE_APP}
az spring service-registry bind --app ${CATALOG_SERVICE_APP}
```

### Task 8: Configure Spring Cloud Gateway

1. Assign an endpoint and update the Spring Cloud Gateway configuration with API information.

```shell
az spring gateway update --assign-endpoint true
export GATEWAY_URL=$(az spring gateway show --query properties.url -o tsv)
    
az spring gateway update \
    --api-description "Acme Fitness Store API" \
    --api-title "Acme Fitness Store" \
    --api-version "v1.0" \
    --server-url "https://${GATEWAY_URL}" \
    --allowed-origins "*" \
    --no-wait
```

### Task 9: Create  routing rules for the applications:

1. Copy the commmand and paste into the terminal to configure the routing.

```shell
az spring gateway route-config create \
    --name ${CART_SERVICE_APP} \
    --app-name ${CART_SERVICE_APP} \
    --routes-file ../resources/json/routes/cart-service.json
    
az spring gateway route-config create \
    --name ${ORDER_SERVICE_APP} \
    --app-name ${ORDER_SERVICE_APP} \
    --routes-file ../resources/json/routes/order-service.json

az spring gateway route-config create \
    --name ${CATALOG_SERVICE_APP} \
    --app-name ${CATALOG_SERVICE_APP} \
    --routes-file ../resources/json/routes/catalog-service.json

az spring gateway route-config create \
    --name ${FRONTEND_APP} \
    --app-name ${FRONTEND_APP} \
    --routes-file ../resources/json/routes/frontend.json
```

### Task 10: Build and Deploy Polyglot Applications

1. Deploy and build each application, specifying its required parameters.

```shell
# Deploy Payment Service
az spring app deploy --name ${PAYMENT_SERVICE_APP} \
    --config-file-pattern payment/default \
    --source-path ../../apps/acme-payment \
    --build-env BP_JVM_VERSION=17

# Deploy Catalog Service
az spring app deploy --name ${CATALOG_SERVICE_APP} \
    --config-file-pattern catalog/default \
    --source-path ../../apps/acme-catalog \
    --build-env BP_JVM_VERSION=17

# Deploy Order Service
az spring app deploy --name ${ORDER_SERVICE_APP} \
    --builder ${CUSTOM_BUILDER} \
    --source-path ../../apps/acme-order 

# Deploy Cart Service 
az spring app deploy --name ${CART_SERVICE_APP} \
    --builder ${CUSTOM_BUILDER} \
    --env "CART_PORT=8080" \
    --source-path ../../apps/acme-cart 

# Deploy Frontend App
az spring app deploy --name ${FRONTEND_APP} \
    --builder ${CUSTOM_BUILDER} \
    --source-path ../../apps/acme-shopping 
```

> Note: Deploying all applications will take 5-10 minutes

### Task 11: Access the Application through Spring Cloud Gateway

1. Retrieve the URL for Spring Cloud Gateway and open it in a browser, paste the command in terminal and copy the url and browse.

```shell
echo "https://${GATEWAY_URL}"
```

> You should see the ACME Fitness Store Application

### Task 12: Explore the API using API Portal

1. Assign an endpoint to API Portal and open it in a browser.

```shell
az spring api-portal update --assign-endpoint true
export PORTAL_URL=$(az spring api-portal show --query properties.url -o tsv)

echo "https://${PORTAL_URL}"
```

> Now, click on **Next** in the lab guide section in the bottom right corner to jump to the next exercise instructions.
