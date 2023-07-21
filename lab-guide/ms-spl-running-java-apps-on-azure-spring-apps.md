## Hands-On Lab:  Migrate Java Web Apps to Azure App Service

Duration: 40 minutes

In this lab, you will learn how to build and deploy both frontend and backend Spring applications to Azure Spring Apps. In order to develop a high-level grasp of how to deploy and operate the same, you will first attempt to setup a very basic hello-world Spring Boot app. After that, you will configure Spring Cloud Gateway, deploy the frontend and backend apps of ACME-FITNESS (the demo application you will use in this lab), and verify that you can access the frontend as well as the backend. Additionally, you will change the Spring Cloud Gateway rules for these backend apps and set them up to communicate with the Application Configuration Service and the Service Registry.

### Task 1: Deploy a Hello World service to ASA-E 

In this task, you will try to deploy a very simple hello-world Spring Boot app to get a high-level understanding of how to deploy an ASA-E app and access it.

1. If you are not logged in already, click on the Azure portal shortcut that is available on the desktop and log in with the below Azure credentials.
    
    * **Azure Username/Email**: <inject key="AzureAdUserEmail"></inject> 
    * **Azure Password**: <inject key="AzureAdUserPassword"></inject>
    
1. Now, click on the start button, search for git and open **Git Bash**.  

     ![](Images/gitbash.png)                          

1. Once the Git Bash is open, please continue with the next step.

1. Run the following command to remove previous versions and install the latest Azure Spring Apps Enterprise tier extension:

    ```shell
    az extension remove --name spring-cloud
    az extension add --name spring
    ```
    
1. To change the directory to the sample app repository in your shell, run the following command in the Bash shell pane: 

    ```shell
    cd source-code/acme-fitness-store
    ```
    
1. Run the following command to create a bash script with environment variables by making a copy of the supplied template:

    ```shell
    cp ./azure/setup-env-variables-template.sh ./azure/setup-env-variables.sh
    ```

1. To open the `./scripts/./azure/setup-env-variables.sh` file, run the following command:

   ```shell
   cd azure
   code setup-env-variables.sh
   ```

1. Update the following variables in the setup-env-variables.sh file by replacing the following values and **Save** it using **Ctrl+S** key and **Close** the file:

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

1. Run the following command to move back to the acme-fitness-store directory and then set up the environment:
  
   ```shell
   cd ..
   source ./azure/setup-env-variables.sh
   ```   
  
1. Run the following command to log in to Azure:

   ```shell
   az login
   ```   
   
   > **Note:** Once you run the command, you will be redirected to the default browser. Enter the following:
   > - **Azure username:** <inject key="AzureAdUserEmail"></inject>  
   > - **Password:** <inject key="AzureAdUserPassword"></inject> 
   > 
   > Close the tab when you see the successful login message and proceed with the next command.


11. Run the following commands to get the list of subscriptions and to set your subscription:

    * Replace ${SUBSCRIPTION} with the SubscriptionID: **<inject key="Subscription Id" enableCopy="true"/>**

     ```shell
     az account list -o table
     az account set --subscription ${SUBSCRIPTION}
     ```     
    
    ![](Images/mjv2-4.png)
   
12. Now, run the following command to set your default resource group name and cluster name:

     ```shell
      az configure --defaults \
      group=${RESOURCE_GROUP} \
      location=${REGION} \
      spring=${SPRING_APPS_SERVICE}
     ```
    
13. To deploy the hello world app and create the Spring Boot application, run the following command and change the directory to hello world:

     ```shell
     cd hello-world/complete
     ```

14. Run the following command to create the **hello-world** app instance and deploy it to Azure Spring Apps Enterprise:

     ```shell
     az spring app create -n hello-world --assign-endpoint true
     mvn clean package -DskipTests
     az spring app deploy -n hello-world --artifact-path target/spring-boot-complete-0.0.1-SNAPSHOT.jar
     cd ..
     cd ..
     ```
 
      > **Note:** Creating and deploying the hello-world app will take around **2-3** minutes.

15. Return to the Azure portal in the browser and select **Resource groups** from the Azure services menu.

    ![acme-fitness](Images/L1-e1-s15.png)
    
16. Under the Resource groups page, select **Modernize-java-apps**.

    ![acme-fitness](Images/mja-verify-rg.png) 
    
17. Under your resource group page, select **azure-spring-apps-<inject key="DeploymentID" enableCopy="false" />** instance from the right-hand side under the Resources section.   

    ![acme-fitness](Images/mja-lab1-ex1-step17.png) 

18. Click on **Apps** under the **Settings** section of the navigation pane and select **hello-world**.

    ![acme-fitness](Images/hrlloword.png)

19. On the Overview page, find the **Test endpoint** under the **Essentials** section, and click on the **link** to browse the application.

    ![acme-fitness](Images/testend.png)
    
20. A new browser tab will open, and you should be able to see your **hello world** app successfully deployed. 
   
    ![acme-fitness](Images/Ex1-T2-S6.png)    
    

### Task 2: Deploy a Frontend Application

 In this section, you are going to deploy the frontend of ACME-FITNESS (the demo application that you will be using in this lab), configure it with Spring Cloud Gateway (SCG), and validate that you are able to access the frontend. You will create a Spring Cloud Gateway instance for Acme-Fitness and connect all the frontend and backend services to this gateway instance. This way, the gateway instance acts as the proxy for any requests that are targeted towards the Acme-Fitness application. Routing rules bind endpoints in the request to the backend applications. In the task below, you will also create a rule in SCG for the frontend app.

The diagram below shows the final result once this section is complete:

   ![](Images/upd-frontend.png)
    
> **Please note that we have already deployed the Azure Spring app and created the required frontend app to save time during the lab.**
 
1. To assign a public endpoint and update the Spring Cloud Gateway configuration with API information, run the following command:

   ```shell
   az spring gateway update --assign-endpoint true
   export GATEWAY_URL=$(az spring gateway show | jq -r '.properties.url')
    
   az spring gateway update \
    --api-description "Acme Fitness Store API" \
    --api-title "Acme Fitness Store" \
    --api-version "v1.0" \
    --server-url "https://${GATEWAY_URL}" \
    --allowed-origins "*" \
    --no-wait
   ```

    ![](Images/mjv2-7-new.png)
    
   > **Note:** Please be aware that the below commands can run for up to two minutes. Hold off until the commands have been completed.

1. Run the following command to create a routing rule for the frontend application:
   
   ```shell
   az spring gateway route-config create \
    --name ${FRONTEND_APP} \
    --app-name ${FRONTEND_APP} \
    --routes-file ./routes/frontend.json
   ```
   ![](Images/frontend-route.png)

1. Run the following command to deploy and build the frontend application with its required parameters:
   
   ```shell
   az spring app deploy --name ${FRONTEND_APP} \
    --source-path ./apps/acme-shopping 
   ```
   ![](Images/frontend-deploy.png)
   
   > **Note:** Deploying the application will take approximately **2-3** minutes.

1. Run the following command. Copy the output URL and paste it in a browser.

   ```shell
   echo "https://${GATEWAY_URL}"
   ```
   ![](Images/mjv2-10.png)
  
   > **Note:** If you see the ACME-FITNESS home page displayed as below, then it means that your frontend app and its corresponding route in SCG is configured correctly and deployed successfully. Explore the application, but notice that not everything is functioning yet. Continue on to the next exercise to configure the rest of the functionality.
    
   ![](Images/acme-fitness-homepage.png)
      

### Task 3: Deploy Backend Applications

In this section, you are going to deploy the backend apps for ACME-FITNESS application. You will also update the rules for these backend apps in Spring Cloud Gateway and configure these apps to talk to the Application Configuration Service and Service Registry. The Application Configuration Service is a feature of Azure Spring Apps Enterprise that makes Spring Apps configuration server capabilities available in a polyglot way. ASA-E internally uses the Tanzu Service Registry for dynamic service discovery.

The diagram below shows the final result once this section is complete:

   ![](Images/upd-scg-frontend-backend.png)

> **Please note that we have already deployed the Azure Spring app and created the required backend apps to save time during the lab.**

1. Run the following command to bind the spring applications to the Application Configuration Service:

   ```shell
   az spring application-configuration-service bind --app ${PAYMENT_SERVICE_APP}
   az spring application-configuration-service bind --app ${CATALOG_SERVICE_APP}
   ```
    
   ![](Images/mjv2-5.png)
   
   > **Note:** Please note that the above commands can run for up to two minutes. 

1. Run the following command to bind the spring applications to the Service Registry:

   ```shell
   az spring service-registry bind --app ${PAYMENT_SERVICE_APP}
   az spring service-registry bind --app ${CATALOG_SERVICE_APP}
   ```

   ![](Images/mjv2-6.png)

   > **Note:** Please note that the above commands can take up to two minutes to complete.

1. Run the following command to create routing rules for all backend applications:

    ```shell
    az spring gateway route-config create \
      --name ${CART_SERVICE_APP} \
      --app-name ${CART_SERVICE_APP} \
      --routes-file ./routes/cart-service.json
    
    az spring gateway route-config create \
      --name ${ORDER_SERVICE_APP} \
      --app-name ${ORDER_SERVICE_APP} \
      --routes-file ./routes/order-service.json

   az spring gateway route-config create \
      --name ${CATALOG_SERVICE_APP} \
      --app-name ${CATALOG_SERVICE_APP} \
      --routes-file ./routes/catalog-service.json
   ```
   
   > **Note:** Routing rules bind endpoints in the request to the backend applications. For example, in the Cart route below, the routing rule indicates any requests to /cart/** endpoint gets routed to the backend Cart App.

   ![](Images/upd-mjv2-8.png)
   
   > **Note:** Please note that the above commands can run for up to two minutes. 

1. Run the following command to deploy and build each backend application with its required parameters:

    ```shell
    # Deploy Payment Service
    az spring app deploy --name ${PAYMENT_SERVICE_APP} \
       --config-file-pattern payment/default \
       --source-path ./apps/acme-payment \
       --build-env BP_JVM_VERSION=17

    # Deploy Catalog Service
    az spring app deploy --name ${CATALOG_SERVICE_APP} \
       --config-file-pattern catalog/default \
       --source-path ./apps/acme-catalog \
       --build-env BP_JVM_VERSION=17

    # Deploy Order Service
    az spring app deploy --name ${ORDER_SERVICE_APP} \
       --source-path ./apps/acme-order 

    # Deploy Cart Service 
    az spring app deploy --name ${CART_SERVICE_APP} \
       --env "CART_PORT=8080" \
       --source-path ./apps/acme-cart 
    ```

   > **Note:** Deploying all applications will take approximately **10-15** minutes.

   ![](Images/mjv2-9-new.png)

1. Run the following command to get the Gateway URL.

   ```shell
   echo "https://${GATEWAY_URL}"
   ```
   ![](Images/mjv2-10.png)

1. Copy the Gateway URL and paste it in a new browser, and then you should see the ACME FITNESS Store application. Now that all the required apps are deployed, you should be able to open the home page and access it through the entire app. Explore the application, but notice that not everything is functioning yet. Continue to Lab 2 to configure Single Sign-On to enable the rest of the functionality (features like logging in, adding items to the cart, or placing an order).

   ![](Images/mjv2-11.png)

1. To assign an endpoint to API Portal, move back to Git Bash and run the following command:

   ```shell 
   az spring api-portal update --assign-endpoint true
   export PORTAL_URL=$(az spring api-portal show | jq -r '.properties.url')
   ```

1. Run the following command to get the API Portal URL.

   ```shell
   echo "https://${PORTAL_URL}"
   ```
   
   ![](Images/mjv2-12-new.png)
   
1. Copy the URL and paste it in a new browser, and then you should see the API portal for the ACME Fitness Store application.

    ![](Images/api1.png)   
    
 > **Note:** After finishing the exercise, be sure not to close the Git Bash window.

Now, click on **Next** in the lab guide section in the bottom right corner to jump to the next exercise instructions.
