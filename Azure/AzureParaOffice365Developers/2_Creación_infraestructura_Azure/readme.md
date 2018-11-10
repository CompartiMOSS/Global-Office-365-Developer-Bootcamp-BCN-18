# Creación Infraestructura

----------


1 - Abrir el programa de Windows PowerShell ISE como administrador.  Acordaros de tener el modulo de AzureRM instalado y actualizado con estos comandos:
```	
	Install-Module -Name AzureRM -AllowClobber
	
	Update-Module -Name AzureRM
```

2 - Buscar y abrir el archivo [Azure\Azure Template\deploy.ps1](../../AzureTemplate/deploy.ps1)  


3 -  Ejecutar el script y seguir las instrucciones


> Si os dá el siguiente error:

> ![alt text](../media/Infraestructure/error.png)


> Ejecutar el siguiente comando:


> 	Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass


4 - Una vez creado iremos al portal y crearemos una collection en CosmosDB

![alt text](../media/Infraestructure/cosmoscollection.png)

Ir al siguiente ejecicio: [3-Creación entorno Sharepoint](../3_Creación_entorno_Sharepoint/readme.md)




