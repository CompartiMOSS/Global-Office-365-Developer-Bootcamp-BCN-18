##SPFX for ninjas
# Global Office 365 Developer Bootcamp Barcelona 2018 
En este repositorio podrás encontrar los materiales utilizados en el Workshop **SPFx for Ningas** Global Office 365 Developer Bootcamp Barcelona 2018 .

## Ponentes
1. Adrián Díaz - MS Office Development MVP [@AdrianDiaz81](https://twitter.com/AdrianDiaz81)
2. Juan Carlos Martinez - Software Architect at Encamina [@jcmartinezg23](https://twitter.com/jcmartinezg23)

# SPFx for Ninjas

## Pre-requisitos

Para completar el workshop, previamente se debe configurar tanto la tenant de Office 365, como el entorno local. Para ello se recomienda seguir los siguientes artículos:

1. [Setup your Office 365 Tenant](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-developer-tenant)
2. [Set up your SharePoint Framework development environment](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment)

# Elección del FrameWork
Desde el equipo de SharePoint se ha hecho mucho hincapie que SPFx es agnostico al framework/libreria que queramos elegir durante nuestro desarrollo. Dicho esto esta claro que hay un Framework que es el que más se utiliza dentro de los desarrollos, ese no es otro que ReactJS.  En este Workshop vamos a ver como implementar el patrón **FLUX** y ver como poder solucionar temas de perfomance dentro de nuestros desarllos

## Porqué ReactJS y porque en SharePoint

Si bien es cierto que hay multitud de frameworks/librerias JS y la elección de una o otra se puede hacer dependiendo de las skills que tiene tu equipo de desarrollo. Sinos ceñimos a lo que son desarrollos de SharePoint y su integración dentro de SharePoint Online, si tenemos que elegir una frameworks/libreria buscariamos una libreria ligera que solamente se encargue de la vista. De forma que lo que tenga que hacer lo haga muy bien y sea su función. Esto encaja con el principio sobre el cual debemos de extender SharePoint (ya sea online o onpremise). 
Ahora bien pueden haber supuestos en los que es necesario que nuestro desarrollo sea un poco más complejo y sea necesario añadirle más "complementos" a nuestro desarrollo. Es decir necesitaremos enrutado, acciones, .... Con todo esto ReactJS tiene una serie de librerias que si pueden incorporar react-redux,redux-thunk, etc...  
Todo esto esta muy bien, que es lo novedoso que trae ReactJS y su implementación el patrón FLUX

## Qué es el patrón FLUX 
Flux es una patrón/arquitectura  para el manejo y el flujo de los datos en una aplicación web. Fue ideada por el equipo de Facebook siendo su funcionalidad principal facilitar el manejo de datos en aplicaciones web con cierto grado de complejidad.

Estamos acostumbrados a las arquitecturas MVC en la que hay un flujo de datos bidireccional, es decir cualquier modificación en el servidor se modifica en la vista y viceversa, esto hace que en flujos complejos los problemas de rendimiento están a la orden del día. Con Flux esto cambia, propone una arquitectura en la que el flujo de datos es unidireccional. Los datos viajan desde la vista por medio de acciones y llegan a un Store desde el cual se actualizará la vista de nuevo.

![patronflux](./assets/flux.png)

Teniendo todo el flujo de la aplicación centralizado es mucho más sencillo depurar las aplicaciones y encontrar los errores en la misma.

Que actores entran en juego en una arquitectura Flux:

·       **Vista**: Serían los propios componentes de React.

·       **Store**: Guarda los datos de la aplicación. No hay métodos en la store que permitan modificar directamente sobre ella, se tiene que hacer a través de dispatcher y acciones.

·       **Actions o Acciones**: Una acción es simplemente un objeto que indica una intención de realizar algo y que lleva datos asociados en caso de ser necesario.

·       **Dispatcher**:  No es más que un mediador entre la Store y las acciones. Sirve para desacoplar la Store de la vista, ya que así no es necesario conocer que Store maneja una acción concreta.

El flujo que sigue la aplicación sería el siguiente:

·       La vista, mediante un evento envía una acción con la intención de realizar un cambio en el estado.

·       La acción contiene el tipo y los datos, y es enviada al dispatcher.

·       El dispatcher propaga la acción al Store y se procesa en orden de llegada.

·       El Store recibe la acción y dependiendo del tipo recibido, actualiza el estado y notifica a las vistas de ese cambio.

·       La vista recibe la notificación y se actualiza con los cambios.

Este patrón se puede implementar bien de forma propia o bien utilizando alguna librería como pueda ser Redux, ReFlux, Fluxxor, Fluxible, etc…  De todas ellas la más utilizada es Redux, es una pequeña librería de menos de 2kb y que con unos pocos métodos implementa el patrón Flux. Es agnóstica al framework por lo que esta se puede implementar en otros frameworks como Angular, Vue, etc.

## ¿Qué hace Redux?

Se encarga en cierta manera de desacoplar el estado global de una aplicación web de la parte visual. El estado de la aplicación pueden ser varias cosas, normalmente se trata los datos que se reciben a través de peticiones a servicios REST (consultas a listas de SharePoint). Pero también se refiere al estado de la UI en un determinado momento, por ejemplo: mostrar una información al usuario o no, un mensaje de error, ocultar desplegar un panel, etc.

Los conceptos claves de Redux:

**1.- La Store**=> La única fuente de datos, aunque el patrón Flux indica que pueden haber más de una store, Redux simplifica unificando todo en un único árbol.

**2.- El Estado**=> Solo podemos modificar el estado a través de acciones

**3.- Reducers**=> Es básicamente una función que recibe dos parámetros, el estado inicial y una acción y dependiendo del tipo de acción realizará una operación u otra en el estado.

## Show me the Code Talk is Cheap !! 

Para empezar con el workshop, tenemos que crear nuestra solución spfx. Para ello vamos a crear un proyecto webpart haciendo uso del generador de Yeoman para spfx. Cuando nos pida la elección del FrameWork seleccionaremos naturalmente ReactJS.

![yo](./assets/yo.PNG)

**Nota :** En este workshop solamente nos centramos en la parte del Framework no en la forma en la que se va a desplegar en el CDN por lo que los parámetros que hay puestos para ello NO tienen porque ser los recomendados.

Una vez creado agregaremos los siguientes paquetes de npm 

```js
npm install --save-dev react-redux@5.0.6  redux@3.7.2 redux-thunk@2.2.0 react-router-dom@4.3.1 react-hot-loader@4.3.12 
npm install --save-dev @types/redux @3.6.0
npm install --save-dev @bootstrap
```

Una vez tenemos las dependedencias tendremos que tener un package.json similar el siguiente: 
```js
{
  "name": "global-office-365-developer-bootcamp-bcn-18",
  "version": "0.0.1",
  "private": true,
  "engines": {
    "node": ">=0.10.0"
  },
  "scripts": {
    "build": "gulp bundle",
    "clean": "gulp clean",
    "test": "gulp test"
  },
  "dependencies": {
    "react": "15.6.2",
    "react-dom": "15.6.2",
    "@types/react": "15.6.6",
    "@types/react-dom": "15.5.6",
    "@microsoft/sp-core-library": "1.6.0",
    "@microsoft/sp-webpart-base": "1.6.0",
    "@microsoft/sp-lodash-subset": "1.6.0",
    "@microsoft/sp-office-ui-fabric-core": "1.6.0",
    "@types/webpack-env": "1.13.1",
    "@types/es6-promise": "0.0.33",
    "@types/redux": "3.6.0"
  },
  "devDependencies": {
    "@microsoft/sp-build-web": "1.6.0",
    "@microsoft/sp-module-interfaces": "1.6.0",
    "@microsoft/sp-webpart-workbench": "1.6.0",
    "@types/chai": "3.4.34",
    "@types/mocha": "2.2.38",
    "@types/redux": "^3.6.0",
    "ajv": "~5.2.2",
    "bootstrap": "^4.1.3",
    "gulp": "~3.9.1",
    "react-hot-loader": "^4.3.12",
    "react-redux": "5.0.6",
    "react-router-dom": "4.3.1",
    "redux": "3.7.2",
    "redux-thunk": "2.2.0",
    "tslint-microsoft-contrib": "~5.0.0"
  }
}
```

A continuación dentro de la carpeta donde esta WebPart nos crearemos las siguientes carpetas: api, common, components,model, reducer

Empezaremos en primer lugar vamos a definirnos los modelos que vamos a utilizar, para ello nos crearemos los siguientes ficheros en la carpeta Model
**IContinent.ts**
```js
export default interface IContinent{
    name:string;
}
```
**ICountry.ts**
```js
import IContinent from "./IContinent";
export default interface ICountry {
    name:string;
    continent:IContinent;
}
```
**IPlayer.ts**
```js
import ICountry from "./ICountry";
export default interface IPlayer {
    id:string;
    fullName:string;
    club:string;
    image:string;
    country:ICountry;
}
```
Con los modelos creados crearemos las llamadas a nuestra API, dentro de esta carpeta lo que vamos a tener es un fichero en el que vamos a tener los métodos en los que llamamos a nuestro API (esta API puede ser apis de SharePoint, apis custom o apis de terceros). Este fichero lo llamaremos **index.ts**
y tendrá el siguiente contenido:
```js
import IPlayer from "../model/IPlayer";
import ICountry from "../model/ICountry";
import IContinent from "../model/IContinent";

const mapToContinent=(response:any):IContinent=>{
    const result:IContinent={
        name:response.name
    };
    return result;
};
const maptoCountry=(response:any):ICountry=>{
    const result:ICountry={
        continent: mapToContinent(response.contintent),
        name:response.name
    };

    return result;
};

const mapToIDPlayer= (response:any):IPlayer =>{
    return {
        country:maptoCountry(response.country),
        club:response.club,
        id:response.id,
        fullName:response.fullName,
image:response.image
    };
};

const mapToPlayer = (response:any): IPlayer[] => {

    const result: IPlayer[] = [];
    response.map((item:any) => {
        const playerMap: IPlayer = {
            id:item.id,
            country:maptoCountry(item.country),
            club:item.club,
            fullName:item.fullName,
            image:item.image
        };
        result.push(playerMap);
    });
    return result;
};


const addPlayerAsync= async (player:IPlayer):Promise<boolean> =>{
    const addManagedURL = `https://localhost:44376/api/player`;
    const obj = {
        body: JSON.stringify(
            {
                Id: player.id,

 FullName: player.fullName,

 Club: player.club,

 Image: player.image,
 Country: {
    Name: "Portugal",
    Contintent: {
        Name: "Europe"
    }
}
            }
        )    ,

      headers:
       { 
           'Content-Type': 'application/json' 
        } ,    

        method: 'POST'

        
    };
    const response = await fetch(addManagedURL, obj);

    return response.ok;

};


const updatePlayerAsync= async (player:IPlayer):Promise<boolean> =>{
    const addManagedURL = `https://localhost:44376/api/player`;
    const obj = {
        body: JSON.stringify(
            {
                Id: player.id,

 FullName: player.fullName,

 Club: player.club,

 Image: player.image,
 Country: {
    Name: "Portugal",
    Contintent: {
        Name: "Europe"
    }
}
            }
        )    ,

      headers:
       { 
           'Content-Type': 'application/json' 
        } ,    
        method: 'PUT'

        
    };
    const response = await fetch(addManagedURL, obj);

    return response.ok;

};



const getPlayerAsync = async (): Promise<IPlayer[]> => {
    const addManagedURL = `https://localhost:44376/api/player`;
    const obj = {
      headers:
       { 
           'Content-Type': 'application/json' 
        } ,    
        method: 'GET'                 
    };

    const response = await fetch(addManagedURL, obj);
    const responseOne = await (response.json());
    return mapToPlayer(responseOne);
};

const getPlayerIdAsync = async (id:string): Promise<IPlayer> => {
    const addManagedURL = `https://localhost:44376//api/player/` +id;
    const obj = {
      headers:
       { 
           'Content-Type': 'application/json' 
        } ,    
        method: 'GET'          
    };

    const response = await fetch(addManagedURL, obj);
    const responseOne = await (response.json());
    return mapToIDPlayer(responseOne);
};

export const playerAPI = {
    addPlayerAsync,    
    getPlayerAsync,    
    getPlayerIdAsync,    
    updatePlayerAsync    
};
```
Una vez tenemos las llamadas de la API el siguiente paso que vamos a realizar es crear las acciones. Para ello dentro de la carpeta **Actions** nos crearemos dos subcarpetas **actions** y **constants**
En constants nos crearemos un fichero **actionTypes.ts** donde vamos a tener las acciones a realizar 
```js
export const actionTypes = {
    HTTP_CALL_END: 'HTTP_CALL_END',
    HTTP_CALL_START: 'HTTP_CALL_START',
    LOAD_PLAYER:'LOAD_PLAYER',
    LOAD_ID_PLAYER: 'LOAD_ID_PLAYER',
    ADD_PLAYER:'ADD_PLAYER',
    UPDATE_PLAYER:'UPDATE_PLAYER',
    DELETE_PLAYER:'DELETE_PLAYER'
  };
  ```
  En la carpeta constants nos crearemos un fichero por cada una de las acciones que vamos a implementar. 
  **addPlayerAction.ts** 
```js
import { playerAPI } from "../../api";
import { actionTypes } from "../../common/constants/actionTypes";
import IPlayer from "../../model/IPlayer";

const loadPlayerCompleted = (result: boolean) => ({
    type: actionTypes.ADD_PLAYER,
    payload: result,
    meta: {
        httpEnd: true
    }        
});

export const addPlayerAction = (player:IPlayer) => (dispatch: any) => {
    playerAPI.addPlayerAsync(player).then((result) => {        
        history.back();
        dispatch(loadPlayerCompleted(result));
    });    
};
```
**loadIdPlayer.ts**
```js
import { playerAPI } from "../../api";
import { actionTypes } from "../../common/constants/actionTypes";
import IPlayer from "../../model/IPlayer";
const loadIdPlayerCompleted = (result: IPlayer) => ({
    type: actionTypes.LOAD_ID_PLAYER,
    payload: result,
    meta: {
        httpEnd: true
    }        
});
export const loadIdPlayerAction = (id:string) => (dispatch: any) => {
    playerAPI.getPlayerIdAsync(id).then((result) => {
        dispatch(loadIdPlayerCompleted(result));
    });    
};
```
**loadPlayerAction.ts** 
```js
import { playerAPI } from "../../api";
import { actionTypes } from "../../common/constants/actionTypes";
import IPlayer from "../../model/IPlayer";
const loadPlayerCompleted = (result: IPlayer[]) => ({
    type: actionTypes.LOAD_PLAYER,
    payload: result,
    meta: {
        httpEnd: true
    }        
});
export const loadPlayerAction = () => (dispatch: any) => {
    playerAPI.getPlayerAsync().then((result) => {
        dispatch(loadPlayerCompleted(result));
    });    
};
```
**updatePlayerAction.ts**
```js
import { playerAPI } from "../../api";
import { actionTypes } from "../constants/actionTypes";
import IPlayer from "../../model/IPlayer";

const updatePlayerCompleted = (result: boolean) => ({
    type: actionTypes.UPDATE_PLAYER,
    payload: result,
    meta: {
        httpEnd: true
    }        
});
export const updatePlayerAction = (player:IPlayer) => (dispatch: any) => {
    playerAPI.updatePlayerAsync(player).then((result) => {
        history.back();
        dispatch(updatePlayerCompleted(result));
    });    
};
```

Una vez ya tenemos creada las Acciones, el siguiente paso es crear los Reducers que van a hacer uso nuestra aplicación para ello en la carpeta **Reducer** nos creamos los siguientes ficheros 
**addPlayerReducer.ts**
```js
import { actionTypes } from '../common/constants/actionTypes';

const handleaddPlayerCompleted = (state: boolean=true, payload: boolean) => {
    return payload;
};

export const addPlayerReducer = (state: boolean , action: any) => {
    switch (action.type) {
        case actionTypes.ADD_PLAYER:
            return handleaddPlayerCompleted(state, action.payload);
    }
    return state;
};
```
**loadPlayerIdReducer.ts**
```js
import { actionTypes } from '../common/constants/actionTypes';
import IPlayer from '../model/IPlayer';
const emptyPlayer:IPlayer={
    fullName:'',
    club:'',
    country:{
        name:'',
        continent:{
            name:''
        }},
        image:'',

        id:''
    };
const handleloadPlayerIdCompleted = (state: IPlayer, payload: IPlayer) => {
        return payload;
    };
export const loadPlayerIdReducer = (state: IPlayer = emptyPlayer, action: any) => {
    switch (action.type) {
        case actionTypes.LOAD_ID_PLAYER:
            return handleloadPlayerIdCompleted(state, action.payload);
    }
    return state;
};
```
**loadPlayerReducer.ts**
```js
import { actionTypes } from '../common/constants/actionTypes';
import IPlayer from '../model/IPlayer';

const handleloadPlayerCompleted = (state: IPlayer[], payload: IPlayer[]) => {
    return payload;
};

export const loadPlayerReducer = (state: IPlayer[] = [], action: any) => {
    switch (action.type) {
        case actionTypes.LOAD_PLAYER:
            return handleloadPlayerCompleted(state, action.payload);
    }
    return state;
};
```
**updatePlayerReducer.ts**
```js
import { actionTypes } from '../common/constants/actionTypes';

const handleupdatePlayerCompleted = (state: boolean=true, payload: boolean) => {
    return payload;
};

export const updatePlayerReducer = (state: boolean , action: any) => {
    switch (action.type) {
        case actionTypes.UPDATE_PLAYER:
            return handleupdatePlayerCompleted(state, action.payload);
    }
    return state;
};
```
**index.ts**
```js
import IPlayer from '../model/IPlayer';
export interface IStateReducer {
    playerCollection: IPlayer[];
    player:IPlayer;
}
```

Una vez tenemos los reducers implementados el siguiente paso que vamos a realizar es crear nuestros componentes **Player**. para ello dentro de la carpeta components nos creamos una carpeta **Player** en la que nos vamos a crear los siguientes ficheros
**addPlayer.tsx**
```js
import * as React from 'react';
import IPlayer from './../../model/IPlayer';

export interface IAddPlayerProps {
addPlayer:(player:IPlayer)=>Promise<boolean>;
}

export interface IAddPlayerState{
    name:string;
    club:string;
    country:string;
    image:string;
}


export class AddPlayer extends React.Component<IAddPlayerProps, any> {
    
    constructor(props:IAddPlayerProps) {
        super(props);
        this.onClick.bind(this);
        this.onChangeClub.bind(this);
        this.onChangeCountry.bind(this);
        this.onChangeImage.bind(this);
        this.onChangeName.bind(this);
        this.state={
            name:'',
            club:'',
            country:'',
            image:''
        };
    }

    public onClick(){
        this.props.addPlayer({
            id:'',
            club:this.state.club,
            fullName:this.state.name,
            image:this.state.image,
            country:{                
                name:this.state.name,
                continent:{                
                name:''
                }
            }
        });
    }


public onChangeName(value:any){
this.setState({name:value.target.value});
}
public onChangeClub(value:any){
    this.setState({club:value.target.value});
}

public onChangeCountry(value:any){
    this.setState({country:value.target.value});
}
        public onChangeImage(value:any){
            this.setState({image:value.target.value});
            }

  

    public render() {    
        
        const handleOnClickName:any = (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeName(event);
          
          }

          const handleOnClickClub:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeClub(event);
          
          }

          const handleOnClickCountry:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeCountry(event);
          
          }

          const handleOnClickImage:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeImage(event);
          
          }


          const handleOnClick:any=(evente:any)=>{
              this.onClick();
          }
return (
<div className="container">
	<div className="row">
      <div className="col-md-6 col-md-offset-3">
        <div className="well well-sm">
          <fieldset>
            <legend className="text-center">Alta 1</legend>
    
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="name">Name</label>
              <div className="col-md-9">
                <input id="name" name="name" type="text" placeholder="Your name" onChange={handleOnClickName} className="form-control"/>
              </div>
            </div>
    

            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="Club">Club</label>
              <div className="col-md-9">
                <input id="club" name="club" type="text" placeholder="Club"  onChange={handleOnClickClub} className="form-control"/>
              </div>
            </div>
    
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="message">Country</label>
              <div className="col-md-9">
              <input id="country" name="country" type="text" placeholder="Contry" onChange={handleOnClickCountry} className="form-control"/>
                       </div>
            </div>
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="message">Image</label>
              <div className="col-md-9">
              <input id="image" name="image" type="text" placeholder="Image" onChange={handleOnClickImage} className="form-control"/>
                       </div>
            </div>

            <div className="form-group">
              <div className="col-md-12 text-right">
                <button type="submit" className="btn btn-primary btn-lg" onClick={handleOnClick} >Submit</button>
              </div>
            </div>
          </fieldset>
        </div>
      </div>
	</div>
</div>
);

    }
}
```
**editPlayer.tsx**
```js
import * as React from 'react';
import IPlayer from '../../model/IPlayer';


export interface IEditPlayerProps {
  loadIdPlayer:(id:string)=>Promise<IPlayer>,
  editPlayer:(player:IPlayer)=>Promise<boolean>,
  player:IPlayer,
  match?: any;
}

export interface IEditPlayerState{
    name:string;
    club:string;
    country:string;
    image:string;
    id:string
}

export class EditPlayer extends React.Component<IEditPlayerProps, any> {    
    constructor(props:IEditPlayerProps) {
        super(props);
        this.onClick.bind(this);
        this.onChangeClub.bind(this);
        this.onChangeCountry.bind(this);
        this.onChangeImage.bind(this);
        this.onChangeName.bind(this);
        this.state={
            name:'',
            club:'',
            country:'',
            image:'',
            id:''
        };
    }
    public componentWillMount(){
      
      this.props.loadIdPlayer(this.props.match.params['id']);
    }

  public componentWillReceiveProps(nextProps:IEditPlayerProps, olldProps:any){
this.setState({
            name:nextProps.player.fullName,
            club:nextProps.player.club,
            country:'',
            image:nextProps.player.image,
            id:this.props.match.params['id']
});
  }

    public onClick(){
        this.props.editPlayer({
            id:this.state.id,
            club:this.state.club,
            fullName:this.state.name,
            image:this.state.image,
            country:{                
                name:this.state.name,
                continent:{                
                name:''
                }
            }
        });
    }


public onChangeName(value:any){
this.setState({name:value.target.value});
}
public onChangeClub(value:any){
    this.setState({club:value.target.value});
    }
    public onChangeCountry(value:any){
        this.setState({country:value.target.value});
        }
        public onChangeImage(value:any){
            this.setState({image:value.target.value});
            }
            
    public render() {    
        
        const handleOnClickName:any = (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeName(event);
          
          }

          const handleOnClickClub:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeClub(event);
          
          }

          const handleOnClickCountry:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeCountry(event);
          
          }

          const handleOnClickImage:any=       (event: React.MouseEvent<HTMLElement>) => {
      
            this.onChangeImage(event);
          
          }


          const handleOnClick:any=(evente:any)=>{
              this.onClick();
          }
return (
<div className="container">
	<div className="row">
      <div className="col-md-6 col-md-offset-3">
        <div className="well well-sm">
          <fieldset>
            <legend className="text-center">Edicion</legend>
    
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="name">Name</label>
              <div className="col-md-9">
                <input id="name" name="name" type="text" value={this.state.name} placeholder="Your name" onChange={handleOnClickName} className="form-control"/>
              </div>
            </div>
    

            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="Club">Club</label>
              <div className="col-md-9">
                <input id="club" name="club" type="text" value={this.state.club} placeholder="Club"  onChange={handleOnClickClub} className="form-control"/>
              </div>
            </div>
    
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="message">Country</label>
              <div className="col-md-9">
              <input id="country" name="country" type="text" placeholder="Contry" onChange={handleOnClickCountry} className="form-control"/>
                       </div>
            </div>
            <div className="form-group">
              <label className="col-md-3 control-label" htmlFor="message">Image</label>
              <div className="col-md-9">
              <input id="image" name="image" type="text" value={this.state.image} placeholder="Image" onChange={handleOnClickImage} className="form-control"/>
                       </div>
            </div>

            <div className="form-group">
              <div className="col-md-12 text-right">
                <button type="submit" className="btn btn-primary btn-lg" onClick={handleOnClick} >Editar</button>
              </div>
            </div>
          </fieldset>
        </div>
      </div>
	</div>
</div>
);
    }
}
```
**listPlayer.tsx**
```js
import * as React from 'react';
import { NavLink } from 'react-router-dom';
import IPlayer from '../../model/IPlayer';

export interface IListPlayerProps {
playerCollection:IPlayer[],
loadPlayer:()=>Promise<IPlayer[]>;
}

export class ListPlayer extends React.Component<IListPlayerProps, any> {    
    constructor(props:IListPlayerProps) {
        super(props);
  
    }
    public componentWillMount(){
        this.props.loadPlayer();
    }

    public render() {            
       const playerCollection:IPlayer[]=this.props.playerCollection;
       const exact: boolean = true;    
        let i:number=0;       
return (

<div className="container">
<div className="row">
<div className="col-lg-12 my-3">
            <div className="pull-right">
                <div className="btn-group">
                <NavLink className='btn btn-info' exact={exact} to={'/anyadir'} >
        Añadir
            </NavLink>
                </div>
            </div>
        </div>
    </div> 
    
    <div id="products" className="row view-group">
        {            
            playerCollection.map((item:IPlayer)=>{
                i=i+1;
                const editar: string = '/editar/'+item.id;
                return (
                    <div className="item col-xs-4 col-lg-4" key={item.id}>
                    <div className="thumbnail card">
                        <div className="img-event">
                            <img className="group list-group-image img-fluid" src={item.image} alt="" />
                        </div>
                        <div className="caption card-body">
                            <h4 className="group card-title inner list-group-item-heading">
                                {item.fullName}</h4>
                            <p className="group inner list-group-item-text">
                                {item.club}
                            </p>
                            <div className="row">                                
                                <div className="col-xs-12 col-md-6">
                                <NavLink exact={exact} to={editar} className="btn btn-success" >
        Editar
            </NavLink>
                                    
                                </div>
                            </div>
                        </div>
                    </div>
                </div>                );                
            })
        } 
   </div>                    
</div>)
    }

}
```
Una vez hemos implementado nuestros componentes de ReactJS, vamos a crearnos los contenedores que es donde vamos a pasar los métodos, elementos que estan almacenados mediante **Flux** para ello nos crearemos los siguientes container

 **addPlayerContainer.ts** 
 ```js
 import { connect } from 'react-redux';
import { AddPlayer } from './addPlayer';
import {IStateReducer} from '../../reducer';
import IPlayer from '../../model/IPlayer';
import {addPlayerAction} from '../../common/actions/addPlayerAction';

const mapStateToProps = (state: IStateReducer) => ({
});

const mapDispatchToProps = (dispatch: any) => ({
    addPlayer: (player:IPlayer) => dispatch(addPlayerAction(player))
});

export const AddPlayerContainer: any = connect(
    mapStateToProps,
    mapDispatchToProps
)(AddPlayer);
```
 **editPlayerContainer.ts** 
 ```js
 import { connect } from 'react-redux';
import { EditPlayer } from './editPlayer';
import { IStateReducer } from '../../reducer';
import { loadIdPlayerAction } from '../../common/actions/loadIdPlayer';
import IPlayer from '../../model/IPlayer';
import { updatePlayerAction } from '../../common/actions/updatePlayerAction';

const mapStateToProps = (state: IStateReducer) => ({
    player: state.player
});

const mapDispatchToProps = (dispatch: any) => ({
    loadIdPlayer: (id:string) => dispatch(loadIdPlayerAction(id)),
    editPlayer:(player:IPlayer)=>dispatch(updatePlayerAction(player)),
});

export const EditPlayerContainer: any = connect(
    mapStateToProps,
    mapDispatchToProps
)(EditPlayer);
 ```
**listPlayerContainer.ts** 
 ```js
 import { connect } from 'react-redux';
import { ListPlayer } from './listPlayer';
import { IStateReducer } from '../../reducer';
import { loadPlayerAction } from '../../common/actions/loadPlayerAction';

const mapStateToProps = (state: IStateReducer) => ({
    playerCollection: state.playerCollection,
});

const mapDispatchToProps = (dispatch: any) => ({
    loadPlayer: () => dispatch(loadPlayerAction())
});

export const ListPlayerContainer: any = connect(
    mapStateToProps,
    mapDispatchToProps
)(ListPlayer);
 ```

 Una vez tenemos ya implementado nuestra aplicación vamos a crearnos por un lado la **store** Para ello dentro de la carpeta Components nos creamos un fichero **store.ts**
 ```js
import { IStateReducer } from "../reducer";
import { applyMiddleware, createStore as reduxCreateStore, Store } from "redux";
import reduxThunk from "redux-thunk";
import { combineReducers } from "redux";

import { loadPlayerReducer } from '../reducer/loadPlayerReducer';
import  { loadPlayerIdReducer}  from '../reducer/loadPlayerIdReducer';

export function createStore(initialState?: IStateReducer): Store<IStateReducer> {
const middlewares = [
  reduxThunk
];

return reduxCreateStore(
  combineReducers<IStateReducer>({
    playerCollection: loadPlayerReducer,
    player:loadPlayerIdReducer
  }),  
  applyMiddleware(...middlewares)
);
}
```

Ahora como estamos viendo estamos creando una aplicación SPA por lo que nos hará falta definirnos las rutas y el marco de las mismas para ello nos creamos un fichero **Layout.tsx**
```js
import * as React from 'react';
export interface ILayoutProps {
    children?: React.ReactNode;
}
export class Layout extends React.Component<ILayoutProps> {
    public render() {
        return (
            <div> 
                <div>
                    {this.props.children}
                </div>
            </div>
        );
    }
}
```
y nos crearemos un fichero de routas **routes.tsx**
```js
import * as React from 'react';
import { Route } from 'react-router';
import { Layout } from './Layout';
import {ListPlayerContainer} from './player/listPlayerContainer';
import  {AddPlayerContainer} from './player/addPlayerContainer';
import {EditPlayerContainer } from './player/editPlayerContainer';
const exact: boolean = true;
export const routes =
    <Layout>
    <Route  exact={exact}  path='/' component={ListPlayerContainer} />
    <Route exact={exact} path='/anyadir' component={AddPlayerContainer} />
    <Route exact path="/editar/:id" component={EditPlayerContainer} />
    </Layout>;
```

Una vez tenemos el armazón montado. Lo que nos quedaría por crear el punto de arranque de nuestra aplicación para ello dentro de SharePoint lo que necesitamos crearnos un componente App de la siguiente forma:
**app.tsx**
```js
import * as React from 'react';
import { Version } from '@microsoft/sp-core-library';
import 'bootstrap/dist/css/bootstrap.css';
import {
  BaseClientSideWebPart,
  IPropertyPaneConfiguration,
  PropertyPaneTextField
} from '@microsoft/sp-webpart-base';
import { AppContainer } from 'react-hot-loader';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import * as RoutesModule from './routes';

export interface IReactReduxNinjaWebPartProps {
  description: string;
  store:any;
  domElement:any,
}

export default class App  extends React.Component<IReactReduxNinjaWebPartProps, any> {  
  public render(): any {    
      const routes = RoutesModule.routes;
      const baseUrl = "/temp/workbench.html";
       return(
       <AppContainer>
        <Provider store={this.props.store}>
            <BrowserRouter children={routes} basename={baseUrl} />
        </Provider>
    </AppContainer>)
       
  }
 
  protected get dataVersion(): Version {
    return Version.parse('1.0');
  }

  protected getPropertyPaneConfiguration(): IPropertyPaneConfiguration {
    return {
      pages: [
        {
          header: {
            description: ""
          },
          groups: [
            {
              groupName: "",
              groupFields: [
                PropertyPaneTextField('description', {
                  label: ""
                })
              ]
            }
          ]
        }
      ]
    };
  }
}
```

Todo esto esta muy chulo pero ... todavia tenemos que hacer un paso más y es modificar el punto de arranque de nuestra aplicación para ello en el ts donde se invoca al proyecto de React vamos a tener que realizar las siguientes modificaciones en el render 
```js
 private store:Store<{}>;
  constructor(props: IReactReduxNinjaWebPartProps) {
    super();
    this.store=createStore();
        }
  public render(): void {
    if (this.renderedOnce) { return; }

    const element: React.ReactElement<any > = React.createElement(
      App,
      {
        description: this.properties.description,
        domElement:this.domElement,
        store:this.store
      }
    );

    ReactDom.render(element, this.domElement);
    
       
  }
```
