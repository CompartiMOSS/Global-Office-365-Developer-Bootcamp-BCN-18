# Azure para Office365 Developers

----------
## Overview
¿Como podemos integrar nuestro office365 con Azure? Hay dos formas básicas:

- WebHooks
- Servicios que se conectan a office365 y realizan operaciones.

Dentro de este último apartado nos encontramos con las LogicApps que nos permiten conectarnos a los diferentes productos de Office 365 entre otros y reaccionar a cambios ejecutando un flujo de trabjao.
También podemos integrar nuestras aplicaciones mediante AzureAD y hacer SSO.

## Objectives

Que aprenderemos:

- Realizar un SSO entre Office365 y una webapp.
- Crear la infraestructura necesaria de Azure
- Creación de una LogicApp y creación de un flujo.
- Primera aproximación a CosmosDB

## Prerequisites
Que es necesario:

- Un tenant de Office 365. Podéis crearos una cuenta gratuita de developer aquí: [https://developer.microsoft.com/es-es/office/dev-program](https://developer.microsoft.com/es-es/office/dev-program "Office 365 Devs")

- Una subscripción de Azure. Podéis crearos una cuenta gratuita aquí: [https://azure.microsoft.com/es-es/offers/ms-azr-0044p/](https://azure.microsoft.com/es-es/offers/ms-azr-0044p/ "Azure Tiral")
- La última versión de Visual Studio 2017 (instalar cualquier actualización pendiente - puede tardar más de media hora). Podeis descargaros la versión community de aquí:
[https://visualstudio.microsoft.com/es/vs/community/](https://visualstudio.microsoft.com/es/vs/community/ "VS2017")
- El modulo de AzureRM para Powershell instalado y actualizado
```	
	Install-Module -Name AzureRM -AllowClobber
	
	Update-Module -Name AzureRM
```

## Exercises

Este hands on lab contiene los siguientes ejercicios.

- [1-AzureADSSO](./AzureParaOffice365Developers/1_AzureAD_SSO/readme.md)  
 
- [2-Creación infraestructura Azure](./AzureParaOffice365Developers/2_Creación_infraestructura_Azure/readme.md)

- [3-Creación entorno Sharepoint](./AzureParaOffice365Developers/3_Creación_entorno_Sharepoint/readme.md)

- [4-Creación Lógic App](./AzureParaOffice365Developers/4_Creación_Logic_App/readme.md)

