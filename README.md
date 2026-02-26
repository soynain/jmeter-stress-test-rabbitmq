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





