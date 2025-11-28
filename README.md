# Informe Final del Proyecto Melodía

Este documento sirve como informe final para el proyecto Melodía, una plataforma de streaming de música desarrollada con una arquitectura de microservicios moderna.

## Diagramas de Arquitectura

### Vista Conceptual (Diagrama de Arquitectura)

![Vista Conceptual (Diagrama de Arquitectura)](./images/diagrama.png)

### Vista Física (Diagrama de Railway)

![Vista Física (Diagrama de Railway)](./images/railway.png)

## Decisiones de Arquitectura

#### 1. Arquitectura de Microservicios

Optamos por separar la lógica en servicios independientes (`content-service`, `users-service`, `notification-service`) orquestados por un `gateway`.

- **Por qué:** Esto nos permite escalar cada componente de forma aislada, cada uno manejando su propia logica y sus propios datos, encapsulando la complejidad, con los servicios comunicandose entre si mediante APIs bien definidas (mediante OpenAPI). Además, nos permitió trabajar en paralelo en distintos dominios sin pisarnos el código.
- **Gateway:** Implementamos un API Gateway que actúa como único punto de entrada para los clientes (la App frontend). Este se encarga de la validación de sesión centralizada para cada request que lo atraviesa y el enrutamiento a los servicios correspondientes, simplificando la lógica en las aplicaciones cliente, donde se tiene un único entrypoint (tipado por OpenAPI) para interactuar todos los servicios de la plataforma de forma agnóstica.

#### 2. Selección del Stack Tecnológico

- **Hono:** Para los servicios backend, utilizamos Hono. Es un framework extremadamente ligero y rápido que soporta los estándares Web API. Su integración con TypeScript, Zod y OpenAPI para la validación de esquemas nos permitió definir contratos de API claros y seguros.
- **OpenAPI Fetch** Para la comunicacion entre servicios (tanto interna entre servicios como app <-> gateway) utilizamos OpenAPI Fetch, una libreria que permite hacer requests HTTP tipadas a partir de esquemas OpenAPI.
- **PostgreSQL & Prisma:** Para la persistencia de datos estructurados (usuarios, metadatos de canciones, playlists), confiamos en la robustez de PostgreSQL junto con Prisma como ORM. Prisma nos facilitó mantener un esquema de datos coherente y tipado end-to-end en conjunto con las definiciones de OpenAPI.
- **MongoDB (Notification Service):** Para las notificaciones, optamos por MongoDB debido a la naturaleza no estructurada.
- **Expo & React Native:** Para la aplicación móvil, Expo fue la elección natural para iterar rápido en Android debido a la previa experiencia del equipo en React.
- **Next.js (Dashboard):** Para el panel de administración, aprovechamos mantenernos en React al igual que la App, utilizando Next.js como framework fullstack, lo que nos permitió configurar un RPC (librería oRPC) para poder tener un ciclo completo desde los servicios, al backend del frontend, al frontend completamente typesafe con inferencia de los tipos desde el OpenAPI y Prisma.
- **Better Auth:** Utilizamos Better Auth para la autenticación y gestión de usuarios, tanto para la App frontend como para el Dashboard.

## Funcionalidades Incompletas y Errores Conocidos

A pesar de nuestros esfuerzos, existen áreas que quedaron pendientes o que podrían mejorarse en futuras iteraciones:

- **Sincronización Offline en Móvil:** La aplicación móvil reproduce música en streaming, pero la funcionalidad de descargar canciones para escucha offline ("Modo Avión") quedó marcada como una mejora futura, dado que fue elegido priorizar la funcionalidad de la app y recomendaciones sobre la sincronización offline.
- **Búsqueda Avanzada:** Actualmente, la búsqueda se realiza directamente sobre la base de datos. Para un catálogo masivo, sería ideal integrar un motor de búsqueda dedicado como Elasticsearch para manejar typos y relevancia de mejor manera.

## Problemas Encontrados y Lecciones Aprendidas

El camino no estuvo exento de desafíos. Aquí algunas de las lecciones más valiosas que nos llevamos:

- **La complejidad de lo distribuido:** Al principio subestimamos la complejidad de depurar errores que atraviesan múltiples servicios. Implementar un buen logging y tracing fue vital para no perdernos cuando una petición fallaba entre el Gateway y un microservicio.
- **Manejo de Estado en Móvil:** A pesar que React Native es muy similar a React web, tiene ciertas diferencias a la hora de manejar estados, como por ejemplo que las paginas no vuelven a renderizarse al cambiar de ruta como sucede en web.
