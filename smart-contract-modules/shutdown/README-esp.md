# Apagado de Emergencia del Protocolo de Maker

## Introducción

El Protocolo de Maker, que impulsa el DAI Multi Colateral, es un sistema de _smart contracts_ que respalda y estabiliza el valor del DAI a través de una combinación dinámica de _vaults_, sistemas autónomos de _smart contracts_ y actores externos adecuadamente incentivados. El Precio Objetivo del DAI es 1 dólar estadounidense, 
traduciéndose a una paridad blanda de 1:1 dólar estadounidense. El Apagado de Emergencia es un proceso que se puede utilizar como último recurso para aplicar directamente el precio objetivo a los poseedores de DAI y _vaults_, y proteger el Protocolo de Maker contra ataques a su infraestructura.

> El Apagado se detiene y establece correctamente el Protocolo de Maker al tiempo que garantiza que todos los usuarios, tanto los poseedores de DAI como los de _vaults_, reciban el valor neto de los activos a los que tienen derecho.

En resumen, permite a los poseedores de DAI canjear directamente DAI por colaterales después de un período de procesamiento de Apagado de Emergencia.

## Descripción General del Apagado de Emergencia

* El proceso de inicio de Apagado de Emergencia es descentralizado y controlado por los votantes de MKR, los cuales pueden activarlo al depositar MKR en el Módulo de Apagado de Emergencia.
* El Apagado de Emergencia se activa en caso de emergencias serias, como la irracionalidad del mercado a largo plazo, hackeos y brechas de seguridad.
* El Apagado se detiene y establece correctamente el Protocolo de Maker al tiempo que garantiza que todos los usuarios, tanto los poseedores de DAI como los de _vaults_, reciban el valor neto de los activos a los que tienen derecho.
* Los dueños de _vaults_ pueden retirar, inmediatamente, el exceso de colateral de sus _vaults_ luego de iniciado el Apagado de Emergencia. Ellos pueden realizar esto a través de los _frontends_ de las _vaults_, como "Oasis Borrow", que tiene implementado un soporte de Apagado de Emergencia, así como con herramientas de líneas de comando.
* Los poseedores de DAI pueden, luego de un período de espera determinado por los votantes de MKR, cambiar su DAI por una parte relativa de todos los tipos de colaterales en el sistema. Inicialmente, la Fundación Maker ofrecerá una página web para este fin.
* Los poseedores de DAI siempre reciben el mismo monto relativo de colaterales del sistema, sin importar si fueron los primeros o últimos es realizar el reclamo.
* Los poseedores de DAI también pueden vender sus DAIs a los _keepers_ (si están disponible) para evitar la autogestión de los diferentes tipos de colaterales en el sistema.

Para más información acerca del Apagado de Emergencia del Protocolo de Maker, lee el **Final - Documentación Detallada** así como también la [**Documentación del Módulo de Apagado de Emergencia**](https://docs.makerdao.com/smart-contract-modules/emergency-shutdown-module-esp).&#x20;
