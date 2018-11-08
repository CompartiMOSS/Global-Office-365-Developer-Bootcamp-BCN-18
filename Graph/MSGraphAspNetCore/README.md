# Graph de principio a fin

----------
## Overview
Este ejemplo de ASP.NET Core 2.1 MVC muestra cómo conectarse a Microsoft Graph utilizando los permisos de los delegados y el endpoint de Azure AD v2.0 (MSAL) para recuperar el perfil y la imagen de perfil de un usuario y enviar un correo electrónico que contenga la foto como archivo adjunto junto a un documento convertido a PDF.

## Objectives

Que aprenderemos:

- Conectar una aplicación ASP.NET Core con Graph
- Interactuar con Microsoft Graph con Microsoft Graph Client Library for .NET

## Prerequisites
Que es necesario:

- Un tenant de office365. Podéis crearos una cuenta gratuita aquí: [https://products.office.com/es-es/try](https://products.office.com/es-es/try "Office trial")

- La última versión de Visual Studio 2017 con [.NET Core 2.1 SDK](https://www.microsoft.com/net/download). Podeis descargaros la versión community de aquí:
[https://visualstudio.microsoft.com/es/vs/community/](https://visualstudio.microsoft.com/es/vs/community/ "VS2017")
## Exercises

Este hands on lab contiene los siguientes ejercicios.

 
- [1-Creación del proyecto](./01_CreateProject.md)

- [2-Configurar la conexión a Graph](./02_ConfigureGraph.md)

- [3-Obtener los datos del usuario](./03_RetrieveUserProfile.md)

- [4-Convertir un documento a PDF y enviar un mail](./04_UploadOneDriveConvertPDF.md)

