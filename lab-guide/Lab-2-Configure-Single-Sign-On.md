## Lab 2: Configure Single Sign-On

Duration: 30 minutes
 
In this lab, you will configure Single Sign-On for Spring Cloud Gateway using Microsoft Entra ID.

### Task 1: Register Application with Microsoft Entra ID

1. Choose a unique display name for your Application Registration.

```shell
export AD_DISPLAY_NAME=change-me    # unique application display name
```

2. Create an Application registration with Microsoft Entra ID and save the output.

```shell
az ad app create --display-name ${AD_DISPLAY_NAME} > ../resources/json/ad.json
```

3. Retrieve the Application ID and collect the client secret.

```shell
export APPLICATION_ID=$(cat ../resources/json/ad.json | jq -r '.appId')

az ad app credential reset --id ${APPLICATION_ID} --append > ../resources/json/sso.json
```

4. Assign a Service Principal to the Application Registration.

```shell
az ad sp create --id ${APPLICATION_ID}
```

5. Prepare your environment for SSO Deployments and Set the environment using the provided script and verify the environment variables are set.

```shell
chmod +x ./setup-sso-variables-ad.sh
source ./setup-sso-variables-ad.sh
```

6. Echo the following values.

```shell
echo ${CLIENT_ID}
echo ${CLIENT_SECRET}
echo ${ISSUER_URI}
echo ${JWK_SET_URI}
echo ${GATEWAY_URL}
echo ${PORTAL_URL}
```

7. Verify the urls should look similiar to below example urls.

> The `ISSUER_URI` should take the form `https://login.microsoftonline.com/${TENANT_ID}/v2.0`
> The `JWK_SET_URI` should take the form `https://login.microsoftonline.com/${TENANT_ID}/discovery/v2.0/keys`

8. To add the necessary web redirect URIs to App Registration in Microsoft Entra ID.

```shell
az ad app update --id ${APPLICATION_ID} \
    --web-redirect-uris "https://${GATEWAY_URL}/login/oauth2/code/sso" "https://${PORTAL_URL}/oauth2-redirect.html" "https://${PORTAL_URL}/login/oauth2/code/sso"
```

### Task 2: Using an Existing SSO Identity Provider

1. To use an existing SSO Identity Provider, copy the existing template. Again, make sure you are operating from the ./scripts folder.

```shell
pwd
```

2. Next, make a copy of setup-sso-variables-template.sh for your custom values.

```shell
cp ./setup-sso-variables-template.sh ./setup-sso-variables.sh
```

3. Echo the following values.

```shell
echo ${CLIENT_ID}
echo ${CLIENT_SECRET}
echo ${ISSUER_URI}
echo ${JWK_SET_URI}
```

4. Edit the copied file.

```
vi setup-sso-variables.sh 
```

5. Add the required values.

```shell
export CLIENT_ID="change-me"        # Your SSO Provider Client ID
export CLIENT_SECRET="change-me"    # Your SSO Provider Client Secret
export ISSUER_URI="change-me"       # Your SSO Provider Issuer URI
export JWK_SET_URI="change-me"      # Your SSO Provider Json Web Token URI
```

>The `JWK_SET_URI` typically takes the form `${ISSUER_URI}/$VERSION/keys`

6. Set your copy as "executable".

```shell
chmod +x setup-sso-variables.sh
```

7. Set the environment variables.

```shell
source ./setup-sso-variables.sh
```

8. Add the following to your SSO provider's list of approved redirect URIs.

```shell
echo "https://${GATEWAY_URL}/login/oauth2/code/sso"
echo "https://${PORTAL_URL}/oauth2-redirect.html" 
echo "https://${PORTAL_URL}/login/oauth2/code/sso"
```

### Task 3: Configure Spring Cloud Gateway with SSO

1. Configure Spring Cloud Gateway with SSO enabled.

```shell
az spring gateway update \
    --client-id ${CLIENT_ID} \
    --client-secret ${CLIENT_SECRET} \
    --scope ${SCOPE} \
    --issuer-uri ${ISSUER_URI} \
    --no-wait
```

2. Setup, Configure, and Deploy the identity service application.

```shell
az spring app create --name ${IDENTITY_SERVICE_APP} --instance-count 1 --memory 1Gi
```

3. Bind the identity service to Application Configuration Service.

```shell
az spring application-configuration-service bind --app ${IDENTITY_SERVICE_APP}
```

4. Bind the identity service to Service Registry.

```shell
az spring service-registry bind --app ${IDENTITY_SERVICE_APP}
```

5. Create routing rules for the identity service application.

```shell
az spring gateway route-config create \
    --name ${IDENTITY_SERVICE_APP} \
    --app-name ${IDENTITY_SERVICE_APP} \
    --routes-file ../resources/json/routes/identity-service.json
```

6. Deploy the Identity Service.

```shell
az spring app deploy --name ${IDENTITY_SERVICE_APP} \
    --env "JWK_URI=${JWK_SET_URI}" \
    --config-file-pattern identity/default \
    --source-path ../../apps/acme-identity \
    --build-env BP_JVM_VERSION=17
```

> **Note:** The application will take around 3-5 minutes to deploy.

### Task 4: Update Existing Applications

1. Update the existing applications to use authorization information from Spring Cloud Gateway.

```shell
# Update the Cart Service
az spring app update --name ${CART_SERVICE_APP} \
    --env "AUTH_URL=https://${GATEWAY_URL}" "CART_PORT=8080" 
    
# Update the Order Service
az spring app update --name ${ORDER_SERVICE_APP} \
    --env "AcmeServiceSettings__AuthUrl=https://${GATEWAY_URL}" 
```

2. Login to the Application through Spring Cloud Gateway, retrieve the URL for Spring Cloud Gateway and open it in a browser.

```shell
echo "https://${GATEWAY_URL}"
```
> **Note:** If you get any popup for accept the terms of entra registered apps click on **Accept**.

3. Configure SSO for API Portal.

```shell
export PORTAL_URL=$(az spring api-portal show --query properties.url -o tsv)

az spring api-portal update \
    --client-id ${CLIENT_ID} \
    --client-secret ${CLIENT_SECRET}\
    --scope "openid,profile,email" \
    --issuer-uri ${ISSUER_URI}
```

4. Explore the API using API Portal, Open API Portal in a browser this will redirect you to log in now if you are not already logged into the browser.

```shell
echo "https://${PORTAL_URL}"
```

> Now, click on **Next** in the lab guide section in the bottom right corner to jump to the next exercise instructions.
