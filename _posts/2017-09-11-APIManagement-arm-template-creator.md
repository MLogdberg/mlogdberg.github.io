---
layout: post
title: API Management ARM Template Creator
author: Mattias Lögdberg
tags: [API Management, Integration, ARM, Build & Release]
categories: [API Management]
image: 
description: 
permalink: /apimanagement/arm-template-creator
---

ARM support was release in the July rlease of API Management ([read more here](https://blogs.msdn.microsoft.com/apimanagement/2017/07/14/release-notes-july-14th-2017/)) and this is relly a great improvement on the deployment experience of API Management.
For samples on ARM templates released by the product group got the [Sample Github page](https://github.com/Azure/azure-quickstart-templates/tree/master/201-api-management-create-all-resources)

But there is some "blocking" things here, we are not using ARM templates when devloping, so how do we get the ARM Templates?
First you can download the entire API Management instance ARM specification directly in the portal, **Automation Script**
[![](/assets/uploads/2017/09/apim-arm-template-creator-automation-script.png)](/assets/uploads/2017/09/apim-arm-template-creator-automation-script.png)

But when starting to look into this I got a little dissapointed, it's to mutch hardcoded values like backend URL's parameters and there is for me unwanted things aswell as Users, Groups, Subscriptions etc. All that could be sorted out and scripted (alot of work but could be done) but the real deal breaker using the automation script was the missing request/response representation and the missing *queryParameters* wich lead to crahsing imports.
It's import to understand the Environment reference and boundries, we donät want cross reference communcation that is not intended:
[![](/assets/uploads/2017/09/apim-arm-template-creator-environments.png)](/assets/uploads/2017/09/apim-arm-template-creator-environments.png)



When doing a deployment we want as little manual work as possible and above we have 2 issues, manual configuration equals downtime and sample messagas is one of the best things when onboarding new API consumers we can't ignore this.  

Let's look at an example, this is how an operation is looking:
[![](/assets/uploads/2017/09/apim-arm-template-creator-currency-convererter-operation-gui.png)](/assets/uploads/2017/09/apim-arm-template-creator-currency-convererter-operation-gui.png)

But when exported via the **Automation Script** it's not there:

```
 {
    "comments": "Generalized from resource: '/subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/apis/58a3138ca9f54041296baffd/operations/58a313e40647c00eecf8d408'.",
    "type": "Microsoft.ApiManagement/service/apis/operations",
    "name": "[concat(parameters('service_ibizmalo_name'), '/', parameters('apis_58a3138ca9f54041296baffd_name'), '/', parameters('operations_58a313e40647c00eecf8d408_name'))]",
    "apiVersion": "2017-03-01",
    "scale": null,
    "properties": {
        "displayName": "Conversion Rate",
        "method": "POST",
        "urlTemplate": "/conversionrate",
        "description": "<br><b>Get conversion rate from one currency to another currency <b><br><p><b><font color='#000080' size='1' face='Verdana'><u>Differenct currency Code and Names around the world</u></font></b></p><blockquote><p><font face='Verdana' size='1'>AFA-Afghanistan Afghani<br>ALL-Albanian Lek<br>DZD-Algerian Dinar<br>ARS-Argentine Peso<br>AWG-Aruba Florin<br>AUD-Australian Dollar<br>BSD-Bahamian Dollar<br>BHD-Bahraini Dinar<br>BDT-Bangladesh Taka<br>BBD-Barbados Dollar<br>BZD-Belize Dollar<br>BMD-Bermuda Dollar<br>BTN-Bhutan Ngultrum<br>BOB-Bolivian Boliviano<br>BWP-Botswana Pula<br>BRL-Brazilian Real<br>GBP-British Pound<br>BND-Brunei Dollar<br>BIF-Burundi Franc<br>XOF-CFA Franc (BCEAO)<br>XAF-CFA Franc (BEAC)<br>KHR-Cambodia Riel<br>CAD-Canadian Dollar<br>CVE-Cape Verde Escudo<br>KYD-Cayman Islands Dollar<br>CLP-Chilean Peso<br>CNY-Chinese Yuan<br>COP-Colombian Peso<br>KMF-Comoros Franc<br>CRC-Costa Rica Colon<br>HRK-Croatian Kuna<br>CUP-Cuban Peso<br>CYP-Cyprus Pound<br>CZK-Czech Koruna<br>DKK-Danish Krone<br>DJF-Dijibouti Franc<br>DOP-Dominican Peso<br>XCD-East Caribbean Dollar<br>EGP-Egyptian Pound<br>SVC-El Salvador Colon<br>EEK-Estonian Kroon<br>ETB-Ethiopian Birr<br>EUR-Euro<br>FKP-Falkland Islands Pound<br>GMD-Gambian Dalasi<br>GHC-Ghanian Cedi<br>GIP-Gibraltar Pound<br>XAU-Gold Ounces<br>GTQ-Guatemala Quetzal<br>GNF-Guinea Franc<br>GYD-Guyana Dollar<br>HTG-Haiti Gourde<br>HNL-Honduras Lempira<br>HKD-Hong Kong Dollar<br>HUF-Hungarian Forint<br>ISK-Iceland Krona<br>INR-Indian Rupee<br>IDR-Indonesian Rupiah<br>IQD-Iraqi Dinar<br>ILS-Israeli Shekel<br>JMD-Jamaican Dollar<br>JPY-Japanese Yen<br>JOD-Jordanian Dinar<br>KZT-Kazakhstan Tenge<br>KES-Kenyan Shilling<br>KRW-Korean Won<br>KWD-Kuwaiti Dinar<br>LAK-Lao Kip<br>LVL-Latvian Lat<br>LBP-Lebanese Pound<br>LSL-Lesotho Loti<br>LRD-Liberian Dollar<br>LYD-Libyan Dinar<br>LTL-Lithuanian Lita<br>MOP-Macau Pataca<br>MKD-Macedonian Denar<br>MGF-Malagasy Franc<br>MWK-Malawi Kwacha<br>MYR-Malaysian Ringgit<br>MVR-Maldives Rufiyaa<br>MTL-Maltese Lira<br>MRO-Mauritania Ougulya<br>MUR-Mauritius Rupee<br>MXN-Mexican Peso<br>MDL-Moldovan Leu<br>MNT-Mongolian Tugrik<br>MAD-Moroccan Dirham<br>MZM-Mozambique Metical<br>MMK-Myanmar Kyat<br>NAD-Namibian Dollar<br>NPR-Nepalese Rupee<br>ANG-Neth Antilles Guilder<br>NZD-New Zealand Dollar<br>NIO-Nicaragua Cordoba<br>NGN-Nigerian Naira<br>KPW-North Korean Won<br>NOK-Norwegian Krone<br>OMR-Omani Rial<br>XPF-Pacific Franc<br>PKR-Pakistani Rupee<br>XPD-Palladium Ounces<br>PAB-Panama Balboa<br>PGK-Papua New Guinea Kina<br>PYG-Paraguayan Guarani<br>PEN-Peruvian Nuevo Sol<br>PHP-Philippine Peso<br>XPT-Platinum Ounces<br>PLN-Polish Zloty<br>QAR-Qatar Rial<br>ROL-Romanian Leu<br>RUB-Russian Rouble<br>WST-Samoa Tala<br>STD-Sao Tome Dobra<br>SAR-Saudi Arabian Riyal<br>SCR-Seychelles Rupee<br>SLL-Sierra Leone Leone<br>XAG-Silver Ounces<br>SGD-Singapore Dollar<br>SKK-Slovak Koruna<br>SIT-Slovenian Tolar<br>SBD-Solomon Islands Dollar<br>SOS-Somali Shilling<br>ZAR-South African Rand<br>LKR-Sri Lanka Rupee<br>SHP-St Helena Pound<br>SDD-Sudanese Dinar<br>SRG-Surinam Guilder<br>SZL-Swaziland Lilageni<br>SEK-Swedish Krona<br>TRY-Turkey Lira<br>CHF-Swiss Franc<br>SYP-Syrian Pound<br>TWD-Taiwan Dollar<br>TZS-Tanzanian Shilling<br>THB-Thai Baht<br>TOP-Tonga Pa'anga<br>TTD-Trinidad&amp;amp;Tobago Dollar<br>TND-Tunisian Dinar<br>TRL-Turkish Lira<br>USD-U.S. Dollar<br>AED-UAE Dirham<br>UGX-Ugandan Shilling<br>UAH-Ukraine Hryvnia<br>UYU-Uruguayan New Peso<br>VUV-Vanuatu Vatu<br>VEB-Venezuelan Bolivar<br>VND-Vietnam Dong<br>YER-Yemen Riyal<br>YUM-Yugoslav Dinar<br>ZMK-Zambian Kwacha<br>ZWD-Zimbabwe Dollar</font></p></blockquote>",
        "policies": null
    },
    "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', parameters('service_ibizmalo_name'))]",
        "[resourceId('Microsoft.ApiManagement/service/apis', parameters('service_ibizmalo_name'), parameters('apis_58a3138ca9f54041296baffd_name'))]"
    ]
},
```

To be compared with the representation recieved when using the REST api:

```
{
  {
    "id": "/subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/apis/58a3138ca9f54041296baffd/operations/58a313e40647c00eecf8d408",
    "type": "Microsoft.ApiManagement/service/apis/operations",
    "name": "58a313e40647c00eecf8d408",
    "properties": {
      "displayName": "Conversion Rate",
      "method": "POST",
      "urlTemplate": "/conversionrate",
      "templateParameters": [],
      "description": "<br><b>Get conversion rate from one currency to another currency <b><br><p><b><font color='#000080' size='1' face='Verdana'><u>Differenct currency Code and Names around the world</u></font></b></p><blockquote><p><font face='Verdana' size='1'>AFA-Afghanistan Afghani<br>ALL-Albanian Lek<br>DZD-Algerian Dinar<br>ARS-Argentine Peso<br>AWG-Aruba Florin<br>AUD-Australian Dollar<br>BSD-Bahamian Dollar<br>BHD-Bahraini Dinar<br>BDT-Bangladesh Taka<br>BBD-Barbados Dollar<br>BZD-Belize Dollar<br>BMD-Bermuda Dollar<br>BTN-Bhutan Ngultrum<br>BOB-Bolivian Boliviano<br>BWP-Botswana Pula<br>BRL-Brazilian Real<br>GBP-British Pound<br>BND-Brunei Dollar<br>BIF-Burundi Franc<br>XOF-CFA Franc (BCEAO)<br>XAF-CFA Franc (BEAC)<br>KHR-Cambodia Riel<br>CAD-Canadian Dollar<br>CVE-Cape Verde Escudo<br>KYD-Cayman Islands Dollar<br>CLP-Chilean Peso<br>CNY-Chinese Yuan<br>COP-Colombian Peso<br>KMF-Comoros Franc<br>CRC-Costa Rica Colon<br>HRK-Croatian Kuna<br>CUP-Cuban Peso<br>CYP-Cyprus Pound<br>CZK-Czech Koruna<br>DKK-Danish Krone<br>DJF-Dijibouti Franc<br>DOP-Dominican Peso<br>XCD-East Caribbean Dollar<br>EGP-Egyptian Pound<br>SVC-El Salvador Colon<br>EEK-Estonian Kroon<br>ETB-Ethiopian Birr<br>EUR-Euro<br>FKP-Falkland Islands Pound<br>GMD-Gambian Dalasi<br>GHC-Ghanian Cedi<br>GIP-Gibraltar Pound<br>XAU-Gold Ounces<br>GTQ-Guatemala Quetzal<br>GNF-Guinea Franc<br>GYD-Guyana Dollar<br>HTG-Haiti Gourde<br>HNL-Honduras Lempira<br>HKD-Hong Kong Dollar<br>HUF-Hungarian Forint<br>ISK-Iceland Krona<br>INR-Indian Rupee<br>IDR-Indonesian Rupiah<br>IQD-Iraqi Dinar<br>ILS-Israeli Shekel<br>JMD-Jamaican Dollar<br>JPY-Japanese Yen<br>JOD-Jordanian Dinar<br>KZT-Kazakhstan Tenge<br>KES-Kenyan Shilling<br>KRW-Korean Won<br>KWD-Kuwaiti Dinar<br>LAK-Lao Kip<br>LVL-Latvian Lat<br>LBP-Lebanese Pound<br>LSL-Lesotho Loti<br>LRD-Liberian Dollar<br>LYD-Libyan Dinar<br>LTL-Lithuanian Lita<br>MOP-Macau Pataca<br>MKD-Macedonian Denar<br>MGF-Malagasy Franc<br>MWK-Malawi Kwacha<br>MYR-Malaysian Ringgit<br>MVR-Maldives Rufiyaa<br>MTL-Maltese Lira<br>MRO-Mauritania Ougulya<br>MUR-Mauritius Rupee<br>MXN-Mexican Peso<br>MDL-Moldovan Leu<br>MNT-Mongolian Tugrik<br>MAD-Moroccan Dirham<br>MZM-Mozambique Metical<br>MMK-Myanmar Kyat<br>NAD-Namibian Dollar<br>NPR-Nepalese Rupee<br>ANG-Neth Antilles Guilder<br>NZD-New Zealand Dollar<br>NIO-Nicaragua Cordoba<br>NGN-Nigerian Naira<br>KPW-North Korean Won<br>NOK-Norwegian Krone<br>OMR-Omani Rial<br>XPF-Pacific Franc<br>PKR-Pakistani Rupee<br>XPD-Palladium Ounces<br>PAB-Panama Balboa<br>PGK-Papua New Guinea Kina<br>PYG-Paraguayan Guarani<br>PEN-Peruvian Nuevo Sol<br>PHP-Philippine Peso<br>XPT-Platinum Ounces<br>PLN-Polish Zloty<br>QAR-Qatar Rial<br>ROL-Romanian Leu<br>RUB-Russian Rouble<br>WST-Samoa Tala<br>STD-Sao Tome Dobra<br>SAR-Saudi Arabian Riyal<br>SCR-Seychelles Rupee<br>SLL-Sierra Leone Leone<br>XAG-Silver Ounces<br>SGD-Singapore Dollar<br>SKK-Slovak Koruna<br>SIT-Slovenian Tolar<br>SBD-Solomon Islands Dollar<br>SOS-Somali Shilling<br>ZAR-South African Rand<br>LKR-Sri Lanka Rupee<br>SHP-St Helena Pound<br>SDD-Sudanese Dinar<br>SRG-Surinam Guilder<br>SZL-Swaziland Lilageni<br>SEK-Swedish Krona<br>TRY-Turkey Lira<br>CHF-Swiss Franc<br>SYP-Syrian Pound<br>TWD-Taiwan Dollar<br>TZS-Tanzanian Shilling<br>THB-Thai Baht<br>TOP-Tonga Pa'anga<br>TTD-Trinidad&amp;amp;Tobago Dollar<br>TND-Tunisian Dinar<br>TRL-Turkish Lira<br>USD-U.S. Dollar<br>AED-UAE Dirham<br>UGX-Ugandan Shilling<br>UAH-Ukraine Hryvnia<br>UYU-Uruguayan New Peso<br>VUV-Vanuatu Vatu<br>VEB-Venezuelan Bolivar<br>VND-Vietnam Dong<br>YER-Yemen Riyal<br>YUM-Yugoslav Dinar<br>ZMK-Zambian Kwacha<br>ZWD-Zimbabwe Dollar</font></p></blockquote>",
      "request": {
        "description": "ConversionRequest",
        "queryParameters": [],
        "headers": [],
        "representations": [
          {
            "contentType": "application/json",
            "sample": "{     \"fromCurrency\": \"SEK\",     \"toCurrency\": \"USD\" }",
            "schemaId": "322649e9-bfd5-4c93-8ad1-3fc1780e8045",
            "typeName": "ConversionRate"
          }
        ]
      },
      "responses": [
        {
          "statusCode": 200,
          "description": "ConversionResult",
          "representations": [
            {
              "contentType": "application/json",
              "sample": "{\r\n    \"rate\": 1.0\r\n}",
              "schemaId": "322649e9-bfd5-4c93-8ad1-3fc1780e8045",
              "typeName": "ConversionRateResponse"
            }
          ],
          "headers": []
        }
      ],
      "policies": null
    }
  }
}
```

We can clearly see that the REST object contains far more inforamtion and since we are working withe environment setups with DEV/TEST/PROD it would be dissapointing to not be able to provide samples in DEV but not in TEST and PRODUCTION (where it makes most sense):

 
So this was the start of my new project **API Management Template Extractor**


While starting this I added a few more requirements
1. Possible to only extract one specific api, but get all required linked attributes such as Named Values.
1. Parameterized URL's (both API and backend)
1. Parameterized Named Values value
1. Automated url/Named values handling for Logic Apps
1. Parameterized values on all changeable values that is needed for groups, products, authorization servers etc

The first release is made today and the project can be found at my GIT site: {URL}

How does it work?

Well it's no magic, I've just combined the result from the REST API's to ARM Templates and along the way added some ARM functionality to improve the development experience.

Let's show with a sample with one of my API's:
I have a "simple" rest API, that has two GET operations, with a policy for api and one for each operation, each operation need's an authentication key that is stored in Named Values.

[![](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-overview.png)](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-overview.png)

The key components that need's to be changed are the URL and the authentication key used in the request to the backend.

**The URL**
[![](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-webserviceurl.png)](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-webserviceurl.png)

**The authenticationkey in the policy ({{trafficauthkey}})**
[![](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-policy.png)](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-policy.png)

**The authenticationkey in Named Values ({{trafficauthkey}})**
[![](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-namedvalues.png)](/assets/uploads/2017/09/apim-arm-template-creator-traffic-api-namedvalues.png)


Even for a rather simple API there are som places that need's to be changed and in order to prevent downtime on your API this is needed to be changed before/during deployment.
So using the PowerShell module I can extract a ARM template that will represent this API.

Let's look at some samples:

### Manually created REST API

Parameterization of URL, the API looks like this and as you can see the URL is parameterized with **[parameters('order_serviceUrl')]**
```
{
      "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/apis/order",
      "type": "Microsoft.ApiManagement/service/apis",
      "name": "[concat(parameters('service_ibizmalo_name'), '/' ,parameters('api_order_name'))]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "Order",
        "apiRevision": "[parameters('order_apiRevision')]",
        "description": "Azure Logic App.",
        "serviceUrl": "[parameters('order_serviceUrl')]",
        "path": "api/v1/order",
        "protocols": [
          "https"
        ],
```

Parameterization of authentication key, the Named Value (Properties it's called in ARM) also has it's value parameterized with **[parameters('trafficauthkey_value')]**
```
{
      "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/properties/trafficauthkey",
      "type": "Microsoft.ApiManagement/service/properties",
      "name": "[concat(parameters('service_ibizmalo_name'), '/' ,parameters('property_trafficauthkey_name'))]",
      "apiVersion": "2017-03-01",
      "properties": {
        "displayName": "trafficauthkey",
        "value": "[parameters('trafficauthkey_value')]",
        "tags": null,
        "secret": true
      },
      "resources": [],
      "dependsOn": []
    }
```

And due to these parameters it's easy to change the values between environments and deliver robust and quality deployments.


### Logic App
When generating the API from a Logic App the needed changes between enviroments are to change the following things: the resource group, the name of the logic app and the trigger name.
The rest will be automated according the standard generation in API Management for Logic Apps.

Description of the standard generation of policy for Logic App instances, first there is a rewrite URL that will change the URL to match the Logic App and also add the sv and sig values (authentication) in a Named Value (Property) in to the URL.
Second step is to set the Base URL of the Call to match the Logic App URL.

Lastly there is a comment section added with the path to the original Azure Resource (our Logic App) used when generating the policy. This is something we are using to automate the URL and authentication values.

[![](/assets/uploads/2017/09/apim-arm-template-creator-logicapp-policy.png)](/assets/uploads/2017/09/apim-arm-template-creator-logicapp-policy.png)

The policy will be updated so that the *policyContent* will use the *listCallbackUrl* ARM function to retrieve the Logic App's base url.
```
 {
  "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/apis/order/operations/59a6b4730d691205f068a8be/policies/policy",
  "type": "Microsoft.ApiManagement/service/apis/operations/policies",
  "name": "[concat(parameters('service_ibizmalo_name'), '/' ,parameters('api_order_name'), '/' ,parameters('operations_59a6b4730d691205f068a8be_name'), '/' ,parameters('policy_policy_name'))]",
  "apiVersion": "2017-03-01",
  "properties": {
    "policyContent": "[Concat('<policies>\r\n  <inbound>\r\n    <rewrite-uri id=\"apim-generated-policy\" template=\"?api-version=2016-06-01&amp;sp=/triggers/request/run&amp;{{orderrequest59a6b4783fb21a7984df42ae}}\" />\r\n    <set-backend-service id=\"apim-generated-policy\" base-url=\"',listCallbackUrl(resourceId(parameters('logicApp_INT001-GetOrderInfo_resourcegroup'),'Microsoft.Logic/workflows/triggers', parameters('logicApp_INT001-GetOrderInfo_name'),parameters('logicApp_INT001-GetOrderInfo_trigger')), providers('Microsoft.Logic', 'workflows').apiVersions[0]).basePath,'\" />\r\n    <base />\r\n    <set-header name=\"Ocp-Apim-Subscription-Key\" exists-action=\"delete\" />\r\n  </inbound>\r\n  <outbound>\r\n    <base />\r\n  </outbound>\r\n  <backend>\r\n    <base />\r\n    <!-- { \"azureResource\": { \"type\": \"logicapp\", \"id\": \"/subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.Logic/workflows/INT001-GetOrderInfo/triggers/request\" } } -->\r\n  </backend>\r\n</policies>')]"
  },
  "resources": [],
  "dependsOn": [
    "[resourceId('Microsoft.ApiManagement/service/apis', parameters('service_ibizmalo_name') ,parameters('api_order_name'))]",
    "[resourceId('Microsoft.ApiManagement/service/apis/operations', parameters('service_ibizmalo_name'), parameters('api_order_name'), parameters('operations_59a6b4730d691205f068a8be_name'))]"
  ]
}
```

The Named Value (Property) will be automated the same way, using the *listCallbackUrl* ARM function.
```
{
  "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/properties/59a6b478285385ab3dfbb752",
  "type": "Microsoft.ApiManagement/service/properties",
  "name": "[concat(parameters('service_ibizmalo_name'), '/' ,parameters('property_59a6b478285385ab3dfbb752_name'))]",
  "apiVersion": "2017-03-01",
  "properties": {
    "displayName": "orderrequest59a6b4783fb21a7984df42ae",
    "value": "[concat('sv=',listCallbackUrl(resourceId(parameters('logicApp_INT001-GetOrderInfo_resourcegroup'),'Microsoft.Logic/workflows/triggers', parameters('logicApp_INT001-GetOrderInfo_name'),parameters('logicApp_INT001-GetOrderInfo_trigger')), providers('Microsoft.Logic', 'workflows').apiVersions[0]).queries.sv,'&sig=',listCallbackUrl(resourceId(parameters('logicApp_INT001-GetOrderInfo_resourcegroup'),'Microsoft.Logic/workflows/triggers', parameters('logicApp_INT001-GetOrderInfo_name'),parameters('logicApp_INT001-GetOrderInfo_trigger')), providers('Microsoft.Logic', 'workflows').apiVersions[0]).queries.sig)]",
    "tags": [],
    "secret": true
  },
  "resources": [],
  "dependsOn": []
}
```

By using the **listCallbackUrl** function we can retrieve the values from the specific Logic App at deployment time, removing all manual handling of URL's and authentication values when working with API's exposing Logic Apps. 

### SOAP
So when SOAP was introduced to API Management a new resource was added, **Backend** this is unfortunally not editable in the GUI yet but we can do changes on with ARM deployments, the REST and PS API's.
So in order to create a full ARM template for SOAP we need that extra resource and all other allready described information.

In my sample API Manegement instance my SOAP Currency Converter API is generated with a backend that has a parameter *[parameters('f9211da9-8ce1-48c7-8e13-e38bc5dbcbcd_url')]* to be used to change the URL.
```
{
  "comments": "Generated for resource /subscriptions/c107df29-a4af-4bc9-a733-f88f0eaa4296/resourceGroups/PreDemoTest/providers/Microsoft.ApiManagement/service/ibizmalo/backends/f9211da9-8ce1-48c7-8e13-e38bc5dbcbcd",
  "type": "Microsoft.ApiManagement/service/backends",
  "name": "[concat(parameters('service_ibizmalo_name'), '/' ,parameters('backend_f9211da9-8ce1-48c7-8e13-e38bc5dbcbcd_name'))]",
  "apiVersion": "2017-03-01",
  "properties": {
    "title": "CurrencyConverter Soap Backend",
    "description": "CurrencyConverter Soap Backend",
    "url": "[parameters('f9211da9-8ce1-48c7-8e13-e38bc5dbcbcd_url')]",
    "protocol": "soap",
    "properties": {},
    "tls": {}
  },
  "resources": []
},
```

### Get Started
So how do you get started? First of download the repo, compile it and use this PowerShell script (replace the values with appropiate ones from your subscription and API Management instance).
``` PowerShell
#if you have problem with execution policy execute this in a administrator runned powershell window.
#Set-ExecutionPolicy -ExecutionPolicy Unrestricted

Import-Module "C:\temp\APIManagement\APIManagementTemplate\bin\Debug\APIManagementTemplate.dll"


#Set the name of the API Mangement instance
$apimanagementname = 'ibizmalo'

#Set the resource group 
$resourcegroupname = 'PreDemoTest' # 'app-tst-cs-int-Migrated'
#Set the subscription id 
$subscriptionid = 'c107df29-a4af-4bc9-a733-f88f0eaa4296'#'d4baa1e9-15f5-4c85-bb3e-1e108dc79b00'#
#Set the tenant to use when login ing, make sure it has the right tennant
$tenant = 'mattiaslogdbergibizsolution.onmicrosoft.com'

#optional set filter for a specific api (using standard REST filter, with path we can select api based on the API path)
#$filter = "path eq 'api/v1/order'"
#$filter = "path eq 'api/v1/currencyconverter'"

 
#setting the output filename
$filenname = $apimanagementname + '.json'

Get-APIManagementTemplate -APIFilters $filter -APIManagement $apimanagementname -ResourceGroup $resourcegroupname -SubscriptionId $subscriptionid -TenantName $tenant -ExportPIManagementInstance $false  | Out-File $filenname

```

### Upcoming
So is this all done? No there are more work to come and here are some of my Todo's in this project:

* Automated support for all Azure Resources.
* Operations Schema support.
* Retrieve authroization servers linked to API, and parameterize them as needed.
* Better support for authorization servers.
* Support for Products and linking API's to products

