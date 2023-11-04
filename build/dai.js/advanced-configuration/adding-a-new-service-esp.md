# Agregando un Nuevo Servicio

## Resumen

Puedes aprovechar la arquitectura conectable de esta librería eligiendo diferentes implementaciones para los servicios y/o agregando nuevos roles de servicio. Un servicio es solo una clase de Javascript que hereda de `PublicService` (Servicio Público), `PrivateService` (Servicio Privado) o `LocalService` (Servicio Local), y contiene métodos públicos.

Puede depender de otros servicios a través de nuestro [marco de inyección de dependencia](https://github.com/makerdao/dai.js/tree/dev/packages/services-core) incorporado y también puede ser configurado a través de las opciones/archivo de configuración de Maker.

## Pasos para agregar un nuevo servicio

{% hint style="info" %}
La información que se encuentra descripta debajo, es para desarrolladores que deseen agregar a dai.js, pero también puedes agregar servicios a través de [*plugins*](../maker/plugins.md).
{% endhint %}

Aquí se encuentran los pasos para agregar un nuevo servicio llamado *ExampleService* (Servicio Ejemplo):

\(1\) En el directorio del src, crea un archivo ExampleService.js en uno de los subdirectorios.

\(2\) En `src/config/DefaultServiceProvider.js`, importa ExampleService.js y agrégalo a la matriz `\_services`.

\(3\) Crea una clase llamada *ExampleService* en ExampleService.js

\(4\) El servicio debe extender uno de los siguientes:

* `PrivateService` (Servicio Privado) - requiere tanto una conexión de red como autenticación
* `PublicService` (Servicio Público) - requiere solo una conexión de red
* `LocalService` (Servicio Local) - no tiene requisitos

Para más información, lee la sección del Ciclo de Vida del Servicio que se encuentra debajo.

\(5\) En el constructor, llama al constructor de la clase padre con los siguientes argumentos:

* El nombre del servicio. Así es como el servicio puede ser referenciado por otros servicios.
* Una matriz de los nombres de los servicios de los que depender.

\(6\) Agrega los métodos públicos necesarios.

```javascript
//código de ejemplo del ExampleService.js para los pasos 3-6
import PublicService from '../core/PublicService';

export default class ExampleService extends PublicService {
    constructor (name='example') {
        super(name, ['log']);  
    }

    test(){
        this.get('log').info('test');
    }
```

\(7\) Si tu servicio será utilizado para reemplazar un servicio que se encuentre por defecto \(el listado completo de los roles de los servicios por defecto pueden encontrarse en `src/config/ConfigFactory`\), entonces, saltea este paso. De lo contrario, deberás agregar un nuevo rol de servicio \(e.j. "example" - ejemplo\) a la matriz *ServiceRoles* (Roles de Servicio) en `src/config/ConfigFactory`.

\(8\) Crea el archivo `ExampleService.spec.js` correspondiente en el directorio de testeo. Escribe una prueba en el archivo de prueba que crea un objeto de Maker utilizando tu servicio.

```javascript
//código de ejemplo del ExampleService.js para el paso 8
import Maker from '../../src/index';

//paso 8: un nuevo rol de servicio ('example' - ejemplo) es utilizado
test('test 1', async () => {
  const maker = await Maker.create('http', {example: "ExampleService"});
  const exampleService = customMaker.service('example');
  exampleService.test(); //logs "test"
});

//paso 8: un servicio personalizado reemplaza un servicio pr  edeterminado (Web3)
test('test 2', async () => {
  const maker = await Maker.create('http', {web3: "MyCustomWeb3Service"});
  const mycustomWeb3Service = maker.service('web3');
});
```

\(9\) \(Opcional\) Implementa las funciones de servicio del ciclo de vida relevantes \(initialize\(\) (iniciar), connect\(\) (conectar), and authenticate\(\)\) (autenticar). Para más información, lee la sección del Ciclo de Vida del Servicio que se encuentra debajo.

\(10\) \(Opcional\) Permite configuraciones. La configuración específica del servicio se puede pasar a un servicio mediante el archivo de configuración de Maker o de las opciones de configuración. Se puede acceder a esta configuración específica del servicio desde dentro de un servicio cuando el parámetro se pasa a la función de inicialización \(consulta la sección Ciclo de Vida del servicio a continuación\)

```javascript
//paso 10: en ExampleService.spec.js
const maker = await Maker.create('http', {
    example: ["ExampleService", {
    exampleSetting: "this is a configuration setting"
  }]
});

//paso 10: acceder a los ajustes de configuración en ExampleService.js
initialize(settings) {
  if(settings.exampleSetting){
    this.get('log').info(settings.exampleSetting);
  }
}
```

## Ciclo de Vida de Servicios

Los tres tipos de servicios, mencionados en el paso 4 más arriba, siguen los siguientes diagramas de máquina de estado de la imagen que se encuentra a continuación.

Para especificar lo que implica inicializar, conectar y autenticar, implementa las funciones initialize\(\), connect\(\) y authenticate\(\) en el propio servicio. Este se llamará mientras el administrador del servicio lleva el servicio al estado correspondiente.

```javascript
//ejemplo de la función initialize() en ExampleService.js
  initialize(settings) {
    this.get('log').info('ExampleService is initializing...');
    this._setSettings(settings);
  }
```
Un servicio no terminará de inicializarse/conectarse/autenticarse hasta que todos sus servicios dependientes hayan completado el mismo estado (si es aplicable, por ejemplo, un _LocalService_ - Servicio Local - es considerado como autenticado/conectado, además de inicializado, si ha terminado de inicializarse). El código de ejemplo muestra cómo esperar a que el servicio esté en un estado determinado.

```javascript
const maker = await Maker.create('http', {example: "ExampleService"});
const exampleService = customMaker.service('example');

//espera a que el servicio de ejemplo y sus dependencias se encuentren iniciadas
await exampleService.manager().initialize();

//espera a que el servicio de ejemplo y sus dependencias se encuentren conectadas
await exampleService.manager().connect();

//espera a que el servicio de ejemplo y sus dependencias se encuentren autenticadas
await exampleService.manager().authenticate();

//también puede utilizar la sintaxis de llamada de retorno
exampleService.manager().onConnected(()=>{
    /*se ejecuta luego de conectarse*/
});

//espera a que todos los servicios utilizados por el objeto de Maker sean autenticados
maker.authenticate();
```

## Agregando Eventos Personalizados

Una forma de agregar un evento es "registrar" una función que se llame en cada nuevo bloque, utilizando la función `registerPollEvents\(\)` (registrar los eventos de las encuestas) del servicio de eventos. Por ejemplo, aquí hay un código del servicio de precios. `this.getEthPrice\(\)` se llamará en cada nuevo bloque y, si el estado ha cambiado desde la última llamada, se emitirá un evento de precio/ETH\_USD con la carga útil { precio: \[nuevo\_precio\] } .

Otra forma de agregar un evento es emitir manualmente un evento utilizando la función de emisión del servicio de eventos. Por ejemplo, cuando Web3Service se inicializa, emite un evento que contiene información sobre el proveedor.

Ten en cuenta que llamar a `registerPollEvents` y `emit\(\)` directamente en el servicio de eventos, como en los dos ejemplos anteriores, registrará eventos en la instancia de emisor de eventos "predeterminada". Sin embargo, puedes crear una nueva instancia de emisor de eventos para tu nuevo servicio. Por ejemplo, el objeto CDP/_vault_ define su propio emisor de eventos, como se puede ver aquí, llamando a la función `buildEmitter\(\)` (crear emisor) del servicio de eventos.

```javascript
//en PriceService.js
this.get('event').registerPollEvents({
      'price/ETH_USD': {
        price: () => this.getEthPrice()
      }
    });

//en Web3Service.js
this.get('event').emit('web3/INITIALIZED', {
  provider: { ...settings.provider }
});

//en el constructor en el Cdp.js
this._emitterInstance = this._cdpService.get('event').buildEmitter();
this.on = this._emitterInstance.on;
this._emitterInstance.registerPollEvents({
  COLLATERAL: {
    USD: () => this.getCollateralValueInUSD(),
    ETH: () => this.getCollateralValueInEth()
  },
  DEBT: {
    dai: () => this.getDebtValueInDai()
  }
});
```
