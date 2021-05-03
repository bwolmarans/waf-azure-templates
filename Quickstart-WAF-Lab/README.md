# Barracuda WAF for Azure - Quickstart Lab with 1 x WAF and 1 x Host running multiple targets

## Introduction
This Azure Resource Manager template will create a new Virtual Network and deploy a single Barracuda Web Application Firewall and a single Host running known vulnerable website on various ports.

## Prerequisites
The solution does a check of the template when you use the provided scripts. It does require that [Programmatic Deployment](https://azure.microsoft.com/en-us/blog/working-with-marketplace-images-on-azure-resource-manager/) is enabled for the Barracuda Web Application Firewall BYOL or PAYG images. Barracuda recommends use of **D**, **D_v2**, **F** or newer series. 

You can enable programatic deployment via Powershell using the Cloud Shell feature in the portal. Below are two powershell examples for byol and hourly, please adapt as required to your version of powershell and byol or hourly license requirement.

`Get-AzMarketplaceTerms -Publisher "barracudanetworks" -Product "waf" -Name "byol" | Set-AzMarketplaceTerms -Accept`

`Get-AzureRmMarketplaceTerms -Publisher "barracudanetworks" -Product "waf" -Name "hourly" | Set-AzureRmMarketplaceTerms -Accept`

`az vm image terms accept --urn barracudanetworks:waf:byol:*`

## Deployment

The package provides a deploy.ps1 and deploy.sh for Powershell or Azure CLI based deployments. This can be peformed from the Azure Portal as well as the any system that has either of these scripting infrastructures installed. Or you can deploy from the Azure Portal using the provided link.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fbarracudanetworks%2Fwaf-azure-templates%2Fmaster%2FQuickstart-WAF-Lab%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fbarracudanetworks%2Fwaf-azure-templates%2Fmaster%2FQuickstart-WAF-Lab%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

## Deployed resources
Following resources will be created by the template:
- One Azure VNET with 2 subnets (1 for the WAF and a second for the Host )
- One Barracuda WAF
- One Host (Ubuntu) with docker images deployed for various vulnerable target applications.


## Next Steps
After succesfull deployment you should be able to access the WAF via it's management GUI http://<WAF Public IP or DNS Name>:8001 if you selected PAYG then you can immediately use the Secure https://<WAF Public IP or DNS Name>:8443

## Post Deployment Configuration
Once provisioned you will be able to access via the DNS name or Public IP of the WAF and the applications on the Host.
These testing applications are courtesy of the OWASP Vulerability Project, among others (see credits below) and allow you to test the WAF functionality. 

The Host will present on the private IP of: 172.16.137.4

You will need to configure the WAF services to deliver the sites which are arranged on the Host as follows:


| Application | Port | Example Path| More Info | 
|:------|:--------:|:--------|:--------|
|DVWA| 1000 | http://`<Host Public IP>`:1000/login.php | http://www.dvwa.co.uk/ |
|Mutillidae | 2000 | http://`<Host Public IP>`:2000/ |http://www.irongeek.com/i.php?page=mutillidae/mutillidae-deliberately-vulnerable-php-owasp-top-10 |
|WebGoat | 3000 | http://`<Host Public IP>`:1000/ |https://owasp.org/www-project-webgoat/|
|BWAPP | 4000 | http://`<Host Public IP>`:4000/ | |
|JuiceShop | 5000 | http://`<Host Public IP>`:5000/ | https://owasp.org/www-project-juice-shop/|
|Altoro | 6001 | http://`<Host Public IP>`:6001/ ||	
|HTTPBin | 7000 | not currently loading || 	
|Hackazon | 8000 | http://`<Host Public IP>`:8000/ ||
|Petstore | 9000| http://`<Host Public IP>`:9000/ ||

### Step 1
Configure the WAF services to deliver the sites.

There are a number of ways to configure the WAF's in Azure to deliver these services. 
- You could create a service per application on a different port
	![Network diagram](images/image_waf_differentports.png)
- You could create a single service and split the applications by hostname using content rules to direct traffic to different ports on the web server
	![Network diagram](images/image_waf_contentrules.png)
- You could enable Azure Multi-IP and have each service running on different IP's but the same port


To assist in accessing you can add the following line to your hosts file: 
`
yourwafpublicip  dvwa.local mutillidae.local webgoat.local bwapp.local juiceshop.local altoro.local httpbin.local hackazon.local petstore.local
`

### Step 2 - Petstore
The Petstore site requires some additional setup in the WAF, to allow it to deliver it's content a Website Translation rule is required. The below is shown using the DNS name for the hostsfile but this could also be <publicip>:9000
![Network diagram](images/image_petstore_translation.png)

