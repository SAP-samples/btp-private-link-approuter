# SAP Private Link service with SAP Cloud Integration for building diverse integration scenarios

The SAP Cloud Integration capability of SAP Integration Suite enables enterprises to connect different systems and applications in hybrid and cloud landscapes, that are developed and maintained on different technology stacks. These stacks, usually follow different security standards and requirements. 

With the help of the SAP Private Link service, you can extend your hybrid integration scenarios to suit stricter security policies and communicate with your SAP S/4HANA on Microsoft Azure through private network connectivity. 

The main idea of this architecture is to use the Application Router as a proxy for the private connectivity between SAP S/4HANA and SAP Cloud Integration. Detailed configuration steps you can find below.

>Please note that for the moment, we cannot use SAP Private Link service directly from SAP Cloud Integration; nevertheless, we can bridge this gap with help of SAP’s Application Router (approuter), which can play the role of a proxy between SAP Private Link service and SAP Cloud Integration, meanwhile product team is working to enable direct integration.

![solution diagram](../img/approuter-cloudintegration.png)


## Configure and deploy Application Router 

You can configure and deploy the Application Router with the IDE of your choice, but with help of **SAP Business Application Studio (BAS)**, it is much simpler, since you can take advantage of the provided templates for your Application Router configuration. 

If you choose BAS, select **Standalone Approuter** as an option in the template wizard. 

![Approuter configuration](../img/approuter-config.png)

The only configuration required in the Application Router is to define the route and the destination used for the SAP Private Link connectivity. This can be done in the xs-app.json file where BusinessPartner-approuter is the destination configured for SAP Private Link connectivity in the target SAP BTP subaccount (see destination configuration below).  

 
```json
{ 
  "authenticationMethod": "route", 
  "routes": [ 
    { 
        "source": "^/sap/(.*)$", 
        "target": "/sap/$1", 
        "destination": "BusinessPartner-approuter", 
        "authenticationType": "xsuaa", 
        "csrfProtection": false 
    } 
  ] 
} 
```

![Destination configuration](../img/destination-config.png)

After setting up the route with destination and authentication, you can deploy the Application Router to your SAP BTP subaccount 
```json
e.g. by  mbt build and cf deploy privatelink-proxy.mtar 
```

Once the Application Router is up and running, it can be used by your integration flows to connect with the SAP backend system. 

## Use the Application Router proxy in SAP Cloud Integration iFlows

All your standard integration patterns can stay the same, you need to add a couple of additional steps to use the approuter proxy with the private link connectivity to your SAP backend system.

To showcase the scenario, in my example I’m simply calling one of the standard [S/4HANA APIs](https://api.sap.com/package/S4HANAOPAPI/all)

![iflow](../img/iflow.png)

Following additional steps are required:

1.	Get the access token from the XSUAA component of approuter. Please maintain the *client_id* and *client_secret* of XSUAA component as **"User Credentials"** in the **Security Material** of SAP Cloud Integration

 ![iflow xsuaa connection](../img/iflow-connection.png)

2.	Transform the response to XML (to add in a next step the custom header attribute)

  ![iflow converter](../img/iflow-converter.png)

3.	Modify the content by adding the header “x-approuter-authorization” (required by approuter) from the Authorization header value

  ![iflow modifier](../img/iflow-content-modifier.png)

4.	Make the final call to the approuter, which will route the request to S/4HANA System via Private Link connectivity)

  ![iflow s4 requiest](../img/iflow-s4-connection.png)

Having all these steps in place, you can deploy the integration flow and test it by calling your integration endpoint.

The example iFlow configuration you can get [here](iflow/PrivateLinkProxy.zip) and import to your SAP Cloud Integration tenant.

Go to your Cloud Integration cockpit and **import** the above-provided package.

  ![iflow import](../img/iflow-import.png)

Open the imported package

  ![iflow import](../img/iflow-import-open.png)

Open the integration flow

  ![iflow import](../img/iflow-open.png)

Configure the parameters based on your tenant

  ![iflow import](../img/iflow-configure.png)

Provide the XSUAA URL and Credential name from Security Material

  ![iflow import](../img/iflow-configure1.png)

Provide the Approuter URL

  ![iflow import](../img/iflow-configure2.png)

Deploy the iFlow

  ![iflow import](../img/iflow-deploy.png)

After deployment you can run your integration flow based on SAP Private Link connectivity.

## Summary

Congratulations, you successfully connected your hybrid integration flow with the SAP Private Link service



