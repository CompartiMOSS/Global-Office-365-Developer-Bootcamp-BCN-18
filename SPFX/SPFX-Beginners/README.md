# SPFx for Beginners

En este repositorio podrás encontrar el material utilizado en el Global Office Developer Bootcamp 2018 en Barcelona, durante el cual aprenderemos a desarrollar con Sharepoint Framework (SPFx).


## Ponentes

* **Ángel Rubén Yui** - [Twitter](https://twitter.com/angelrubenyui)

* **Ferran Chopo** - [Twitter](https://twitter.com/fchopo)

## Laboratorio

El objetivo del laboratorio es el desarrollo de un ChatBot que pueda incluirse en cualquier página de Sharepoint. Para ello crearemos una extensión SPFx que se conectará a un Bot implementado con la herramienta [QnA Maker](https://www.qnamaker.ai/). 

![ChatBotExtension](https://github.com/CompartiMOSS/Global-Office-365-Developer-Bootcamp-BCN-18/blob/master/SPFX/SPFX-Beginners/assets/01_ChatBotExtension.png)

### Prerequisitos

Antes de hacer el laboratorio, es necesario realizar los pasos que se indican a continuación:

1. [Unirse al programa de desarrollo de Office 365](https://docs.microsoft.com/es-es/office/developer-program/office-365-developer-program).

2. [Crear y configurar un tenant de desarrollo de Office 365](https://docs.microsoft.com/es-es/office/developer-program/office-365-developer-program-get-started).

3. Instalar [Office365Cli](https://pnp.github.io/office365-cli/), herramienta mediante la cual podremos administrar un tenant de Office 365 mediante línea de comandos desde cualquier plataforma (Windows, Linux, Mac). 

4. Crear un sitio tipo *Catálogo de aplicaciones* con Office365Cli.

```
o365
spo login https://<nombre_del_tenant>-admin.sharepoint.com
spo site appcatalog add --url https://<nombe_del_tenant>.sharepoint.com/sites/AppCatalog
```

5. [Habilitar el CDN para el tenant de Office 365](https://docs.microsoft.com/es-es/office365/enterprise/use-office-365-cdn-with-spo), ya que será necesario para poder hacer el deploy de nuestros desarrollos.

```
spo cdn set –-type Public –-enabled true
```

6. Crear un *Sitio de Equipo* para probar nuestros desarrollos (en este caso le llamamos *DevSite*, por ejemplo).

```
spo site add –type TeamSite -t DevSite –url https://<nombre_del_tenant>.sharepoint.com/sites/DevSite
```

### Desarrollo de una SPFx Extension

#### Crear un Bot con QnA Maker

Podéis crear un Bot desde la web de [QnAMaker](https://www.qnamaker.ai/), en la que se podrán crear y gestionar nuestras *Knowledge Base*, que no son más que conjuntos de datos con preguntas y respuestas. Los pasos a realizar son:

1. En una suscripción de Azure crear un *QnA Maker*.

![QnAMakerService](https://github.com/CompartiMOSS/Global-Office-365-Developer-Bootcamp-BCN-18/blob/master/SPFX/SPFX-Beginners/assets/02_AzureQnaMaker.png)


2. [Crear una Knowledge Base](https://www.qnamaker.ai/Create), donde deberemos elegir el servicio de Azure creado previamente, indicar el nombre que queremos darle a esta base de datos, así como el fichero o las URLs mediante las cuáles construir el Bot.

![KnowledgeBase](https://github.com/CompartiMOSS/Global-Office-365-Developer-Bootcamp-BCN-18/blob/master/SPFX/SPFX-Beginners/assets/03_ConnectQnaService.png)


3. Una vez tenemos la *Knowledge Base* podemos ver qué preguntas y respuestas contiene, y gestionar las entradas detectadas. Si realizamos cualquier modificación deberemos guardarla y entrenar al bot (save and train), para finalmente publicarla.

![DeploymentDetails](https://github.com/CompartiMOSS/Global-Office-365-Developer-Bootcamp-BCN-18/blob/master/SPFX/SPFX-Beginners/assets/04_ShareKB.png)


En la sección de *Deployment details* podremos ver ejemplos de como hacer la llamada al servicio, y que usaremos en el desarrollo de nuestra extensión SPFx (necesitaremos el id del QnA Maker creado en Azure, así como el EndpointKey).


#### Crear una SPFx Extension

En este punto crearemos nuestro proyecto SPFx con los pasos que se detallan seguidamente:

1. Crear una carpeta donde ubicar nuestro proyecto.

2. Desde la carpeta creada en el paso anterior, crearemos la estructura básica del proyecto con [Yeoman](http://yeoman.io/):

```
yo @microsoft/sharepoint
```

Seguidamente deberemos indicar algunos datos para crear el proyecto:

* Solution name: *qna-chat*
* Platform: *Sharepoint Online only*
* File placement: *Use the current folder*
* Allow admin the choice of being able to deploy the solution to all sites: *No*
* Component type: *Extension*
* Extension type: *Application Customizer*
* Application Customizer name: *qnAChat*

A partir de este momento se crea la estructura del proyecto y se descargan los ficheros necesarios para que podamos desarrollar y ejecutar nuestro proyecto.

3. Añadir el paquete react-chat-widget a nuestro proyecto.

```
npm install –react-chat-widget
```

4. Ejecutar Visual Studio Code para poder empezar a desarrollar.

```
code .
```

5. Una vez que se abre nuestro proyecto:

* Crear la carpeta *services*, donde ubicaremos los ficheros con la lógica para hacer la llamada a la API de QnA Maker.

* Crear el fichero *QnaServices.ts* en la carpeta creada anteriormente.

![Carpetas](https://github.com/CompartiMOSS/Global-Office-365-Developer-Bootcamp-BCN-18/blob/master/SPFX/SPFX-Beginners/assets/05_Carpetas.png)


6. Editaremos el fichero *QnAServices.ts* que contendrá la lógica para hacer la llamada a la API del QnA Maker.

* Copia y pega el siguiente código al principio del fichero

```
import { HttpClient, IHttpClientOptions, HttpClientConfiguration } from "@microsoft/sp-http";
import { ApplicationCustomizerContext } from "@microsoft/sp-application-base";
```

SharePoint Framework incluye una clase auxiliar [spHttpClient](https://docs.microsoft.com/es-es/javascript/api/sp-http/sphttpclient?view=sp-typescript-latest) para ejecutar solicitudes de la API de REST con SharePoint. Agrega encabezados predeterminados, administra la síntesis necesaria para escrituras y recopila telemetría que ayuda al servicio a supervisar el rendimiento de una aplicación. [ApplicationCustomizerContext](https://docs.microsoft.com/es-es/javascript/api/sp-application-base/applicationcustomizercontext?view=sp-typescript-latest) proporciona acceso al contexto del componente. 

* Crearemos una interface para poder recibir el contexto de ejecución del *Application Customizer*.

```
export interface QnAServiceConfiguration {
    context: ApplicationCustomizerContext;
}
```

* Crearemos la clase que implementa la llamada al servicio a partir de un texto que se recibe por parámetro en la función *getQnaAnswer*.

```
export class QnAService {
    private context: ApplicationCustomizerContext;
    private knowledgebaseId: string = "{Token-Acceso-QnAMaker}";

    constructor(config: QnAServiceConfiguration) {
        this.context = config.context;
    }

    public async getQnaAnswer(userQuery: string): Promise<String> {
        let answer: string = 'Lo siento... ¡No he podido encontrar una respuesta a tu pregunta!';
        // Build URI
        const postURL=`https://{urlsite}.azurewebsites.net/qnamaker/knowledgebases/${this.knowledgebaseId}/generateAnswer`

        // Build body
        const body: string = JSON.stringify({
            'question': userQuery
        });

        // Build headers
        const requestHeaders: Headers = new Headers();
        requestHeaders.append('Content-type', 'application/json');
        requestHeaders.append('Authorization','EndpointKey {TokenID}');

        const httpClientOptions: IHttpClientOptions = {
            body: body,
            headers: requestHeaders
        };

        let response = await this.context.httpClient.post(
            postURL,
            HttpClient.configurations.v1,
            httpClientOptions
        );

        if (response.ok) {
            let json = await response.json();
            if (json.answers[0].answer != 'No good match found in the KB')
                answer = json.answers[0].answer;
        }
        return answer;
    }
}
```

Cabe indicar que debéris indicar el *KnowledgeBaseId* asignado cuando creasteis vuestra base de datos de preguntas y respuestas, así como el *EndPointKey* que para poder acceder a ella.

8. Dentro la carpeta *qnAChat/components* crearemos los siguientes ficheros que a continuación implementaremos.

9. Editaremos el fichero *FooterChat.module.scss* que contendrá los estilos del chat.
```
.FooterChat{

    .FooterChat>div{
        margin: 0 20px 60px 0 !important;
    }
}
```

10. Editaremos el fichero *IFooterChatProps.ts* que contiene las propiedades del componente FooterChat. En este caso, el componente solo necesita hacer referencia a la clase QnAService, sin que sea necesario añadir cualquier propiedad más.

```
import { QnAService } from "../../../services/QnAServices";

export interface IFooterChatProps
{
    qnaService: QnAService;
}
```

11. A continuación editaremos el fichero FooterChat.ts, que es el componente React.

* Inicialmente introduciremos todos los imports de las librerías que necesitamos:

```
import * as React from 'react';
import * as strings from 'QnAChatApplicationCustomizerStrings';
import styles from './FooterChat.module.scss';
import { IFooterChatProps } from './IFooterChatProps';
import { Widget, addResponseMessage, renderCustomComponent, addUserMessage } from 'react-chat-widget';
import 'react-chat-widget/lib/styles.css';
```

* Implementamos la clase del componente, donde indicamos que implementa la Interface creada en el paso anterior:

```
export default class FooterChat extends React.Component<IFooterChatProps, {}> {
    constructor(props: IFooterChatProps,{}) {
        super(props);

        this.state = {
            items: []
        };
    }

    private _handleNewUserMessage = (newMessage) => {
        this.props.qnaService.getQnaAnswer(newMessage).then((answer) => {
            addResponseMessage(answer);
        });
    }

    public render() {
        return (
            <div className={styles.FooterChat}>
                <Widget
                    handleNewUserMessage={this._handleNewUserMessage}
                    title={strings.ChatTitle}
                    subtitle={strings.ChatSubtitle}
                />
            </div>
        );
    }
}
```

* En el método *Render* indicamos que queremos mostrar un componente Widget, y que cada vez que un usuario introduzca un mensaje, se llame a la función *handleNewUserMessage*.

* El método *handleNewUserMessage* llama a la función *QnaAnswer* del servicio *QnAService*, que nos devolverá una respuesta en función del texto introducido.

12. Finalmente, editaremos el fichero *QnAChatApplicationCustomizer.ts*, que contiene la lógica del *Application Customizer* y que se ejecutará nada más referenciarlo.

```
import { override } from '@microsoft/decorators';
import { Log } from '@microsoft/sp-core-library';
import {
  BaseApplicationCustomizer,
  PlaceholderContent,
  PlaceholderName
} from '@microsoft/sp-application-base';
import { Dialog } from '@microsoft/sp-dialog';

import * as React from 'react';
import * as ReactDom from 'react-dom';

import * as strings from 'QnAChatApplicationCustomizerStrings';
import { QnAService } from '../../services/QnAServices';
import { IFooterChatProps } from './components/IFooterChatProps';
import FooterChat from './components/FooterChat';

const LOG_SOURCE: string = 'QnAChatApplicationCustomizer';

export interface IQnAChatApplicationCustomizerProperties {
  testMessage: string;
}

/** A Custom Action which can be run during execution of a Client Side Application */
export default class QnAChatApplicationCustomizer
  extends BaseApplicationCustomizer<IQnAChatApplicationCustomizerProperties> {
    private _bottomPlaceholder: PlaceholderContent | undefined;

  @override
  public onInit(): Promise<void> {
    this._renderFooter();
    return Promise.resolve();
  }

  private _renderFooter(): void {
    // Instantiate cognitive service
    const service = new QnAService({
      context: this.context,
    });

    // Handling the bottom placeholder
    if (!this._bottomPlaceholder) {
      this._bottomPlaceholder =
        this.context.placeholderProvider.tryCreateContent(
          PlaceholderName.Bottom,
          { onDispose: this._onDispose });

      // The extension should not assume that the expected placeholder is available.
      if (!this._bottomPlaceholder) {
        console.error('The expected placeholder (Bottom) was not found.');
        return;
      }

      const element: React.ReactElement<IFooterChatProps> = React.createElement(
        FooterChat,
        {
          qnaService: service
        });

      ReactDom.render(element, this._bottomPlaceholder.domElement);
    }
  }
  private _onDispose(): void {
    console.log('Disposed custom bottom placeholders.');
  }
}
```

Básicamente lo que se hace es localizar la parte inferior de la pantalla (actualmente solo se puden añadir Application Customizer en la parte superior e inferior), y allí renderiza un componente de React llamado FooterChat, que hemos creado en los pasos anteriores.

## Pruebas

Antes de poder probar el componente modificaremos el fichero *config/serve.json*, donde podemos indicar a que página queremos ir cuando probemos el componente. Unicamente tenemos que modificar la propiedad *pageUrl* de la sección *default*. Debería quedar algo como lo siguiente

```
"default": {
      "pageUrl": "https://nombe_del_tenant.sharepoint.com/sites/nombre_del_site/SitePages/Home.aspx",
      "customActions": {
        "5c15e79f-b964-4556-b744-ab6af8a002ec": {
          "location": "ClientSideExtension.ApplicationCustomizer",
          "properties": {
            "testMessage": "Test message"
          }
        }
      }
    },
```

## Instalación

Para poder instalar el componente utilizaremos Office365Cli mediante los pasos siguientes:

1. Conectarse al tenant de Office365 y añadir la aplicación al catálogo:

```
spo login https://tutenant-admin.sharepoint.com
spo app add -p C:\TuCarpetaProyecto\sharepoint\solution\qna-chat.sppkg
```

2. Al añadir nuestro aplicación al catálogo, recibiremos un *id*, que utilizaremos para hacer el *deploy* y posteriormente instalarla en la colección de sitios que queramos:

```
spo app deploy --id id_de_aplicacion
spo app install -i id_de_aplicacion -s https://tutenant.sharepoint.com/sites/DevSite
```

Si todos los pasos se han ejecutado correctamente, deberíamos ver nuestro ChatBot en el sitio de Sharepoint indicado.




