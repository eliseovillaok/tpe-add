# **Iteración 2**

## **ADR 002**

### **Titulo**

Autenticación y Modulo de Pagos para el Cliente

**Motivación**

Dado que la seguridad de los datos de los clientes y la integridad de las transacciones de pagos son aspectos críticos, es esencial que el diseño arquitectónico priorice estos atributos de calidad. Implementar patrones para la autenticación y autorización garantizará un control de acceso seguro y confiable, protegiendo los datos personales. 

Asimismo, la integración de un microservicio de pagos con la pasarela de Mercado Pago proporcionará un entorno de transacciones seguro, fortaleciendo la confianza del cliente y mejorando la reputación de la empresa. 

**Drivers elegidos:** Garantizar la seguridad de los datos del cliente.

**Justificación:** Este driver ha sido seleccionado debido a la criticidad de los módulos de 'Clientes' y 'Pagos', que contienen información altamente sensible y deben cumplir con estándares de seguridad estrictos para prevenir accesos no autorizados, brechas de datos y fraudes.

**Meta:** Diseñar una arquitectura de microservicios que optimice la seguridad de los datos personales de los clientes y las transacciones de pagos, garantizando un acceso controlado, autenticado y autorizado.

**Componente a refinar**: Módulo de Clientes y Autenticación

## **Decisión principal para Autenticación**

Centralizar la autenticación en un ***API Gateway.*** Los clientes deben autenticarse una sola vez y recibir un token JWT que se adjunta a sus solicitudes. Cada microservicio (como el de "Pedidos") simplemente valida el token, garantizando que la solicitud proviene de un cliente autenticado sin necesidad de realizar la autenticación completa nuevamente. Esto permite que los microservicios sean ligeros, seguros y más fáciles de mantener.

La autenticación y validación del token se hacen en el gateway antes de que la solicitud llegue a los microservicios backend. El gateway se convierte en el responsable de la seguridad y redirige las solicitudes a los microservicios adecuados después de verificar el token.

**Flujo**

- El cliente (usuario) accede a la aplicación web o móvil y realiza un inicio de sesión proporcionando sus credenciales (usuario y contraseña).
- La aplicación envía las credenciales al **servidor de autenticación** central (por ejemplo, un servidor de OAuth 2.0 o un servicio de autenticación implementado en el backend).
- El servidor de autenticación verifica las credenciales del cliente contra su base de datos de usuarios.
- Si las credenciales son correctas, el servidor emite un **token de acceso** (por ejemplo, un JWT) que incluye:
    - Información del usuario (por ejemplo, ID de usuario, nombre).
    - Roles y permisos del usuario.
    - Tiempo de expiración del token.
    - Una firma digital para garantizar la integridad del token.
- El servidor de autenticación envía este token de vuelta al cliente.
- A partir de este punto, el cliente incluye el token de acceso en la cabecera `Authorization` de cada solicitud HTTP que realiza a los microservicios.
- Cuando un microservicio, como el de **"Pedidos"**, recibe una solicitud con un token en la cabecera, utiliza una biblioteca o middleware para validar dicho token.
- Si el token es válido, el microservicio permite que la solicitud continúe y ejecuta la lógica de negocio (por ejemplo, procesar un pedido).

**Ventajas de utilizar API Gateway:**

- **Reducción de Complejidad:** Asegura que todos los microservicios reciban solicitudes ya autenticadas, sin tener que implementar lógica de autenticación propia en cada uno.
- **Seguridad Uniforme**: Todos los microservicios pueden confiar en que el cliente ya ha sido autenticado y solo necesitan validar el token. Esto reduce la superficie de ataque y asegura que las políticas de autenticación se gestionen de forma centralizada.
- **Modificabilidad:** Si se agregan nuevos microservicios al sistema, la lógica de validación ya estaría implementada y sólo se debería diseñar el sistema de validación de tokens en en dicho microservicio.
- **Escalabilidad**: La validación de tokens en cada microservicio es mucho más ligera y rápida que la autenticación completa, lo que ayuda a mantener la eficiencia. Además la API Gateway puede utilizarse en un futuro como balanceador de carga.

**Desventajas potenciales de la API Gateway:**

- **Dependencia en el Servidor de Autenticación**: Si el servidor de autenticación central falla o tiene problemas, los clientes no pueden obtener nuevos tokens. Sin embargo, los microservicios pueden seguir funcionando y validando tokens ya emitidos hasta que estos expiren.
- **Renovación de Tokens**: Es necesario gestionar de forma adecuada la expiración y renovación de tokens para mantener sesiones seguras y activas.

## ***Alternativas rechazadas***

### **Autenticación Basada en Microservicios Independientes**

Cada microservicio es responsable de su propia autenticación y validación de tokens. No hay un único punto de entrada; los microservicios operan de manera independiente, pero cada uno valida el token JWT al recibir solicitudes. 

**Ventajas:** No hay un solo punto de control, lo que puede aumentar la flexibilidad y escalabilidad.

**Desventajas:** Si el sistema crece y más microservicios se agregan, la administración de tokens se vuelve más compleja, ya que todos los microservicios deben estar alineados en cuanto a las políticas de autenticación y autorización.

- **Problemas de Sincronización:** Si no se maneja correctamente la **sincronización** de las claves públicas/privadas para la validación de los tokens, o si hay diferentes configuraciones entre microservicios, puede haber problemas de **inconsistencia** en la validación de tokens, lo que podría generar errores de autenticación.
    
    Por ejemplo, si un microservicio no está actualizado con las últimas claves para validar los tokens, puede rechazar solicitudes válidas o permitir solicitudes no autenticadas.
    
- **Repetición de Lógica de Autenticación:** La lógica de autenticación y validación de tokens (como la verificación de la firma del JWT, la comprobación de la expiración del token, y la verificación de los permisos o roles) debe ser implementada en cada microservicio que necesita autenticar usuarios. Esto **duplica** esfuerzos y puede llevar a la creación de código redundante en cada microservicio.

En conclusión si bien la centralización de la autenticación en una API Gateway trae problemas de flexibilidad, los mecanismos descentralizados son más complejos de mantener y duplican lógica. Por lo tanto esta opción quedó descartada y continuamos evaluando opciones de autenticación centralizada.

### **Autenticación basada en Proveedor de Identidad Externo (IdP)**

Este enfoque utiliza un servicio externo para gestionar la autenticación y autorización de los usuarios. Los microservicios no se encargan de la autenticación directamente sino que se delega a un sistema de gestión de identidades (IdP), como **OAuth 2.0**, **OpenID Connect (OIDC)**, o un proveedor como **Auth0**, **Okta**, o **Keycloak**.

**Flujo:** 

- El cliente se autentica con un **Proveedor de Identidad Externo** (IdP).
- El IdP valida las credenciales del cliente (por ejemplo, nombre de usuario y contraseña, autenticación multifactor) y si son correctas emite un **token de acceso.**
- Cada microservicio valida el **token de acceso** con el IdP para asegurarse de que es válido. Generalmente, esto se hace utilizando un **middleware** que se conecta al IdP o a un servidor de autorización centralizado.

**Ventajas:**  

- El IdP maneja la autenticación y la gestión de usuarios, lo que **descentraliza** la lógica de autenticación de los microservicios y elimina la necesidad de que cada uno valide credenciales o gestione el ciclo de vida de los usuarios.
- Los microservicios simplemente validan los tokens emitidos por el IdP, lo que reduce la carga de trabajo en los microservicios y mejora la seguridad al contar con un sistema especializado en la gestión de identidades.

**Desventajas:**

- **Dependencia del Proveedor de Identidad**: Si el **IdP** experimenta problemas, toda la autenticación en el sistema podría fallar, afectando a todos los microservicios que dependen de él.
- **Complejidad en la Integración**: Los microservicios deben estar preparados para comunicarse con el IdP y validar los tokens de acceso, lo que puede requerir un desarrollo adicional y un middleware específico para la validación de tokens.
- **Riesgos de Seguridad:** La **seguridad del IdP** es crucial. Si el IdP se ve comprometido, todos los microservicios y usuarios que dependen de él podrían estar en riesgo.

En conclusión, este mecanismo nos provee las mismas ventajas que un API Gateway pero agregándole desventajas como la dependencia de un proveedor externo. Por lo tanto, también se descartó.
