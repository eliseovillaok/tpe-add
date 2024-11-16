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


