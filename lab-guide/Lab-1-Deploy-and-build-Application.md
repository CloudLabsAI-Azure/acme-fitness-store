## Lab 1:  Deploy and Build Applications

In this lab, you will learn how to build and deploy Spring applications to Azure Spring Apps.

### Task 1: Prepare your environment for deployments

1. If you are not logged in already, click on the Azure portal shortcut that is available on the desktop and log in with the below Azure credentials.
    
    * Azure Username/Email: <inject key="AzureAdUserEmail"></inject> 
    * Azure Password: <inject key="AzureAdUserPassword"></inject>
    
1.  Click on the Cloud shell icon on the top right – > Next to the search bar.    

     ![](Images/L1-t1-s2.png)                          

1. Select on bash shell to launch. 

   ![](Images/upd-L1-t1-s3.png)

1. Under that you have no storage mounted pop-up, select **your subscription** (1) and then click on **Create storage** (2).

   ![](Images/upd-L1-t1-s4.png)   

1. Once the cloud drive is created, the cloud shell will be launched.

1. Run the following command to remove previous versions and install the latest Azure Spring Apps Enterprise tier extension:

    ```shell
    az extension remove --name spring-cloud
    az extension add --name spring
    ```
    
1. To create a new folder and clone the sample app repository to your Azure Cloud account, run the following command in the bash shell pane : 

    ```shell
    mkdir source-code
    cd source-code
    git clone https://github.com/Azure-Samples/acme-fitness-store
    cd acme-fitness-store
    ```
    
1. Create a bash script with environment variables by making a copy of the supplied template:

   ```shell
   cp ./azure/setup-env-variables-template.sh ./azure/setup-env-variables.sh
   ```

1. To open the `./scripts/./azure/setup-env-variables.sh` file, run the following command :

   ```shell
   cd azure
   vim setup-env-variables.sh
   ```

1. Update the following variables in the setup-env-variables.sh file by replacing the following values :

   ```shell
    export SUBSCRIPTION=subscription-id                 # replace it with your subscription-id
    export RESOURCE_GROUP=Modernize-java-apps-SUFFIX           # Repleace suffix with deploymentID from environment details page
    export SPRING_APPS_SERVICE=azure-spring-apps-SUFFIX   # Repleace suffix with deploymentID from environment details page
    export LOG_ANALYTICS_WORKSPACE=acme-log-analytic  
    export REGION=eastus                           # choose a region with Enterprise tier support
   ```
   >**Note:** You can copy the above values from the environment details page.
   
   > **Note:** You can use the arrow keys to move around in the file. Press the "i" key to enter insert mode to make the required changes. After making the changes press the escape "ESC" key to exit insert mode and then type ":wq" to save and close the file.
  
   ![](Images/mjv2-1.png)

1. Run the following command to move back to the acme-fitness-store directory and then set up the environment :
  
   ```shell
   cd ..
   source ./azure/setup-env-variables.sh
   ```   
  
1. Run the following command to log in to Azure:

    ```shell
   az login
   ```   

1. After running the above command you will be prompted with the code to authenticate like the code which is shown in the image below.

   ![](Images/mjv2-2.png)

1. Now open a browser and paste the link which you got after running the command "az login" and give the code to authenticate.

   ![](Images/mjv2-3.png)
   
1. Enter the azure credentials to sign in from the environment details page if necessary and click on **Continue**.

   ![](Images/azure1.png)

1. Run the following commands to get the list of subscriptions and also to set your subscription:

     ```shell
   az account list -o table
   az account set --subscription ${SUBSCRIPTION}
   ```     
   > **Note:** Replace ${SUBSCRIPTION} with the subscription Id which you can find on the Environment details page. 

   ![](Images/mjv2-4.png)
   
1. Now, run the following command to set your default resource group name and cluster name :

    ```shell
    az configure --defaults \
    group=${RESOURCE_GROUP} \
    location=${REGION} \
    spring=${SPRING_APPS_SERVICE}
    ```
  
### Task 2: Deploy a Hello World service to ASA-E 

In this task, you will try to deploy a very simple hello-world spring boot app to get a high level understanding of how to deploy an asa-e app and access it.

1. To deploy the hellpw world app and creating the Spring Boot application, run the following command :

   ```shell
   git clone https://github.com/spring-guides/gs-spring-boot.git
   ```

1. Run the following command to create a new file called HelloController.java in the hello-world directory and add the new Spring MVC Controller inside that file.

   ```shell
   cd gs-spring-bootrem
    cat >HelloController.java << EOF
    package com.example.demo;

    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    public class HelloController {

       @GetMapping("/hello")
       public String hello() {
         return "Hello from Azure Spring Apps Enterprise";
     }
    }
    EOF
    mv HelloController.java src/main/java/com/example/demo/HelloController.java
   cd gs-spring-boot/complete
   
   ```
1. Run the following command to create the 'hello-world' app instance and deploy it to Azure Spring Apps Enterprise:

   ```shell
   az spring app create -n hello-world --assign-endpoint true
   mvn clean package -DskipTests
   az spring app deploy -n hello-world --artifact-path target/spring-boot-complete-0.0.1-SNAPSHOT.jar
   cd ..
   ```

1. Now navigate back to the Azure portal and Look for your Azure Spring Apps instance in your resource group.

1. Click on "Apps" in the "Settings" section of the navigation pane and select "hello-world"

1. On the overview page, Find the "Test endpoint" in the "Essentials" section



### Task 3: Introduction to the Acme Fitness app 

This section discusses the demo application that you will be using in this lab to demonstrate the different features of ASA-E.

The below image shows the services involved in the ACME Fitness Store. It depicts the applications and their dependencies on different ASA-E services. We will be implementing this architecture by the end of this workshop.
![acme-fitness](Images/architecture.png)

This application is composed of several services:

* 3 Java Spring Boot applications:
  * A catalog service for fetching available products
  * A payment service for processing and approving payments for users' orders
  * An identity service for referencing the authenticated user

* 1 Python application:
  * A cart service for managing a users' items that have been selected for purchase

* 1 ASP.NET Core applications:
  * An order service for placing orders to buy products that are in the users' carts

* 1 NodeJS and static HTML Application
  * A frontend shopping application

### Task 4: Bind to Application Configuration Service 
The Application configuration service is a feature of azure spring apps enterprise that makes Spring Apps config server capabilities available in a polyglot way. Please note that we have already deployed the Azure Spring app and created the required Apps to save the time during the lab.

1. Run the following command to bind the spring applications to the Application Configuration Service:
 
   ```shell
    az spring application-configuration-service bind --app ${PAYMENT_SERVICE_APP}
    az spring application-configuration-service bind --app ${CATALOG_SERVICE_APP}
   ```
    
      ![](Images/mjv2-5.png)
    
 ### Task 5: Bind to Service Registry 
 
 Applications need to communicate with each other. ASA-E internally uses Tanzu Service Registry for dynamic service discovery.
 
 1.  Run the following command to bind the spring applications to the Service Registry:

     ```shell
     az spring service-registry bind --app ${PAYMENT_SERVICE_APP}
     az spring service-registry bind --app ${CATALOG_SERVICE_APP}
     ```

     ![](Images/mjv2-6.png)

### Task 6: Configure Spring Cloud Gateway 

In this task, you will create a spring cloud gateway instance for acme-fitness and connect all the backend services to this gateway instance. This way the gateway instance acts as the proxy for any requests that are targeted towards the acme-fitness application.

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

    ![](Images/mjv2-7.png)

1. Run the following command to create routing rules for the applications:

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

   az spring gateway route-config create \
      --name ${FRONTEND_APP} \
      --app-name ${FRONTEND_APP} \
      --routes-file ./routes/frontend.json
   ```

   ![](Images/mjv2-8.png)

### Task 7: Build and Deploy Polyglot Applications 

 In this task, you will deploy the more sophisticated acme-fitness application to the same asa-e instance.
 
 1. Run the following command to deploy and build each application with its required parameters:

    ```shell
    # Deploy Payment Service
    az spring app deploy --name ${PAYMENT_SERVICE_APP} \
       --config-file-pattern payment/default \
       --source-path ./apps/acme-payment 

    # Deploy Catalog Service
    az spring app deploy --name ${CATALOG_SERVICE_APP} \
       --config-file-pattern catalog/default \
       --source-path ./apps/acme-catalog 

    # Deploy Order Service
    az spring app deploy --name ${ORDER_SERVICE_APP} \
       --source-path ./apps/acme-order 

    # Deploy Cart Service 
    az spring app deploy --name ${CART_SERVICE_APP} \
       --env "CART_PORT=8080" \
       --source-path ./apps/acme-cart 

    # Deploy Frontend App
    az spring app deploy --name ${FRONTEND_APP} \
       --source-path ./apps/acme-shopping 
    ```

    > **Note:** Deploying all applications will take approximately 10-15 minutes.

    ![](Images/mjv2-9.png)

### Task 8: Access the Application through Spring Cloud Gateway 

1. Run the following command and then open the output from the following command in a browser:

   ```shell
   echo "https://${GATEWAY_URL}"
   ```
   ![](Images/mjv2-10.png)

1. Copy the gateway URL and paste it into a new browser and then you should see the ACME Fitness Store Application. Explore the application, but notice that not      everything is functioning yet. Continue to Lab 2 to configure Single Sign On to enable the rest of the functionality ( features like logging in, adding items to the cart, or completing an order).

   ![](Images/mjv2-11.png)

### Task 9: Explore the API using API Portal 

1. To assign an endpoint to API Portal, run the following command:

   ```shell 
   az spring api-portal update --assign-endpoint true
   export PORTAL_URL=$(az spring api-portal show | jq -r '.properties.url')
   ```

1. Run the following command and then open the output from the following command in a browser:

   ```shell
   echo "https://${PORTAL_URL}"
   ```
   
   ![](Images/mjv2-12.png)
   
1. Copy the URL and paste it in a new browser and then you should see the API portal of the ACME Fitness Store Application.

    ![](Images/api1.png)

Now, click on **Next** in the lab guide section in the bottom right corner to jump to the next exercise instructions.
