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

El deber ser de una saga es, asegurar el orden de los eventos, si tienes una transacción que dispara un evento, deben ejecutarse en el orden en que
se orquestan:

T1->E1 => T2->E2

Si tienes 5 transacciones, se mandan 5 eventos, y si tienes otras 5 transacciones, se ejecutan después de las primeras 5 transacciones
y los eventos después de los primeros 5 eventos, es consecutivo y en orden.

De acuerdo al diagrama que hemos visto, aquí viene el primer patrón complementario a CRQS que vamos a tocar:

## Event sourcing

Es un patrón que asegura orden, atomicidad y garantía de sincronía de datos entre una o más entidades. Y también nos ayuda a asegurar que, si una transacción hace rollback,
el mensaje no debe ser enviado.

Esto es un nuevo paradigma, ya que te obliga a tratar tus tablas como entidades, te empuja a crear una tabla de eventos en la cual los consumers
se suscriben para en base al estado actual de una transacción, poder re procesar los eventos.

El esquema para estos eventos va de la siguiente forma, de acuerdo a este diagrama:

<img width="1451" height="851" alt="event_store_structure drawio" src="https://github.com/user-attachments/assets/202989ef-2bc1-4c8b-b865-009244dd3f2e" />

El esquema general del event sourcing se compone de dos tablas:

**event store**
Un esquema que nos sirve para guardar el historico de eventos. Imagina que tu usas un SAGA ya sea orquestado o por coreografía. 
Por cada evento tu creas un tópico con un nombre, y el método para mandar ese tópico tiene un nombre, puedes reforzar el historial
de tus eventos incluyendo ambos en columnas separadas, un uuid unico de la fila, un uuid para el objeto o entidad, timestamps,
el payload del objeto que puedes guardar completamente o removiendo propiedades para eliminar peso en tu json. La columna más importante
es el "version", ese version te sirve para SIEMPRE tener el orden de ejecución de tus eventos.

La tabla snapshot obtiene la versión más alta de un conjunto de eventos ejecutados. Esto nos permite la obtención de los objetos sin
necesidad de reproducir todos los eventos.

Puede haber escenarios donde necesites reproducir los eventos otra vez: probablemente para debuguear comportamiento!!. Para eso te sirve
este historial también. También puedes implementar versionados desde los pojos y nombrar a tus eventos con prefijos OrderCreatedV2..V3... etc
para obtener un contrato más versionado, obteniendo una compatibilidad entre contratos.

La desventaja es que al serializar tus objetos a JSON, tus consultas pueden ser más dificiles en cuestión de unificar, osea más aun de lo que es...

Ahí entra en juego CQRS, significa que disparas dos eventos por separado o crear un cron job que haga polling sobre los registros para crear tus vistas, pero
lo más apropiado es que sincronizes tus vistas por broker, tu *source of truth* se vuelve el event store y tu reader se vuelve las vistas que generes con esos eventos.

La tabla de snapshots te puede servir más para esos escenarios, cuando tengas el estado final de ese evento, disparas un evento provisional para que comienzen tales sincronizaciones.

¿Vas viendo esos escenarios cierto? más micros, tablas y necesidades de consultas, recaen en que tengas que crear más tópicos, más sincronismos, por lo tanto más supervisión
y que necesites un equipo más gigante.

Por eso siempre se recalca que no todos estos patrones los debes aplicar a fuerza, pero con esto tienes el panorama de porque mencionan que esta arquitectura es compleja.

A continuación veremos otro patrón que también tiene su nivel de complejidad, y haremos unos comparativos:

## Transactional outbox

Me parece que este es el más común de aplicar. Resuelve el mismo problema que el anterior pero con otro approach.

<img width="1298" height="461" alt="image" src="https://github.com/user-attachments/assets/59f3bcf4-cf6e-43db-b0a3-1bf23ceb732a" />

Es más sencillo de implementar que event sourcing, ya que establece que puedes usar tu patrón *Database per service* de manera normal,
crearás la misma tabla que usas para event sourcing que denominarás outbox_table, pero ya no será tu *source of truth*, serán las tablas normales 
que optimizarás para escritura.

Son de hecho muy semejantes, si tu sistema es full event driven y requieres reconstrucción de eventos y versionado de contratos vete
por event sourcing. Si solo requieres un helper para garantizar una sola entrega de tus eventos sobre orquestaciones saga de manera asincrona
y de sub tareas separadas que no estén acoplados [que no dependan tan secuencialmente], outbox es la más común y sencilla de aplicar.

Volviendo al diagrama, el message relay es un cron job que basicamente hacer pulling por intertvalos para mandarlos por medio de un broker al otro micro.

Para escalar relays, una, aumenta el intervalo por batches de las consultas, y que estas sean asincronas, dos, aumenta el número de pods del relay
conforme los eventos crecen, ya que tu tabla será acid aun.

¿Cuál es la diferencia entre event sourcing y transactional outbox aparte de lo ya analizado? sus implementaciones, y los métodos de subscripción.

Yendo hacia el lado del stack, las bdd's relacionales tienen el CDC que es un log que MySQL usa para guardar logs de las operaciones, resultando
en una lectura un poco más rápida, Debezium es una herramienta que lee ese log y lo dirige al broker directamente, se le conoce a esto como
*Transaction log tailling* y con esto se implementa la susbcripción del event sourcing.

Para el outbox también puedes usar el transaction log pero para quitar complejidades, puedes depender modernamente de un jar que solo haga pulling,
 a lo mejor si lo escalasa y asignas buen recurso, puedas aumentar la eficiencia de los pods de los workers con hilos, pero depende de infra ahí. 
 Ahí solo es eso y los mandas asincronamente al broker listener.

Y si, en transactional outbox también implementas CQRS.

Ahora si ya vimos las dos maneras de implementar SAGA's de acuerdo a las arquitecturas que estés proponiendo, y te vuelvo a recordar:
una arquitectura se diseña y escala en base a necesidades crecientes SIEMPRE. Nunca implementes por moda, por lo general en micros escenciales
es mucho mejor depender de HTTP y asincronismo que sincronias con eventual consistencia, al menos que seas transacciones de ARQUITECTURA ASÍNCRONA,no
comportamiento asincrono por aplicación pero en general sobre ciertos flujos.

Ahora sí, más adelante indagaremos en... implementar transactional outbox y usar JMeter tal vez para tratar de trigerearlos.

https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

https://www.milanjovanovic.tech/blog/implementing-the-outbox-pattern

https://spring.io/blog/2023/10/24/a-use-case-for-transactions-adapting-to-transactional-outbox-pattern


Ahora procederemos a, implementar un ejemplo de transactional outbox con Kafka en este caso porque lo tengo configurado, una tabla que crearemos
con el esquema de ejemplo de outbox, un spring app sin controller con CommandLineRunner para el relay y el producer, y otro jar con un simple endpoint
para trigerear el evento, implementar esas estrategias y reproducir escenarios.

Para la entidad de ejemplo usaremos los siguientes esquemas:

````file.sql
CREATE TABLE Person (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    age INT,
    city VARCHAR(255)
);



CREATE TABLE outbox_messages (
    id SERIAL PRIMARY KEY,
    entity_id INT NOT NULL,
    version int not null,
    type VARCHAR(255) NOT NULL,
    payload JSON NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
````

Como ya hemos visto en las otras prácticas, solo configura un producer simulado, por medio de un getter:

````file.java
@Service
public class BasicService {

    @Autowired
    private KafkaTemplate<String, ExampleEntity> kafkaTemplate;

    public void sendEvent() {
        ExampleEntity exampleEntity = ExampleEntity.builder()
                .name(String.valueOf(String.valueOf(new Random().nextInt(100000)).hashCode()))
                .id(new Random().nextInt(100000))
                .age(new Random().nextInt(100))
                .city("New York")
                .build();

        System.out.println(exampleEntity.toString());
        kafkaTemplate.send("outbox-ejemplo", exampleEntity);
    }
}
````

Nos basamos en otras prácticas para levantar una prueba de kafka en docker, y por medio del get enviamos los eventos:

<img width="1206" height="1021" alt="image" src="https://github.com/user-attachments/assets/efd9694a-3be2-4417-8681-2ca6ede626c5" />

<img width="1376" height="275" alt="image" src="https://github.com/user-attachments/assets/d3439965-743f-4121-b167-73573a861eea" />

Falta el consumer, fácil también (no nos metamos en particiones por ahora):

````file.java
@Service
public class MessageReceiverService {

    @KafkaListener(topics = "outbox-ejemplo", containerFactory = "kafkaListenerContainerFactory")
    public void greetingListener(ExampleEntity message) {
        // process greeting message

        System.out.println("Received Message in group outbox-ejemplo: " + message.toString());
    }

}

````

<img width="1550" height="667" alt="image" src="https://github.com/user-attachments/assets/f7626ea4-3b23-4a66-9951-8357b7971b64" />

Con esto ya comunicamos dos micros por saga, ahora, como se orquesta el saga entre estos dos?


Micro b  guardaria la transacción, la responsabilidad de la saga va de la mano de JPA con @Transactional,
uno para la tabla normal y otro para la tabla outbox.

La ventaja que te trae es que haces ambas transacciones al mismo tiempo.

Para nosotros obtener el resultado del transactional outbox como aquí:

<img width="1466" height="331" alt="image" src="https://github.com/user-attachments/assets/ec378f78-d134-4a2a-bdd0-203ee328e697" />

Es tán simple (en escenarios batch crear micro transacciones o estrategias), como declarar una transacción manejada automáticamente
para que al primer rollback, todas las inserciones dentro de ese bloque no lleguen a la bdd, cuando terminen
las últimas transacciones de ese método, se hace commit y el message relay no lo detectará

````main.java
@Transactional
    public void sendEvent() {
        ExampleEntity exampleEntity = ExampleEntity.builder()
                .name(String.valueOf(String.valueOf(new Random().nextInt(100000)).hashCode()))
               // .id(new Random().nextInt(100000))
                .age(new Random().nextInt(100))
                .city("New York")
                .build();

        System.out.println(exampleEntity.toString());

        /**
         * No mandas el evento por este medio, le corresponde al message relay
         * encargarse de ello
         */
        repository.saveAndFlush(exampleEntity);

        /** Y guardas en la tabla de outbox */
        OutboxEntity outboxEntity;
        try {
            outboxEntity = OutboxEntity.builder()
                    .entityId(exampleEntity.getId())
                   .type("sendEvent")
                    .payload(objectMapper.writeValueAsString(exampleEntity))
                    .build();
                    outboxTableRepository.save(outboxEntity);
        } catch (JsonProcessingException e) {
            // TODO Auto-generated catch block
           throw new RuntimeException(e);
        }
        

        /** Esto ya no aplica, al menos que fuera event sourcing */
        // kafkaTemplate.send("outbox-ejemplo", exampleEntity);
    }

````
Para que no se te pierda nada, usa estrategias de retry con Reslience4J para re intentar una transacción x veces.

Para la entidad del outbox queda definido asúi, con un contador iniciado en 0 gracias a  @Version

````main.java
package com.kafkamicroa.app.entities;

import java.time.LocalDateTime;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import jakarta.persistence.Version;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Entity
@Table(name = "outbox_messages")
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@ToString
@Builder

public class OutboxEntity {

    private static final long serialVersionUID = 132123132131L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "entity_id", nullable = false)
    private Integer entityId;

    @Version
    private Integer version;

    @Column(nullable = false, length = 255)
    private String type;

    @Column(nullable = false, columnDefinition = "json")
    private String payload;

    @Column(name = "created_at", insertable = false, updatable = false)
    private LocalDateTime createdAt;
}

````

Para el message relay, tenemos que hacer un micro sencillo, con la ayuda del commandlinerunner que resulta ser muy ligerisimo.







Y así quedaría el resultado de nuestra implementación, al ser un cron job separado y sincrono es más sencillo detectar incosistencias o errores individuales:

<img width="1354" height="343" alt="image" src="https://github.com/user-attachments/assets/0b283e14-80d9-4837-81bd-ebb2926328d5" />


<img width="1051" height="185" alt="image" src="https://github.com/user-attachments/assets/6806b945-7d21-4b96-8249-de58be63c63c" />


````main.java
@Component
public class OutboxEventReader implements CommandLineRunner {

    @Autowired
    private OutboxTableRepository outboxTableRepository;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private KafkaTemplate<String, ExampleEntity> kafkaTemplate;

    @Autowired
    private PersonsRepository personsRepository;

    @Autowired
    private OutboxEventReader eventReader;

    enum OutboxStatus {
        PENDING, SENT, FAILED,DONE
    }

    // Implement the run method to read the outbox table and send the events to
    // Kafka,EVERY 5 seconds
    @Override
    public void run(String... args) throws Exception {
        while (true) {
            List<OutboxEvent> events = outboxTableRepository
                    .findByStatusIn(List.of(OutboxStatus.PENDING.name(), OutboxStatus.FAILED.name()));

            for (OutboxEvent outboxEvent : events) {
                // Cada evento se procesa en su propia transacción
                // Si uno falla, el bucle continúa con el siguiente
                eventReader.processEvent(outboxEvent);
            }

            Thread.sleep(1200);
        }
    }

    @org.springframework.transaction.annotation.Transactional(propagation = Propagation.REQUIRES_NEW)
    public void processEvent(OutboxEvent outboxEvent) {
        try {
            ExampleEntity entity = objectMapper.readValue(outboxEvent.getPayload(), ExampleEntity.class);

            System.out.println("Procesando: " + entity);

            // Enviar a Kafka y esperar confirmación
               kafkaTemplate.send("outbox-ejemplo", entity).get(20, TimeUnit.SECONDS);


            // Marcar el evento original como DONE
            outboxEvent.setStatus(OutboxStatus.DONE.name());
            outboxTableRepository.save(outboxEvent);

            // Crear un nuevo registro de auditoría como SENT
            eventReader.saveOutbox(outboxEvent, entity);


            
        } catch (Exception e) {
            System.err.println("Error procesando evento ID " + outboxEvent.getId() + ": " + e.getMessage());

            } catch (Exception e) {
            System.err.println("Error procesando evento ID " + outboxEvent.getId() + ": " + e.getMessage());

             // catch anidado: si el save(FAILED) también falla, no rompe el for,
             //seguiria en pending si falla el segundo save
            try {
                outboxEvent.setStatus(OutboxStatus.FAILED.name());
                outboxTableRepository.save(outboxEvent);
            } catch (Exception dbException) {
                // BD caída — no podemos persistir, solo logueamos
                // El evento quedará como PENDING en BD cuando vuelva
                System.err.println("No se pudo marcar como FAILED (¿BD caída?): " + dbException.getMessage());
                // Aquí podrías mandar alerta, métricas, etc.
            }
        }
        }
    }

    @org.springframework.transaction.annotation.Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveOutbox(OutboxEvent outboxEvent,ExampleEntity entity){
    
            OutboxEvent sentEvent = OutboxEvent.builder()
                    .type("SentEventExample")
                    .payload(outboxEvent.getPayload())
                    .entityId(entity.getId())
                    .status(OutboxStatus.SENT.name())
                    .version(outboxEvent.getVersion() + 1)
                    .build();

            outboxTableRepository.save(sentEvent);
           // throw new RuntimeException("sdasdasdsad");

        
    }
}

````

Esta implementación actua en modo de que, si la transacción uno falla, en el catch lo guardará como failed, si falla el catch, el
row seguira como pending y no romperá el primer flujo. Si falla la segunda transacción, no estará insertando nada del SentEventExample.

Y así es como creas un message relay, usas un kafaTemplate.send(...).get para esperar la conexión de manera bloqueante, es lo
adecuado, con un time limit para que el hilo no quede colgante.

Y con estos escenarios, ya tienes un relay y ya suscribes un consumer a micro b, este lo procesa y cuando procese el pojo finalmente en su WRITE DATABASE, manda otro 
evento a la tabla outbox. Pudieras hacerlo con un worker o cache en segundo plano, para que sea asincrono y no te falle el método final
al confirmar todo el flujo eventual de tu outbox y del flujo predeterminado.

Y el version recuerda que va incremental, esto se ve por las pruevbas, mucho cuidado si en una columna del relay usas @Version

<img width="325" height="77" alt="image" src="https://github.com/user-attachments/assets/ea1ef391-1123-4ee4-a4d2-a9c9d0a2060c" />

Si usas esa anotación, te va a corromper tus columnas, procura mejor quitar esa anotación en relays.


La otra manera elegante de resolver esto es usando debezium, pero por ahora tenemos en cuenta esta implementación manual.

Prueba componente de relay a consumer exitosa: 

<img width="2524" height="818" alt="image" src="https://github.com/user-attachments/assets/b4d0764d-2d35-4a44-9433-b34e58252582" />

<img width="888" height="188" alt="image" src="https://github.com/user-attachments/assets/15fe55c1-eb73-441a-84d8-7c11a6698af2" />

<img width="760" height="1056" alt="image" src="https://github.com/user-attachments/assets/43dddb02-84aa-4f5b-b4bd-05686239c670" />


Prueba E2E:

<img width="1206" height="1009" alt="image" src="https://github.com/user-attachments/assets/90fc5a79-11d3-42c4-9db4-c3a69f1ef5b1" />

<img width="1976" height="1439" alt="image" src="https://github.com/user-attachments/assets/f22c24be-468e-4e5e-84d2-b81b6c4cacfc" />


Prueba exitosa. Estas pruebas las hago en local, me ahorro por ahora la molestia de meterlo a kube porque tengo otros temas que estudiar.
No confundan flojera con capacidad de favor.

## apache jmeter testing

Por último solo lanzaremos un par de peticiones para aprender a manejar esta herramienta.

Número de hilos: número de usuarios concurrentes.

Periodo de suposición: cuantos llegan por segundo.}

Contador de bucle cuantas veces los hilos deben repetir la operación

Asi como lo configuré se entiende que 100 usuarios llegaran en grupos de 5 segundos, repetirán la operación cada uno
10 veces. Cuando acabe, solo pasaran 500 ms para que repita la acción

<img width="748" height="325" alt="image" src="https://github.com/user-attachments/assets/f713715b-4154-4947-b780-8a5f760aca18" />


<img width="1513" height="382" alt="image" src="https://github.com/user-attachments/assets/abced9f9-a9b3-4324-9962-d334eb02cd86" />


Se puede observar que los registros son consecutivos y eventuales, porque kafka y el outbox aseguran el orden de 
la entrega de tu evento

<img width="1565" height="1053" alt="image" src="https://github.com/user-attachments/assets/0a12faab-6c3f-4828-a033-b4d583390135" />


Así terminamos esta práctica. Ahora sabemos orquestar sagas ¿El límite? nuestra imaginación.

