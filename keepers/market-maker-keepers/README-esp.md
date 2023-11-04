# _Keepers_ de Mercado de Maker 

## Introducción 

Una gran parte del Protocolo de Maker consiste en incentivar a agentes externos, llamados **Keepers** \(los cuales pueden ser humanos pero normalmente son bots automatizados\). Los _Keepers_ de Mercado de Maker trabajan creando una serie de órdenes en las llamadas **bands** (“bandas”) \(definidas más adelante\), que son configuradas con un archivo JSON que contiene parámetros como _spreads_, compromiso máximo, etc. En resumen, el repositorio `market-maker-keeper` es un conjunto de _Keepers_ que facilitan la creación de mercados en _exchanges_. Por ejemplo, tradear DAI motivado por la convergencia a largo plazo esperada alrededor del `Target Price` indicado. Esta guía está dedicada a mostrarte cómo crear tu primer Bot _Keeper_ de Mercado de Maker, así como también educar a la comunidad y ayudar tanto a usuarios como desarrolladores a entender el valor de este increíble _software_. Estamos orgullosos de decir que todo el código necesario para crear tu propio Bot _Keeper_ de Mercado de Maker es código _open-sourced_ (“abierto”).

**¡Visita la guía si estás interesad@ en poner en marcha tu propio Bot _Keeper_ de Mercado de Maker!**