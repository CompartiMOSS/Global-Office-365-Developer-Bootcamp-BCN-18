# Creación del proyecto de Visual Studio

1. Creamos una aplicación ASP.NET Core en Visual Studio 2017

![create-app](./Assets/create-app.jpg)

Asegurate de tener seleccionado **ASP.NET Core 2.1** y **Aplicación Web (controlador de vista de modelos)** y haz click en **Aceptar**

![framework-app](./Assets/framework-app.jpg)

2. Una vez creada, nos aparecerá una pantalla por defecto:

- Seleccionar **Servicios conectados**
- Seleccionar **Authenticación con Azure Active Directory** 
  
![Connected-Services](./Assets/Connected-Services.png)

3. Nos mostrará una pantalla para configurar la autenticación con Azure Active Directory. Seleccionamos **Este sitio web debe ofrecer un inicio de sesión interactivo para los exploradores** y pulsamos **Siguiente**

![Configure-AAD](./Assets/Configure-AAD.jpg)

4. Añadimos el nombre de nuestro dominio e indicamos que vamos a utilizar la aplicación que hemos registrado anteriormente indicando el Id. de cliente y la url de redirección

Opcional: en este paso podemos dejar que Visual Studio registre la aplicación en Azure.

![Configure-AAD-2](./Assets/Configure-AAd-2.jpg)

5. Pulsamos sobre **Finalizar**

Ir a [2-Configurar la conexión a Graph](./02_ConfigureGraph.md)