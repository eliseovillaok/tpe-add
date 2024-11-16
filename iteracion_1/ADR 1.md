# Iteración 1

# **ADR 001**

## **Titulo**

Migración de Sistema Monolítico de Aplicación de Alimentos a Microservicios.

## **Motivación**

La empresa de productos alimenticios tiene la intención de migrar su sistema monolítico existente a una arquitectura más flexible y evolutiva, alineándose con su objetivo de reducir la rigidez y mejorar la mantenibilidad. La lógica de negocio de la empresa consiste en módulos funcionales con diferentes grados de criticidad, y se pretende reemplazar el acceso al sistema existente por protocolos HTTP/REST. El requisito principal es garantizar una alta disponibilidad y seguridad para la funcionalidad de Clientes, mientras se mantiene la consistencia de datos en las dos bases de datos SQL.

La decisión de la empresa de adoptar el Patrón de Arquitectura de Microservicios está impulsada por la necesidad de escalabilidad, flexibilidad, mantenibilidad y resiliencia. Esta decisión tiene como objetivo abordar los requisitos del sistema, incluyendo la necesidad de alta disponibilidad y seguridad para la funcionalidad de Clientes, al tiempo que se garantiza la consistencia de datos en las dos bases de datos SQL.

## **Drivers**

**Drivers elegidos:** Garantizar la modularidad e interoperabilidad de las funciones principales del sistema.

**Justificación:** para comenzar el diseño del sistema debemos estructurar una arquitectura modular del sistema general que antes era de carácter monolítico. Esto nos permitirá en las siguientes iteraciones estructurar cada modulo para satisfacer sus escenarios de calidad específicos.

**Meta:** Descomponer y diseñar los microservicios de `Clientes`, `Pedidos`, y `Reparto y rutas`, asegurando una integración robusta entre ellos que permita la interoperabilidad continua y la preparación para futuras expansiones como `Pagos` y `Estadísticas`.

En este caso tenemos un sistema greenfield, ya que comenzamos el diseño desde cero para rediseñarlo con un enfoque modular, por lo tanto el componente a refinar es el sistema en sí.

## **Decisión principal**

Se adoptará el Patrón de Arquitectura de Microservicios para abordar los requisitos del sistema. Esta decisión se alinea con el objetivo de la empresa de reducir la rigidez y mejorar la mantenibilidad, al tiempo que garantiza una alta disponibilidad y seguridad para la funcionalidad de Clientes⁠.

La decisión asume que la empresa tiene un buen entendimiento del Patrón de Arquitectura de Microservicios y sus beneficios. También asume que la empresa tiene un plan claro para descomponer la aplicación en servicios, definir los límites de los servicios e implementar la comunicación entre ellos⁠.

**Ventajas de utilizar microservicios:**

- **Agilidad:** Los diferentes partes de un programa se pueden dividir en pequeños equipos de desarrollo especializados, aumentando así la productividad a la vez que se reduce el ciclo de desarrollo.
- **Escalables y flexibles:** Los microservicios se pueden escalar de forma independiente en caso de aumentar la demanda en cierta característica del software.
- **Libertad de desarrollo:** Cada microservicio puede desarrollarse con una tecnología o herramienta específica, sin necesidad de que las otras partes también las tengan que utilizar.
- **Consistencia:** cuando un software está dividido en muchas partes pequeñas, su consistencia y resistencia a los errores aumenta notablemente, al contrario de lo que ocurre en una arquitectura monolítica.

**Desventajas de utilizar microservicios:**

- **Alto consumo de memoria:** al tener cada microservicio sus propios recursos y bases de datos, consumen más memoria y CPU.
- **Complejidad en la gestión:** si contamos con un gran número de microservicios, será más complicado controlar la gestión e integración de los mismos.
- **Dificultad en la realización de pruebas:** debido a que los componentes de la aplicación están distribuidos, las pruebas y test globales son más complicados de realizar.
- **Coste de implantación alto:** una arquitectura de microservicios puede suponer un alto coste de implantación debido a costes de infraestructura y pruebas distribuidas.

**Diagrama de la estructura de microservicios:**

Este diagrama representa la vista de componentes de la arquitectura basada en microservicios. Los módulos principales (Clientes, Pedidos, y Repartos) están diseñados para operar de manera independiente, comunicándose a través de servicios REST. La modularidad de la arquitectura permite escalabilidad horizontal, mientras que cada microservicio mantiene su propia base de datos para garantizar independencia y encapsulación.

![image.png](images/image.png)

*Descripción:*

Los módulos principales del sistema son:

- *Clientes:* microservicio encargado de la gestión de los datos de los clientes, junto con el sistema de pago de los pedidos. Posee su propia base de datos relacional donde se almacena dicha información.
- *Pedidos:* microservicio encargado de la gestión y almacenamiento de los pedidos realizados. Cuando se realiza un pedido, el modulo de pedidos se lo informa al modulo de clientes, solicitando la información necesaria del cliente para la autenticación y el almacenamiento del pedido en su base de datos.
- *Repartos y rutas:* microservicio encargado de intercambiar información con el modulo de Pedidos para gestionar los repartos de los mismos. Almacenando los datos de los repartidores y asignándoles los pedidos que les correspondan.

## *Alternativas rechazadas*

*Repartos y rutas acoplado con Pedidos*

Los servicios de “Repartos y rutas” y “Pedidos” se encuentran conectados a la misma base de datos para poder acceder a los pedidos y mantener la consistencia de los repartos de manera mas simple.

Ésta alternativa *fue rechazada* porque no cumple con el criterio de no acoplamiento en microservicios, ya que ambos se encuentran acoplados por la misma base de datos, por lo tanto una falla de uno puede producir una falla en el otro.

![image.png](images/image%201.png)

*Servicios con BD Compartida*

Los servicios del sistema se encuentran conectados a la misma base de datos, para aumentar la performance evitando agregaciones o ensamblados de distintas bases de datos.

Ésta alternativa *fue rechazada* porque introduce acoplamiento entre servicios ademas de un overhead en la base de datos que puede producir fallos al aumentar la cantidad de solicitudes por unidad de tiempo

![image.png](images/image%202.png)


