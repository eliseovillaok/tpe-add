# Enunciado y Drivers

## Descripción del Sistema a Diseñar

---

Una **compañía de productos alimenticios** pretende **migrar su sistema** existente, que posee una
arquitectura de naturaleza **monolítica**, hacia una arquitectura basada en microservicios, de
manera que la nueva arquitectura sea menos rígida y sea más fácil de evolucionar.
La arquitectura existente cuenta con una **parte de clientes PC y móvil** que acceden a los datos
de la empresa alojados en **2 Bases de datos SQL (Clientes, Pedidos)**. La base de Clientes
contiene datos de clientes y pagos mientras que la base de Pedidos se encarga de almacenar
los datos restantes. El acceso actual se pretende que sea sustituido por protocolos HTTP/REST
que faciliten el acceso desde los clientes PC y móvil.
La lógica de negocio de la empresa cuenta con los siguientes módulos funcionales, con
diferentes grados de criticidad.
• **Clientes (crítico):** Esta funcionalidad permite a **acceder los datos de personales de los
clientes.**
• **Pedidos (semi-crítico):** Esta funcionalidad permite a los clientes **realizar pedidos** de los
productos a la empresa. 
Si un cliente intenta realizar un pedido se le permite un número máximo de 3 intentos. 
Las prestaciones y escalabilidad del sistema van a depender del
número de pedidos por hora. **El sistema de pedidos pasa siempre por una cadena de tres
fases, a saber: preprocesado del pedido, autorización y aceptación,** de manera que no es
posible pasar a un estado si no se ha procesado correctamente el estado anterior.
• **Reparto y rutas (crítico):** Este componente complejo cuenta con una funcionalidad que es
necesario desacoplar y gestiona el reparto de las **flotas de transporte a los clientes** y las
**rutas de los camiones**. La gestión cuenta con 2 algoritmos de optimización que se
seleccionan en función de la demora del camión.
• **Estadísticas (no crítico):** Esta funcionalidad proporciona información valiosa sobre el estado
de los pedidos y la situación en tiempo real de los camiones. Las estadísticas proporcionan
también información de clientes.
• **Incidencias (semi-crítico):** Esta funcionalidad reporta a los gestores de las rutas cualquier
tipo de incidencia (por ej., camión averiado, reparto no realizado, etc.)
• **Pagos (crítico)**: Esta funcionalidad se apoya en una pasarela de pago externa que
proporciona la empresa MercadoLibre para garantizar la seguridad de los pagos y la
compatibilidad con otros clientes.
Asimismo, la gestión de los pedidos debe proporcionar una forma de gestionar todas las
peticiones de los pedidos de clientes y gestionar las incidencias en el reparto.
La nueva arquitectura debe contar con los elementos software y/o tecnología adecuados para
poder ejecutar los microservicios.

## **Requerimientos Funcionales**

- **Gestión de clientes**: CRUD (crear, leer, actualizar, eliminar) de datos de clientes.
- **Gestión de pedidos**: Proceso de pedidos con limitación de intentos y secuencias de fases.
- **Gestión de reparto y rutas**: Planificación de rutas y manejo de la flota de transporte.
- **Manejo de incidencias**: Reportes de incidencias en la entrega de pedidos.
- **Procesamiento de pagos**: Integración con pasarela de pago externa.
- **Visualización de estadísticas**: Presentación de datos en tiempo real.

### User Stories

- Como **Admin** quiero acceder a los datos personales de los clientes para corroborar su veracidad
- Como **Admin** quiero ver las estadisticas (estado de pedidos, situacion actual de camiones e info. de clientes) para mejorar continuamente el servicio
- Como **Cliente** quiero realizar un pedido para consumir el producto alimenticio
- Como **Cliente** quiero realizar el pago de un pedido para que se confirme mi compra
- Como **gestor de rutas** quiero gestionar las rutas para obtener el mejor recorrido segun el momento del dia
- Como **gestor de rutas** quiero gestionar los repartos para que cada cliente obtenga su producto en tiempo y forma
- Como **gestor de rutas** quiero ver los incidentes de rutas y repartos para solucionarlos lo mas rapido posible.
- Como **repartidor de pedidos** quiero ingresar las incidencias de cada reparto **para** que se seolucionen rapidamente
- Como **repartidor de pedidos** quiero acceder a algoritmos de optimizacon de rutas **par﻿a** entregar los pedidos con mas rapidez

### Restricciones

- Si un cliente intenta realizar un pedido se le permite un número máximo de 3 intentos.
- El sistema de pedidos pasa siempre por una cadena de tres fases, a saber: preprocesado del pedido, autorización y aceptación**,** de manera que no es posible pasar a un estado si no se ha procesado correctamente el estado anterior.
- La gestión de rutas y repartos cuenta con 2 algoritmos de optimización que se seleccionan en función de la demora del camión.
- Apoyarse en una pasarela de pago externa que proporciona la empresa MercadoLibre

### **Atributos de Calidad**

- **Escalabilidad**: Especialmente en el módulo de pedidos, donde el sistema debe soportar un gran número de pedidos por hora.
- **Disponibilidad**: Alta disponibilidad para módulos críticos como `Clientes`, `Reparto y rutas`, y `Pagos`.
- **Seguridad**: Protección de datos personales y pagos, asegurada por la integración con una pasarela externa (MercadoLibre).
- **Modularidad**: Capacidad de implementar, mantener y actualizar cada microservicio de forma independiente.
- **Interoperabilidad**: Uso de protocolos HTTP/REST para garantizar la interoperabilidad con clientes móviles y PC.

### Escenarios de Atributos de Calidad

- Escalabilidad
    
    
    | Fuente del estimulo | Crecimiento orgánico |
    | --- | --- |
    | Estimulo | Aumento de solicitudes simultáneas |
    | Artefacto | Infraestructura de pedidos |
    | Ambiente | Despliegue local |
    | Respuesta | Manejo del incremento de solicitudes sin perder rendimiento |
    | Medida de la respuesta | 10000 peticiones por unidad de tiempo (segundos) |
- Disponibilidad
    
    
    | Fuente del estimulo | Petición externa |
    | --- | --- |
    | Estimulo | Respuesta nula de las base de datos |
    | Artefacto | Sistema |
    | Ambiente | Estado normal |
    | Respuesta | Detectar, aislar y recuperarse del fallo |
    | Medida de la respuesta | 100ms para recuperarse y devolver respuesta |
- Seguridad
    
    
    | Fuente del estimulo | Atacante |
    | --- | --- |
    | Estimulo | Acceder a la base de datos de clientes |
    | Artefacto | Modulo de clientes |
    | Ambiente | Estado online |
    | Respuesta | Verificar identidades |
    | Medida de la respuesta | Solo datos de publico conocimiento deberían quedar vulnerables ante un acceso indeseado. |
- Modularidad
    
    
    | Fuente del estimulo | Desarrolladores |
    | --- | --- |
    | Estimulo | Agregar nuevos pagos |
    | Artefacto | Sistema |
    | Ambiente | Diseño y desarrollo |
    | Respuesta | Nuevos pagos agregados, testeados y desplegados |
    | Medida de la respuesta | 1 semana para completarse |
- Interoperabilidad
    
    
    | Fuente del estimulo | Sistema externo de MercadoPago |
    | --- | --- |
    | Estimulo | Solicitud para autorizar y validar compra |
    | Artefacto | Sistema y MercadoPago |
    | Ambiente | Sistemas conocidos antes del runtime |
    | Respuesta | Si el pago es autorizado, el sistema valida la compra del cliente, sino, informa pago inválido y ofrece reintentarlo. |
    | Medida de la respuesta | 100% solicitudes intercambiadas correctamente |