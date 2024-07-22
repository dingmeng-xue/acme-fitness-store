```
az account set --subscription 6c933f90-8115-4392-90f2-7077c9fa5dbd
RESOURCE_GROUP="dixue-aca-acme"
VNET_NAME="dixue-aca-acme-vnet"
LOCATION="westus2"
CONTAINERAPPS_ENVIRONMENT="dixue-aca-acme-env"
```

# create container registry
```sh
ACR_NAME="dixueacaacme"
az acr create -n $ACR_NAME -g $RESOURCE_GROUP --sku Standard --location $LOCATION

```

# Create ASA service
```sh
ASA_NAME="dixue-asae-acme"
# az spring create -n $ASA_NAME -g $RESOURCE_GROUP --sku Enterprise --disable-app-insights
```

```sh
az spring build-service build create -n frontend --source-path apps/acme-shopping --service $ASA_NAME -g $RESOURCE_GROUP
az spring build-service build create -n catalog-service --source-path apps/acme-catalog --service $ASA_NAME -g $RESOURCE_GROUP --build-env BP_JVM_VERSION=17
az spring build-service build create -n payment-service --source-path apps/acme-payment --service $ASA_NAME -g $RESOURCE_GROUP --build-env BP_JVM_VERSION=17
az spring build-service build create -n order-service --source-path apps/acme-order --service $ASA_NAME -g $RESOURCE_GROUP
az spring build-service build create -n identity-service --source-path apps/acme-identity --service $ASA_NAME -g $RESOURCE_GROUP
az spring build-service build create -n cart-service --source-path apps/acme-cart --service $ASA_NAME -g $RESOURCE_GROUP
```

# Setup network
```shell
az network vnet create --resource-group $RESOURCE_GROUP --name $VNET_NAME --location $LOCATION --address-prefix 10.0.0.0/16
az network vnet subnet create --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name infrastructure-subnet --address-prefixes 10.0.0.0/23

INFRASTRUCTURE_SUBNET=`az network vnet subnet show --resource-group ${RESOURCE_GROUP} --vnet-name $VNET_NAME --name infrastructure-subnet --query "id" -o tsv | tr -d '[:space:]'`

az containerapp env create --name $CONTAINERAPPS_ENVIRONMENT --resource-group $RESOURCE_GROUP --location $LOCATION --infrastructure-subnet-resource-id $INFRASTRUCTURE_SUBNET --internal-only
```

az containerapp create --name dixue-aca-acme-frontend --resource-group dixue-aca-acme --ingress external --target-port 8080 --environment managedEnvironment-dixueacaacme-9d31 --image dixueacaacme.azurecr.io/frontend:result --cpu 1 --memory 2Gi --registry-server dixueacaacme.azurecr.io --registry-username dixueacaacme --registry-password ${REGISTRY_PASSWORD}

az containerapp create --name dixue-aca-acme-cart-service --resource-group dixue-aca-acme --ingress external --target-port 8080 --environment managedEnvironment-dixueacaacme-9d31 --image dixueacaacme.azurecr.io/cart-service:result --cpu 1 --memory 2Gi --registry-server dixueacaacme.azurecr.io --registry-username dixueacaacme --registry-password ${REGISTRY_PASSWORD} --env-vars CART_PORT=8080

az containerapp create --name dixue-aca-acme-payment-service --resource-group dixue-aca-acme --ingress external --target-port 8080 --environment managedEnvironment-dixueacaacme-9d31 --image dixueacaacme.azurecr.io/payment-service:result --cpu 1 --memory 2Gi --registry-server dixueacaacme.azurecr.io --registry-username dixueacaacme --registry-password ${REGISTRY_PASSWORD}

az containerapp create --name dixue-aca-acme-catalog-service --resource-group dixue-aca-acme --ingress external --target-port 8080 --environment managedEnvironment-dixueacaacme-9d31 --image dixueacaacme.azurecr.io/catalog-service:result --cpu 1 --memory 2Gi --registry-server dixueacaacme.azurecr.io --registry-username dixueacaacme --registry-password ${REGISTRY_PASSWORD}

export ACAEnvName=managedEnvironment-dixueacaacme-9d31
export ACAEnvRGName=dixue-aca-acme

export ACAEnvDomain=$(az containerapp env show -n $ACAEnvName -g $ACAEnvRGName --query properties.defaultDomain --output tsv)
export ACAEnvStaticIP=$(az containerapp env show -n $ACAEnvName -g $ACAEnvRGName --query properties.staticIp --output tsv)

{
    "id": "/subscriptions/6c933f90-8115-4392-90f2-7077c9fa5dbd/resourceGroups/dixue-aca-acme/providers/Microsoft.App/managedEnvironments/managedEnvironment-dixueacaacme-9d31",
    "name": "managedEnvironment-dixueacaacme-9d31",
    "type": "Microsoft.App/managedEnvironments",
    "location": "West US 2",
    "systemData": {
        "createdBy": "dixue@microsoft.com",
        "createdByType": "User",
        "createdAt": "2024-07-19T03:34:06.2718173",
        "lastModifiedBy": "dixue@microsoft.com",
        "lastModifiedByType": "User",
        "lastModifiedAt": "2024-07-19T03:34:06.2718173"
    },
    "properties": {
        "provisioningState": "Succeeded",
        "daprAIInstrumentationKey": null,
        "daprAIConnectionString": null,
        "vnetConfiguration": null,
        "defaultDomain": "whitegrass-8b735982.westus2.azurecontainerapps.io",
        "staticIp": "172.179.180.243",
        "appLogsConfiguration": {
            "destination": "log-analytics",
            "logAnalyticsConfiguration": {
                "customerId": "b4282a75-731e-4dab-859b-33ad3dd49ea4",
                "sharedKey": null,
                "dynamicJsonColumns": false
            }
        },
        "openTelemetryConfiguration": null,
        "zoneRedundant": false,
        "kedaConfiguration": {
            "version": "2.14.0"
        },
        "daprConfiguration": {
            "version": "1.12.5"
        },
        "eventStreamEndpoint": "https://westus2.azurecontainerapps.dev/subscriptions/6c933f90-8115-4392-90f2-7077c9fa5dbd/resourceGroups/dixue-aca-acme/managedEnvironments/managedEnvironment-dixueacaacme-9d31/eventstream",
        "customDomainConfiguration": {
            "customDomainVerificationId": "95FA547AFB8E2C8DC531603DD229C9588F0C952B09163AB1382AFF8D65FAE794",
            "dnsSuffix": null,
            "certificateKeyVaultProperties": null,
            "certificateValue": null,
            "certificatePassword": null,
            "thumbprint": null,
            "subjectName": null,
            "expirationDate": null
        },
        "workloadProfiles": [
            {
                "workloadProfileType": "Consumption",
                "name": "Consumption"
            }
        ],
        "appInsightsConfiguration": null,
        "infrastructureResourceGroup": null,
        "peerAuthentication": {
            "mtls": {
                "enabled": false
            }
        }
    }
}