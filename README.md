# Using vNet integration to secure Logic Apps preview #
This sample contains arm templates and application code for setting app two workflows in two Logic App Preview resources in a virtual network with the storage accounts locked into the vnet. 
* LogicAppFE is integrated with a vNet and configured so that all outbound traffic flow through the vNet and also setup with a storage account with private endpoints. However, this Logic App has a public endpoint through a request trigger workflow named “gateway”. 
* LogicAppBE is another workflow which has both vnet integration and private endpoint setup so that both inbound and outbound traffic are secured to vnet only. This app also has workflow with request trigger which can only be triggered from within the vNet.

To test how the traffic flows between these two apps, the gateway workflow in LogicAppFE is implemented with an HTTP action that will call into the request triggered workflow in LogicAppBE.

![LogicAppvNet](https://user-images.githubusercontent.com/3781206/105968012-647a3680-603b-11eb-9e9f-f38ca96c3850.jpg)

## Set up your repositories ##

Use the following repos as template to create your own copies:
* __logic-apps-samples__ contains the ARM templates and the LogicAppFE project and required github actions for deploying to azure.
* __logicapps-vnet-sample__ [repo](https://github.com/rohithah/logicapps-vnet-sample "LogicAppBE project") contains the LogicAppBE project. This sits on a separate repo so that we can use App service GitHub integration for deploying as cli based deployment will not work since the kudu endpoint is now inside the vnet.

## Provisioning resources ##

* Follow the instructions [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-github-actions "Deploy ARM template using Github actions") to generate deployment credentials for Github actions and store it as `AZURE_CREDENTIALS` in the newly created repo from logic-apps-samples as the template.

* Update the parameters in `LAv2-vNet/templates/azuredeploy.parameters.json` for specific values for your application. Given some of the resource names require to be globally unique, the default values will not work for you.

* Now choose action __provision-azure-resources-logic-app-vnet__ from the __Actions__ tab and provide the subscriptionId and resourceGroup to run the resource provisioning action. Note that the resource group should have been created. This will provision all the resources needed to setup the sample application described above.

## Deploying Logic App FE ##
* You can use the __build-and-deploy-logic-app-vnet__ action from actions tab to deploy the FE Logic app. You need to use the app name fro LogicAppFE app from your parameters file.

## Deploying the Logic App BE ##
* Given the LogicAppBE is locked down behind a vNet, You cannot use the same action you used for LogicAppFE for deploying the LogicAppBE. We can use the Deployment center [integration with GitHub](https://docs.microsoft.com/en-us/azure/azure-functions/functions-continuous-deployment "Continuous deployment for Logic Apps preview") to have the azure pull the app and the deploy to our app which is inside the vNet. Make sure you use the 'External Git' as the source.

## Configuring the LogicAppFE to call Logic App BE ##
There is one last step before our two apps are fully integrated. Get the callback url from the RequestHandler workflow from the LogicAppBE and set it as the value for __backendCallbackUrl__ in the LogicAppFE configurations. This completes setting up our application and we can invoke Gateway workflow in LogicAppFE with a public endpoint to call into the RequestHandler workflow in LogicAppBE which is completely locked down inside the vNet.

