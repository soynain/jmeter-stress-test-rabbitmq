# jmeter-stress-test-rabbitmq
A repository to remember fundamentals of rabbit mq, and get in deep with that

Afortunadamente ya conseguí un puesto de trabajo muy bueno. Sin embargo no significa que deba de detenerme
en el camino del aprendizaje, más en las áreas de computación.

Aproveché estos días a tomarme un break y disfrutar un poco de los frutos que he cosechado el año pasado y estamos listos
para profundizar en los tópicos que debamos de estudiar, de una manera más profunda y no tan superficial, y en poco tiempo.

En el universo de la arquitectura de microservicios hay patrones de implementación que solucionan ciertos escenarios, ya 
que el eje principal de los microservicios es el ownership, dominios y la eventual consistencia. ¿Es importante aprender teoría?
si y si!! estos conceptos que tu dirías al principio: parece teoria de escuela, son conceptos muy escenciales que si debes
de dominar como si aun fueras a una escuela!!! y te los tienes que aprender... creeme, si, porque en campos profesionales
no solo te los preguntan, hay referencias directas a los mismos.

Un sitio REALMENTE ÚTIL que te puedo garantizar te servirá para absorver esa teoría es el siguiente: https://microservices.io/patterns/microservices.html
está garantizado que tiene esos patrones exactos. Recuerden que la complejidad de los micros recae en monitoreo, implementación, y costos.

Un patrón que quisiera testear en este repositorio es la coreografía SAGA. Es una arquitectura aplicada en la arquitectura "común" de una base de datos por micro.
Es el dominio de la eventual consistencia. 

Cuando tu modelas un microservicio, dependiendo problemática y negocio, o usas una bdd central o las esparces. Para diseñar este tipo de bases de datos,
las diseñas normal, con tablas unificadas pero con relaciones implicitas y CATEGORIZADAS POR NEGOCIO, cuando dividas tu aplicativo o migres tienes
que agrupar y ahí categorizas. Tradicionalmente por monolito, por medio de sync/async y funciones invocas multiples métodos, un método por ejemplo
para validar inventario, si está valido, crear el pedido o apartarlo en carrito de compras, confirmas pago, descuentas el stock del carrito y creas la orden en el
modulo de almacen. Para toda esta orquestación, ocupas uno o más métodos, pero dentro de un monolito.

En un microservicios tienes el servicio de stock, servicio de pagos, servicio de catalogo, servicio de usuario (historial de compras, actividad), cómo
cambia la orquestación del mismo flujo con saga? que tu operación parte del servicio de stock, confirma el pedido sobre el mismo, y de ahí dispara un evento al servicio
de pagos para empezar la transacción del pago, dispara otro evento de vuelta al servicio de stock y confirma el descuento del inventario de stock. Pero esta simple
operación ya no la crea desde un método o monolito, si no desde servicios distribuidos.

Eso es lo que SAGA nos ayuda a crear: sincronización de tablas remotas en servicios diferentes de acuerdo a una coreografía diseñada.

Hay dos estilos de orquestación de eventos en resumidas:

-Lo orquestas todo desde un solo método, en un servicio fijo, pero si falla, es sobre la instanciación
del método padre

-Se orquesta el flujo conforme un bloque de negocio termina una actividad, sin una orquesta central, pero eventual. Se le denomina coreografía.

<img width="2250" height="2624" alt="image" src="https://github.com/user-attachments/assets/441b15f3-7c92-40f4-a685-8bd9f51035ff" />


Esos son los dos tipos de SAGA: orquestación y coreografía. Hay un ejemplo más detallado acá https://microservices.io/patterns/data/saga.html
¿Cuáles son las desventajas de SAGA? inconsistencias, puede haber n eventos que tengan el estado anterior y no el actual que probablemente un saga
ya actualizó. Ahí entra en concepto la eventual consistencia.

## Avances 25/02/2026

Para poder estructurar mejor mis sagas y poder crear una arquitectura escalable para un ejemplo de saga's con varias peticiones en un entorno
con un contenedor con 100 mb de recursos para el broker de mensajes y 80 mb para los dos micros, quise darme un repaso por la arquitectura hexagonal.

No desglosaré la explicación de la arquitectura hexagonal, ya que para estudiarlo mejor he hecho apuntes a mano.


## Avances 26/02/2026

Hoy definiremos un resumen de la arquitectura hexagonal, para esto hice apuntes a mano y también ya he absorbido estrategias principales
para la estructuración interactiva de los microservicios.

## Arquitectura hexagonal

Arquitectura de interacción entre aplicaciones, que posibilita la separación a nivel aplicación de las responsabilidades
de negocio aparte de las maneras en que la aplicación interactua con entidades externas.

Quiere decir que tu aplicación no depende de implementaciones de librerias predeterminadas, de librerias de bases de datos o de
clientes para API's. Lo cual facilita tu testing.

La estructura va asi:

````markdown.md
*Infraestructure*
  *Adapters
    *In
      ExampleController.java
    *Out
      UsersRepository.java
    *Entities
      *UserEntity.java
*Application
  *services
    UsersService.java
  *mappers
    *UsersMappers.java
*Domain
  *ports
    *in
      RegisterUserUseCase.java
      DeleteUserUseCase.java
    *out
      UserRepositoryPort.java
      *impl
        UserRepositoryPortImpl.java
  *models
      UserRequestDto.java
````

Este es un markdown improvisado basado en varios ejemplos de esta arquitectura, así como los estándares actuales. Dominio
no depende de ninguna capa, aplicación depende de la capa de dominio y de infraestructura, e infra depende de dominio también.

Los Use case son interfaces, que se heredan a los services, los ports se inyectan por DI hacia los services también, y en su IMPL
tu inyectas los repository's de la capa de infraestructura, algún mapper e implementando la interfaz del port. Con eso defines la funcionalidad del
port que inyectas a services y haces tu código más mockeable. Aprenderte esta arquitectura automáticamente te hace conocedor de
las implementaciones Domain Driven Design.

Esta imagen de referencia es la que encontré más útil

<img width="1536" height="1194" alt="image" src="https://github.com/user-attachments/assets/524e05cb-747b-4422-880b-de33d2d45b53" />


## Estrategias de definición de microservicios

Tus micros deben estar bien definidos, todo parte de la parte central de una aplicación
que se denomina service, tu hexagonal es un service, los packages: tus subdominios.

En muchos lugares se catalogan los micros de manera diferente, en un lugar donde trabajé lo llamaban taxonomía,
aunque era un término zonso y fancy para la homologación conjunta de estándar RESTFUL y de services.

Pero en manera más genéricas, un service tiene subdominios que pueden representar modulos que se comunican entre si o tienen relación.

De acuerdo a Chris Richardson, para definir los micros en resumidas se resume en 4 pasos:

1.Define tus endpoints en un preflujo borrador y los métodos de los mismos (o diagrama UML de secuencia o a mano como te acomodes), 

2.Esas operaciones categorizalas por subdominios de acuerdo a las tablas o entidades que ya hayas identificado, si
tu operación toca x tablas, guiate de eso para ir definiendo tus subdominios

4.Separación de responsabilidades de las operaciones definidas y de los subdominios, para esto
debes pensar en si creas nuevos subdominios de acuerdo a ese nuevo feature que estás implementando,
y verificar si sale mejor separarlo en un service separado o integrarlo en un service existente, todo esto
de acuerdo a la complejidad y viabilidad de la operación, ya que sale mejor en algunos escenarios
agrupar las operaciones en un solo service sin usar saga, y en otros escenarios sale mejor usar SAGA
para la sincronización de los mismos, estos sin salirnos de la arquitectura de microservicios 

Todo esto depende de la relevancia de la entidad que estás creando, así como los tiempos de desarrollo.

5.Evaluación de lo aplicado anteriormente

6.Rerfactorización o división a más subdominios si algo falla sobre lo resultante.

Seguiremos estudiando ahora los patrones a mano para SAGAS, y los iremos resumiendo sobre esta sección previo a la práctica.

## Avances 27/02/2026

Muchos patrones de microservicios promueven la eventual consistencia, que es un concepto que consiste en
diseñar arquitecturas asíncronas, no interacciones asincronas. ¿Qué quiere decir esto? que una transacción
pierde su isolación de los principios ACID. Un ejemplo es un inventario de compras en línea, si lo diseñas con
eventual consistencia, muchas peticiones concurrentes te van a disminuir el inventario y encontrarás escenarios donde
tengas inventarios negativos también, para eso creas un query que valide que hay inventario mayor a 0, si tu transacción
valida la compra de un producto con nada de inventarios y está en ceros, debes bloquear esas peticiones y deshacer operaciones 
predecesoras.

Suena fácil, pero no es fácil de implementar. Pero nos estamos adelantando. ¿Ya se hacen una idea de la complejidad
de los micros?

Iremos por partes pero tengan esta problemática en mente. Tienes micros separados, la regla de los micros es que tengan su propia base, recuerden
que al diseñar o implementar micros, no debes DE CAJÓN implementar todos los conceptos, el propósito de un micro es desglosar monolitos, ESPARCER
TRÁFICO, disminuir consumo de recursos, descentralizar. Pero seguimos, en un escenario normal, micro = bdd única.

¿Qué estrategias se pueden seguir para tener el patrón *Database-Per-Service*?

*Table-per-service* = Nomás asignas una tabla por service, lo más "sencillo"

*Schema-per-service* = Un create database por service, permite crear grants y revokes por servicio para controlar el acceso a los recursos.

*Database-per-service* = Una instancia por servicio, solo en casos DE MUUUUCHO TRÁFICO, MILLONES, no miles.

Si tu implementas el segundo escenario... ponle que en postgress al ser orientado a objetos o incluso en mysql puedes hacer query's entre esquemas, claro está.
Pero dependiendo de los datos y la magnitud de ese tráfico, tu haces joins y eso al gestor le cuesta. Aquí entra otro concepto interesante.

## CQRS
Patrón arquitectónico que permite la separación de responsabilidades de lectura y escritura, compuesto por varios subdominios fusionados.

Qué quiere decir esto? que puedes usar dos tablas o dos bases de datos, una tabla de lectura con indices, no normalizada, pero hecha
para consultas rapiditas y otro para lectura, normalizada, sin indices más que solo el primary key, ya que indices ralentizan las inserciones
al igual que los constraints. O también añadir complejidad y trabajo extra sincronizando dos bdd's, un mongo db para lecturas y un relacional 
para escritura.

La idea es que así puedes crear diferentes vistas, fusionando esas entidades que por defecto están separadas.

¿Cuál es su desventaja? exacto!! la eventual consistencia, puede tardar unos milisegundos o segundos en reflejar el cambio, y si son varios eventos encolados??
o escalas u optimizas. ¿Capish? y además asegurar la atomicidad e isolación de esas transacciones con otras estrategias.

Considera que en industrias criticas, la eventual consistencia siempre se debe de enfocar a modulos que no sean criticos,
transacciones que puedan tolerar ese delay, pero si tienes una vista y mandas una info desactualizada... tu *source of truth* es la base de lectura SIEMPRE!!!!,
exacto, tus validaciones en codigo siempre sobre los writes, nunca sobre los read. Los read son para consultas o vistas.

<img width="807" height="775" alt="image" src="https://github.com/user-attachments/assets/db9d400b-66c0-415e-973c-99b8226ac655" />


Ahora, para consultas múltiples hay otra estrategia más sencillita que va concorde a, tus migraciones o muuuy pequeñas arquitecturas.

## API Composition

Nomás consiste en, simplemente hacer endpoints, llamarlos, y hacer filtrado y joins en memoria, puedes usar api gateway, curls secuenciales o concurrentes, 
graphQL, te facilitas la vida.

<img width="474" height="320" alt="image" src="https://github.com/user-attachments/assets/12fb8145-b96b-479d-b36d-f3d9da5cdc4f" />

¿Ves como no todo se debe aplicar de cajón? mucho cuidado enserio, piensa en la escalabilidad y que modulos reciben más tráfico para esas consideraciones.

*Api composition y CQRS* sirven para la consulta de datos en sistemas distribuidos. Si vamos al escenario más complejo que es CQRS, 
tenemos que asegurarnos de un escenario en particular que es, ¿cómo orientas tu arquitectura? lo iremos viendo.


Estos conceptos los hemos estudiado a mano

https://learn.microsoft.com/es-es/azure/architecture/patterns/cqrs

https://microservices.io/patterns/data/domain-event.html

Un recordatorio de que estos conceptos si son muy importantes de aprenderlos, en las grandes ciudades estos protocolos y gobernanzas
son fuertemente seguidos y estandarizados!!!


## Modalidades de CQRS y un deep-check sobren los sistemas event driven (Avances 27/02/2026)

Es fácil a primera vista decir: pues todos mis payloads los sincronizo mandando eventos a las tablas, antes que nada,
voy a explicar los tipicos escenarios de error en un SAGA simple:

<img width="921" height="2116" alt="diagrama_event_source drawio" src="https://github.com/user-attachments/assets/8ca9576d-b0bd-4604-9866-59d82fe34d47" />

