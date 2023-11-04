# _Keepers_ de Subastas

## Introducción   

El Protocolo de Maker, que impulsa al DAI Multicolateral \(MCD\), es un sistema basado en _smart contracts_ (contratos inteligentes) que respalda y estabiliza el valor del DAI a través de una combinación dinámica de _vaults_ \(anteriormente conocido como CDPs\), mecanismos de retroalimentación autónomos y actores externos incentivados. Para mantener el sistema en un estado financiero estable, es importante evitar que tanto la deuda como el excedente se acumulen más allá de ciertos límites. Aquí es donde entran en juego las Subastas y los _Keepers_ de Subastas. El sistema ha sido diseñado de manera que haya tres tipos de Subastas en el sistema: Subastas de Excedentes, Subastas de Deuda y Subastas de Colateral. Cada subasta es activada como resultado de circunstancias específicas.  

Los _Keepers_ de Subastas son actores externos que se ven incentivados por oportunidades de beneficios o ganancias para contribuir con los sistemas descentralizados. En el contexto del Protocolo de Maker, estos agentes externos están incentivados para automatizar ciertas operaciones alrededor la _blockchain_ de _Ethereum_. Esto incluye: 

* Buscar oportunidades y comenzar nuevas subastas
* Detectar subastas iniciadas por otros participantes
* Ofertar en subastas convirtiendo precios de _tokens_ en ofertas

Mas específicamente, los _Keepers_ participan como postores en las Subastas de Deuda y de Colateral cuando se liquidan las _vaults_ y los _auction-keeper_ (“_keepers_ de subastas”) permiten la interacción automática con estas subastas MCD. Este proceso se automatizó mediante modelos de ofertas específicos que definen el proceso de toma de decisiones, como, en qué situaciones proponer una oferta, con qué frecuencia ofertar, qué tan alto ofertar, etc. Ten en cuenta qué modelos de ofertas son creados basándose en estrategias determinadas individualmente.     

Para todos los interesados en configurar su propio Bot _Keeper_ de Subastas, les recomendamos consultar la siguiente guía.  