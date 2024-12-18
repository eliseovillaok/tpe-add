# **ADR 003**

## **Titulo**

Escalabilidad del Modulo de Pedidos

**Motivación**

La motivación para la escalabilidad en el módulo de **pedidos** radica en garantizar que el sistema pueda manejar un volumen creciente de solicitudes sin sacrificar rendimiento ni experiencia de usuario. Un diseño escalable permite responder eficazmente a picos de carga, mantener alta disponibilidad, y adaptarse al crecimiento del negocio. Esto asegura una experiencia confiable, evita cuellos de botella, y optimiza el rendimiento general del sistema, permitiendo la gestión de transacciones críticas y el manejo de concurrencia con consistencia.

**Drivers elegidos:** Escalabilidad y modularización de Pedidos.

**Justificación:** Este driver ha sido seleccionado debido a que existen 3 etapas de Pedidos que pueden explotarse para obetener escalabilidad horizontal y una comunicación entre ellos que garantice eficiencia al momento de que el cliente realice un pedido, sin perder sincronización.

**Meta:** Diseñar una arquitectura de microservicios que permita al sistema manejar un gran volumen de datos y transacciones incluso en momentos de alta demanda, sin perder rendimiento y manteniendo la usabilidad.

**Componente a refinar**: Módulo de Pedidos.

## **Decisión principal**

### **Cadena de Responsabilidades + Balanceador de Carga**

### **Cadena de Responsabilidades**

Aplicado al módulo de **Pedidos**, la cadena de responsabilidades implica que cada fase del proceso de un pedido (preprocesado, autorización y aceptación) se gestione por un microservicio específico. Cada microservicio actúa como un eslabón en la cadena, pasando la tarea al siguiente solo cuando se completa la fase actual.

**Desacoplamiento y Escalabilidad**:

- Cada microservicio es independiente y puede escalar de forma autónoma en función de la carga de trabajo específica de su fase. Por ejemplo, si la fase de *autorización* requiere más recursos debido a un mayor volumen de solicitudes, solo este microservicio necesita escalarse, no toda la cadena.
- Esto permite una mejor gestión de los recursos y la posibilidad de ajustar la infraestructura según las necesidades de cada etapa.

**Comunicación entre los Microservicios**

Se optó por el uso de un sistema de sincronización Message Queue implementado con Kafka para la comunicación entre las etapas de procesamiento de un pedido para una solución escalable y robusta. 

Este mecanismo se utiliza principalmente para construir sistemas de mensajería que permiten la creación y el manejo de flujos de datos en tiempo real de forma eficiente, segura y escalable.

**Características de Kafka**

- Permite que múltiples productores publiquen mensajes en **tópicos** y que múltiples consumidores se suscriban a esos tópicos para recibir los mensajes en tiempo real.
- Almacena mensajes en disco y permite la persistencia duradera de estos, lo que permite a los consumidores leer mensajes en diferentes momentos y re-procesar flujos de datos si es necesario.
- Está diseñado para escalar horizontalmente, lo que significa que se pueden añadir más servidores (nodos) a un clúster para aumentar la capacidad de procesamiento y almacenamiento.
- Maneja millones de mensajes por segundo con baja latencia, lo que lo hace ideal para gestionar pedidos en tiempo real..

**Persistencia**

Para desacoplar las etapas y que aumentar la escalabilidad, decidimos que cada microservicio tenga su propia base de datos.

**Base de Datos Microservicio de Pedidos**:

- La base de datos asociada a este microservicio gestionaría la información básica del pedido, como el ID del pedido, detalles del cliente, productos solicitados, estado del pedido, etc.

**Base de Datos Microservicio de Preprocesado**:

- Este servicio tiene una base de datos para almacenar los resultados del preprocesado, como la disponibilidad del stock, precios calculados, y otras verificaciones previas.

**Base de Datos Microservicio de Autorización**:

- Tiene una base de datos para almacenar la información relacionada con la autorización de pagos o el historial de autorizaciones para un pedido.

**Base de Datos Microservicio de Aceptación**:

- Este servicio almacena el estado final del pedido, si el pedido fue aceptado, la fecha de aceptación, etc.

**Flujo de Ejecución de un Pedido**

1. **Selección del Pedido**
    - El sistema principal de Pedidos gestiona el carrito de compras del cliente. Cuando el cliente confirma que quiere realizar la compra, se delega el pedido al microservicio de Preprocesamiento.
2. **Publicación del Pedido**:
    - El microservicio de **Preprocesado** recibe un nuevo pedido y valida sus datos. Una vez validado, publica un mensaje `PedidoPreprocesado` en el tópico `preprocesado-pedidos`.
3. **Consumo por la Etapa de Autorización**:
    - El microservicio de **Autorización** está suscrito al tópico `preprocesado-pedidos` y consume el mensaje `PedidoPreprocesado`.
    - Procesa la autorización (verifica la identidad del cliente y los intentos permitidos). Si la autorización es exitosa, publica un mensaje `PedidoAutorizado` en el tópico `autorizacion-pedidos`.
4. **Consumo por la Etapa de Aceptación**:
    - El microservicio de **Aceptación** está suscrito al tópico `autorizacion-pedidos` y consume el mensaje `PedidoAutorizado`.
    - Procesa la aceptación del pedido y, una vez completada la fase, publica un mensaje `PedidoAceptado` en el tópico `aceptacion-pedidos`.
5. **Notificación Final y Persistencia**:
    - El modulo de pago estará suscrito al tópico `aceptacion-pedidos` para concretar el pago de los pedidos que ya pasaron por las tres etapas.

Cada etapa actúa como **consumidor** del tópico anterior y **productor** del tópico siguiente, asegurando un flujo ordenado y desacoplado.  

### **Balanceador de Carga**

Entre el API Gateway y el módulo Pedidos se decidió incorporar un balanceador de carga que pueda distribuir mejor las transacciones solicitadas a través del sistema. Esto incide en la funcionalidad que se quiere atacar en la presente iteración ya que el API Gateway es el módulo que autentica a un consumidor antes de que éste pueda realizar un pedido en el módulo Pedidos.

Para llevar a cabo esta implementación se deberá:

- Definir la estrategia de balanceo de carga (round robin, least connections, etc.) que mejor se adapte a las necesidades del módulo de Pedidos.
- Configurar las reglas de escalabilidad automática que ajusten la cantidad de instancias de cada microservicio en función de la carga de trabajo.
- Establecer dentro del balanceador una funcionalidad de monitoreo para supervisar el estado  de cargas y de los microservicios del sistema. Es necesario asegurarse de tener alertas configuradas para cualquier problema de rendimiento o disponibilidad.

**Ventajas**

- Monitoreo de la carga del sistema para prevención y corrección de sobrecargas que saturen el modulo de Pedidos.
- Aporta disponibilidad y escalabilidad.
- Mejora la eficiencia de las transacciones.

**Desventajas**

- Único punto de fallo. Si el balanceador falla, toda la interacción del cliente con el sistema quedaría inactiva.
- Para poder mantener la consistencia del sistema en todo momento, para lo cuál va a hacer falta que la actualización del balanceador, cuándo se agrega o elimina un servicio sea instantánea.

![image.jpg](images/image.jpg)

| Microservicio | Responsabilidad |
| --- | --- |
| API Gateway | La API gateway valida el perfil de los usuarios y permite al usuario acceder al servicio de Pedidos |
| Pedidos | Se encarga de administrar los productos seleccionados por el usuario para la compra y de organizar la preparación de la orden. |
| Clientes | Gestiona la información de los clientes y se encarga de responder a la API Gateway para realizar la validación del usuario asociado a un cliente. |
| Pagos | Funciona como un intermediario que conecta y facilita la integración con una pasarela externa de pagos al recibir el mensaje PedidoAceptado para realizar la operación de pago. |
| PedidoPreprocesado | Recibe el pedido del sistema principal, valida los datos del pedido y publica un mensaje PedidoPreprocesado en el tópico preprocesado-pedidos. |
| PedidoAutorizado | Consume el mensaje PedidoPreprocesado, verifica la identidad del cliente y los intentos permitidos. Si la autorización es exitosa, publica un mensaje PedidoAutorizado en el tópico autorizacion-pedidos. |
| PedidoAceptado | Consume el mensaje PedidoAutorizado, procesa la aceptación del pedido y, al finalizar, publica un mensaje PedidoAceptado en el tópico aceptacion-pedidos. |

## ***Alternativas rechazadas***

### **Patrón de CQRS (Command Query Responsibility Segregation)**

En CQRS, las operaciones de **lectura** y **escritura** de pedidos están separadas en dos modelos diferentes, optimizando la consulta y el procesamiento de pedidos. Cada microservicio manejaría sus propias operaciones de escritura y lectura, sin necesidad de esperar confirmaciones de otros microservicios en el flujo de un pedido.

**Flujo**: El microservicio de preprocesado realiza la escritura de datos cuando un pedido se crea y publica los resultados de esa acción en un repositorio de comandos. Los microservicios de autorización y aceptación pueden consultar ese repositorio para tomar decisiones.

**Ventajas**:

- **Optimización de consultas**: Las consultas y las escrituras pueden estar optimizadas por separado, lo que mejora el rendimiento.
- **Escalabilidad**: Los microservicios de lectura y escritura pueden escalar de manera independiente.

**Desventajas**:

- **Complejidad**: Mantener dos modelos separados para comandos y consultas puede ser difícil de gestionar y coordinar.
- **Consistencia eventual**: Puede haber un retraso entre las escrituras y las consultas, lo que puede causar problemas de sincronización.

Esta alternativa fue rechazada debido a que puede traer problemas de sincronización de las etapas de un pedido y no ofrece mayores ventajas que la alternativa principal.

### Balancedor de Carga con Service Discovery

Se evaluó incorporar la lógica del patrón de microservicios Server Discovery para el balanceador, proveyendo un servicio en el cuál los módulos no deben estar preguntando sus direcciones en todo momento, y se facilitara la comunicación entre los servicios. 

La ventaja principal de esta variante es que el cliente no necesita desarrollar la lógica para el descubrimiento de servicios ni para el balanceo de cargas, ya que simplemente debe ejecutar una única URL y puede despreocuparse del resto. No obstante, hay desventajas, como la complejidad que introduce al sistema y a la implementación del mismo.

Esta alternativa se rechazó, eligiendo la implementación seleccionada de Event Broker con Kafka, que nos provee la comunicación independiente entre microservicios de forma más simple. Agregar el Service Discovery sería incorporar otra forma de que los servicios se comuniquen entre sí de manera innecesaria. 

De esta forma, el balanceador de carga no presentaría otro patrón para la comunicar los microservicios entre sí.

## **Consecuencias**

Las decisiones descriptas anteriormente atacan la escalabilidad de la funcionalidad de Pedidos de varias maneras. Primero se agura que al usar el patrón de cadena de responsabilidades la comunicación entre los servicios no sea un limitante en el procesamiento de las transacciones y luego se toma un balanceador de carga para poder distribuir esas transacciones a través del sistema. Si bien existen los trade off de atacar con tácticas el atributo de calidad, creemos que se logra un balance positivo con las decisiones tomadas, por que su impacto negativo en los demás atributos no es tan grande cómo el beneficio que traen para la escalabilidad del sistema en una función semi crítica. 

Los desafíos a tener en cuenta tiene que ver con los trade off que relacionan los distintos atributos de calidad:

- Disponibilidad: tanto para el balanceador de carga, cómo para la cadena de responsabilidades, será importante que que se pueda recuperar ante cualquier error de conexión para no perder solicitudes y transacciones en el tránsito del sistema.
- Testeabilidad: la complejidad del diseño puede inducir a que las pruebas de rendimientos se hagan cada vez más complejas, sobre todo aquellas que prueban la integración de los módulos.
- Desplegabilidad: a su vez, el incremento de la complejidad en el sistema impactará la facilidad del mismo en ser desplegado, lo cuál requerirá que todos los módulos, no sólo puedan ser probados de forma individual e integrada, si no que sean gestionados de forma correcta.
- Rendimiento: se deberá tener en cuenta que tanto el balanceador de carga cómo la cadena de responsabilidades no impacten negativamente en el rendimiento de los módulos, tanto en el uso de procesador cómo de memoria, por demás del impacto beneficioso que tiene en la eficiencia de la comunicación
