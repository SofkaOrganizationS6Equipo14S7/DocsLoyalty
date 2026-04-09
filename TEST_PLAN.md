# Plan de Pruebas — LOYALTY

## 1. Identificación del Plan

| Campo | Detalle |
|---|---|
| **Nombre del proyecto** | LOYALTY — Calculadora de Descuentos Dinámicos |
| **Sistema bajo prueba** | Plataforma LOYALTY: Motor de descuentos (API REST S2S) + Dashboard de configuración |
| **Versión** | MVP v1.0 |
| **Fecha** | 9 de Abril de 2026 |
| **Equipo** | Agustín Mites, Matias Regaló, Luis Ospino |

---

## 2. Contexto

### ¿Qué sistema se va a probar?

LOYALTY es una plataforma de descuentos dinámicos compuesta por dos módulos principales:

- **Motor de descuentos (API REST Stateless S2S):** recibe desde el backend del e-commerce un payload con el carrito de compras y las métricas del comprador, valida la autenticación mediante API Key y devuelve en tiempo real el descuento combinado o precio personalizado calculado.
- **Dashboard de configuración UI:** interfaz web para administradores del e-commerce que permite gestionar usuarios por ecommerce según roles (SUPER_ADMIN, STORE_ADMIN, USER), registrar y administrar ecommerces (onboarding y ciclo de vida), definir y activar/desactivar reglas de descuento (por temporada, fidelidad y tipo de producto), establecer topes máximos y prioridades de descuento, gestionar API Keys (generar, rotar, revocar), y consultar el historial de descuentos aplicados y los logs de auditoría de cambios de reglas.

### ¿Qué problema de negocio resuelve?

Las tiendas de e-commerce B2C dependen actualmente de descuentos generales, estáticos y administrados de forma manual, lo que genera ineficiencias operativas y financieras. Este modelo aplica el mismo descuento a compradores recurrentes y a compradores de ofertas puntuales, lo que influye en los márgenes de ganancia, aumentando el costo de adquisición y fomentando que los clientes solo compren en épocas de rebaja.

LOYALTY resuelve este problema al introducir una capa de inteligencia entre el catálogo y el checkout, capaz de calcular e inyectar precios personalizados en tiempo real según la temporada, la fidelidad del cliente y el tipo de producto. Esto permite a los comercios proteger sus márgenes, aumentar la tasa de conversión y mejorar la retención de clientes frecuentes sin sacrificar rentabilidad.

---

## 3. Alcance de las Pruebas

El alcance de las pruebas para este ciclo cubre la totalidad de las historias de usuario **HU-01 a HU-16**. Los escenarios de prueba de cada HU se encuentran en `TEST_CASES.md`.

| HU | Título | Módulo |
|---|---|---|
| HU-01 | Inicio y cierre de sesión (Auth Management) | Dashboard — loyalty-admin |
| HU-02 | Gestión de Usuarios por Ecommerce | Dashboard — loyalty-admin |
| HU-03 | Administra su Ecommerce (STORE_ADMIN) | Dashboard — loyalty-admin |
| HU-04 | Usuario Estándar | Dashboard — loyalty-admin |
| HU-05 | Gestión y Validación de API Keys | Dashboard + Motor (S2S) |
| HU-06 | Gestión de Reglas de Temporada | Dashboard — loyalty-admin |
| HU-07 | Gestión de Reglas por Tipo de Producto | Dashboard — loyalty-admin |
| HU-08 | Rangos de Clasificación de Fidelidad | Dashboard — loyalty-admin |
| HU-09 | Límite y Prioridad de Descuentos | Dashboard + Motor (S2S) |
| HU-10 | Clasificación Dinámica de Clientes (Loyalty Tiers) | Motor — loyalty-engine |
| HU-11 | Cálculo de Carrito con Descuentos Aplicados | Motor — loyalty-engine |
| HU-12 | Historial de Descuentos Aplicados | Dashboard — loyalty-admin |
| HU-13 | Trazabilidad de Cambios de Reglas (Auditoría) | Dashboard — loyalty-admin |
| HU-14 | Activar/Desactivar Reglas | Dashboard — loyalty-admin |
| HU-15 | Registro de Ecommerces (Onboarding) | Dashboard — loyalty-admin |
| HU-16 | Acceso Total (Super Admin) | Dashboard — loyalty-admin |

### Fuera del Alcance

No existen historias de usuario excluidas de este ciclo. Quedan explícitamente fuera del alcance:

- Pruebas de infraestructura o configuración de red interna
- Pruebas de usabilidad o accesibilidad del Dashboard
- Cualquier funcionalidad no descrita en HU-01 a HU-16

---

## 4. Estrategia de Pruebas

Se ejecutarán los escenarios definidos para cada historia de usuario de acuerdo a sus criterios de aceptación. Si se detectan desviaciones respecto al resultado esperado, se reportará el defecto en un issue para que el equipo de desarrollo lo solucione y pueda ser re-validado en el siguiente sprint.

| Tipo de Prueba | HU Cubiertas | Objetivo | Herramienta |
|---|---|---|---|
| **Unitarias — loyalty-admin y loyalty-engine** | HU-01 a HU-16 | Verificar la lógica de negocio aislada: cálculos de descuento, validaciones de reglas, permisos por rol, manejo de tokens JWT y transformaciones de datos. Responsabilidad de DEV; QA valida que el reporte de cobertura supere el umbral antes de iniciar la ejecución funcional. | JUnit 5 + JaCoCo |
| **Funcional — API REST** | HU-01, HU-02, HU-03, HU-04, HU-05, HU-06, HU-09, HU-10, HU-11, HU-14, HU-15, HU-16 | Verificar que los endpoints REST del Motor de Descuentos y del Dashboard de configuración respondan correctamente en escenarios positivos y negativos, cumpliendo los criterios de aceptación definidos. | Karate |
| **E2E — Dashboard UI** | HU-01, HU-03, HU-04, HU-05, HU-09, HU-12, HU-13, HU-16 | Verificar los flujos E2E críticos del Dashboard que requieren interacción real con el navegador: login/logout, restricciones visuales por rol, enmascaramiento de API Keys (****XXXX), configuración de topes, historial de descuentos y panel de auditoría. | SerenityBDD + Cucumber |
| **Rendimiento — Motor de Descuentos** | HU-11 | Verificar que el Motor de Descuentos soporte la carga esperada de peticiones simultáneas dentro de los umbrales de tiempo de respuesta aceptables para el negocio. | k6 |
| **Regresión automatizada — CI/CD** | HU-01 a HU-16 | Ejecutar automáticamente las suites de Karate y SerenityBDD+Cucumber ante cada PR o merge a la rama QA, garantizando que no se introduzcan regresiones. | GitHub Actions |

---

## 5. Criterios de Entrada y Salida

### 5.1 Criterios de Entrada

| # | Criterio |
|---|---|
| 1 | Código fuente disponible en el repositorio con PR aprobado |
| 2 | Ambiente de pruebas desplegado y estable |
| 3 | Base de datos de pruebas configurada y accesible |
| 4 | Endpoints de la API documentados y disponibles |
| 5 | Pruebas unitarias ejecutadas por DEV con cobertura > 80% |
| 6 | Historias de usuario y criterios de aceptación revisados y aprobados |
| 7 | Datos de prueba preparados |
| 8 | Herramientas de prueba configuradas (JUnit 5, JaCoCo, Karate, SerenityBDD, Cucumber, k6, Maven, Docker, GitHub Actions, GitHub Issues, IntelliJ IDEA) |

### 5.2 Criterios de Salida

| # | Criterio |
|---|---|
| 1 | Todos los criterios de aceptación validados (100% ejecutados) |
| 2 | Tasa de aprobación de casos de prueba ≥ 95% |
| 3 | Sin defectos críticos o bloqueantes abiertos |
| 4 | Defectos de severidad media resueltos o con plan de mitigación aprobado |
| 5 | Reporte de SerenityBDD generado y revisado (aplica a HU-01, HU-03, HU-04, HU-05, HU-09, HU-12, HU-13, HU-16) |
| 6 | Reporte de Karate generado y revisado (aplica a HU-01, HU-02, HU-03, HU-04, HU-05, HU-06, HU-09, HU-10, HU-11, HU-14, HU-15, HU-16) |
| 7 | Pruebas de rendimiento k6 ejecutadas y dentro de umbrales aceptables: p95 ≤ 500 ms y tasa de error < 1% bajo carga concurrente (aplica a HU-11) |
| 8 | Reporte de cobertura JaCoCo generado por DEV, revisado y aprobado con resultado ≥ 80% de cobertura de línea en todos los módulos del sistema (aplica a HU-01 a HU-16) |

## 6. Entorno de Pruebas

Entorno dedicado de QA, aislado de producción, con datos controlados y sin impacto en clientes reales.

| Componente | Versión | Acceso / Uso |
|---|---|---|
| Motor de Descuentos — loyalty-engine (API REST S2S) | Docker 26.x | `http://localhost:8082` · HU-05, HU-09, HU-10, HU-11 · auth header `x-api-key` |
| Dashboard de configuración — loyalty-admin (UI) | Docker 26.x | `http://localhost:8081` · HU-01 a HU-09, HU-12, HU-13, HU-14, HU-15, HU-16 · cuenta Super Admin activa; sin MFA |
| Base de datos Admin — `loyalty_admin` | PostgreSQL 15-alpine · puerto 5432 | Usada por loyalty-admin · usuario `loyalty` · inicializada con datos de prueba |
| Base de datos Engine — `loyalty_engine` | PostgreSQL 15-alpine · puerto 5433 | Usada por loyalty-engine · usuario `loyalty` · inicializada con datos de prueba |
| Message Broker — RabbitMQ | 3-management-alpine · puertos 5672 (AMQP) / 15672 (UI) | Sincronización de reglas, API Keys y estados entre loyalty-admin y loyalty-engine · usuario `guest` |
| Windows 11 / Ubuntu (runner CI) | — | Local: ejecución manual y automatizada · CI: runner `ubuntu-latest` en GitHub Actions |
| JDK | 17 LTS (OpenJDK) | Runtime de Maven, Karate y SerenityBDD |
| Maven | 3.9.x | Ejecución de suites Karate y SerenityBDD+Cucumber |
| k6 | 0.50.x | Pruebas de rendimiento (HU-11) |
| Google Chrome | Estable actual | Pruebas UI del Dashboard |
| GitHub Actions | — | Trigger: PR o merge a rama `qa` · publica reportes Karate y SerenityBDD como artefactos |

### Datos de prueba

- Ecommerce de prueba con ID fijo
- Dos API Keys: una activa y una revocada
- Usuarios con roles SUPERADMIN, ADMIN, STORE_ADMIN y USER con credenciales conocidas por el equipo QA
- Reglas de temporada, producto y rangos de fidelidad que activan todos los escenarios
- Tope máximo positivo y prioridades únicas configuradas
- Logs de transacciones de los últimos 7 días precargados para HU-12
- Registros de auditoría precargados para HU-13

> **Prerequisito de arranque:** Se verificará `GET http://localhost:8082/health → HTTP 200` en el Motor de Descuentos, que `http://localhost:8081` es accesible desde el navegador, y que RabbitMQ responde en `http://localhost:15672` (management UI). Si alguno de los tres falla, la ejecución queda bloqueada.

---

## 7. Herramientas

| Herramienta | Versión | Categoría | Propósito específico |
|---|---|---|---|
| **JUnit 5** | 5.10.x | Pruebas unitarias | Framework de pruebas unitarias. Utilizado por DEV para verificar la lógica de negocio aislada: validaciones de reglas, cálculos de descuento, control de acceso por rol, manejo de tokens JWT y transformaciones de DTOs en los módulos loyalty-admin y loyalty-engine. |
| **JaCoCo** | 0.8.x | Cobertura de código | Plugin de cobertura de código integrado con Maven. Genera el reporte de cobertura de línea y rama de las pruebas unitarias JUnit 5. QA verifica que el reporte supere el umbral del 80% como criterio de entrada antes de iniciar la ejecución funcional. |
| **Karate** | 1.4.x | Automatización API | Framework de pruebas de API REST. Automatiza los escenarios funcionales del Motor de Descuentos y del Dashboard de configuración (HU-01, HU-02, HU-03, HU-04, HU-05, HU-06, HU-09, HU-10, HU-11, HU-14, HU-15, HU-16): envío de payloads, validación de respuestas HTTP, control de acceso por rol, verificación de estructura JSON y flujos negativos. |
| **SerenityBDD** | 4.x | Automatización UI / Reporte | Framework de reporte y orquestación BDD. Integrado con Cucumber, automatiza los flujos E2E críticos del Dashboard que requieren navegador real (HU-01, HU-03, HU-04, HU-05, HU-09, HU-12, HU-13, HU-16), documentando la evidencia de ejecución paso a paso con capturas de pantalla. |
| **Cucumber** | 7.x | Especificación BDD | Motor de especificación ejecutable en Gherkin. Define los escenarios de aceptación del Dashboard en lenguaje natural, conectando criterios de aceptación con la automatización de SerenityBDD. |
| **k6** | 0.50.x | Rendimiento | Herramienta de pruebas de rendimiento. Ejecuta escenarios de carga sobre el Motor de Descuentos (HU-11) para verificar el comportamiento bajo volumen de peticiones simultáneas y validar que los tiempos de respuesta se mantienen dentro de los umbrales aceptables. |
| **Maven** | 3.9.x | Build y ejecución | Herramienta de construcción y gestión de dependencias. Orquesta la compilación y ejecución de las suites de Karate y SerenityBDD+Cucumber desde línea de comandos o pipeline CI/CD. |
| **Docker** | 26.x | Entorno / Infraestructura QA | Conteneriza los servicios del entorno de QA (Motor de Descuentos, Dashboard, base de datos) garantizando reproducibilidad y aislamiento entre ejecuciones. |
| **GitHub Actions** | — | CI/CD | Pipeline de integración continua que dispara automáticamente la ejecución de las suites de pruebas ante cada PR o merge a la rama de QA, publicando los reportes generados como artefactos del workflow. |
| **GitHub Issues** | — | Gestión de defectos | Registro y seguimiento de defectos detectados durante la ejecución. Cada issue captura: pasos para reproducir, resultado esperado vs. obtenido, severidad y evidencia adjunta (logs, capturas). |
| **IntelliJ IDEA** | 2024.x | IDE de desarrollo | Entorno de desarrollo integrado utilizado para escribir, depurar y mantener los scripts de prueba de Karate y SerenityBDD+Cucumber. |

---

## 8. Roles y Responsabilidades

| Rol | Persona | Responsabilidades frente a las pruebas |
|---|---|---|
| **QA** | Matías Regaló | Diseñar los casos de prueba y escenarios Gherkin para HU-01 a HU-16; automatizar las suites de SerenityBDD+Cucumber (Dashboard) y Karate (Motor); ejecutar las pruebas en el entorno QA; ejecutar pruebas de rendimiento k6 (HU-11); reportar defectos en GitHub Issues con evidencia; verificar la corrección de defectos; publicar los reportes de SerenityBDD, Karate y k6 como entregables del ciclo |
| **DEV** | Agustín Mites | Escribir y mantener pruebas unitarias con cobertura ≥ 80% para el backend y frontend; garantizar que el entorno Docker QA esté disponible y estable antes de cada ciclo de pruebas; corregir los defectos reportados por QA y notificar cuando estén listos para re-validación |
| **DEV** | Luis Ospino | Escribir y mantener pruebas unitarias con cobertura ≥ 80% para el backend y frontend; garantizar la disponibilidad y correcto funcionamiento de los endpoints bajo prueba; corregir los defectos reportados por QA y notificar cuando estén listos para re-validación |

### HU-01 — Inicio y cierre de sesión

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del flujo de autenticación JWT (login, logout, blacklist de tokens); cubrir rutas de credenciales inválidas y tokens expirados |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints de autenticación (POST /auth/login, POST /auth/logout): credenciales válidas, credenciales inválidas y verificación de que el token queda rechazado tras el logout; automatizar con SerenityBDD+Cucumber el flujo completo de login y cierre de sesión desde el navegador |

### HU-02 — Gestión de Usuarios por Ecommerce

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la lógica de vinculación de usuarios a ecommerce y los filtros de aislamiento por `ecommerce_id` |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints de gestión de usuarios (GET/POST/PUT/DELETE /users): creación exitosa, listado filtrado por ecommerce, actualización de datos y eliminación; verificar que una solicitud con token de otro ecommerce retorna HTTP 403 |

### HU-03 — Administra su Ecommerce (STORE_ADMIN)

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del rol STORE_ADMIN y sus restricciones de acceso a su propio ecommerce |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints de gestión bajo token STORE_ADMIN: acceso permitido a usuarios del propio ecommerce (HTTP 200) y acceso denegado a recursos de otro ecommerce (HTTP 403); automatizar con SerenityBDD+Cucumber la verificación visual de que el Dashboard solo muestra los datos del ecommerce propio |

### HU-04 — Usuario Estándar

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de los permisos del rol USER y el filtro de acceso por ecommerce |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints accesibles bajo token USER: lectura de perfil propio (HTTP 200) y rechazo ante operaciones de gestión reservadas a roles superiores (HTTP 403); automatizar con SerenityBDD+Cucumber la verificación visual de las opciones de menú disponibles y la actualización de perfil desde el Dashboard |

### HU-05 — Gestión y Validación de API Keys

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del middleware de validación de API Keys (hashing, Guard S2S, sincronización vía RabbitMQ); asegurar cobertura de ramas críticas de seguridad |
| **QA** | Matías Regaló | Automatizar con SerenityBDD+Cucumber los escenarios de creación y consulta de claves en el Dashboard; automatizar con Karate la validación y rechazo de API Keys en el Motor; comprobar el enmascaramiento `****XXXX` en la UI |

### HU-06 — Gestión de Reglas de Temporada

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la lógica de superposición de fechas y validación de límites de descuento para `seasonal_rules` |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints de reglas de temporada (GET/POST/PUT/DELETE /seasonal-rules): creación válida, edición de rango de fechas, eliminación, rechazo por superposición con otra regla activa y rechazo por porcentaje de descuento fuera de rango |

### HU-07 — Gestión de Reglas por Tipo de Producto

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la restricción de unicidad por tipo de producto y la validación de parámetros obligatorios en `product_rules` |

### HU-08 — Rangos de Clasificación de Fidelidad

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la validación de continuidad, no superposición y jerarquía ascendente de los rangos en `fidelity_ranges` |

### HU-09 — Límite y Prioridad de Descuentos

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del modelo `discount_config` y la lógica del motor que aplica prioridad y tope; cubrir casos de tope inválido y prioridad duplicada a nivel de unidad |
| **QA** | Matías Regaló | Automatizar con SerenityBDD+Cucumber los escenarios de configuración válida, tope inválido y prioridad ambigua en el Dashboard; automatizar con Karate la aplicación del tope ante acumulación de descuentos en el Motor |

### HU-10 — Clasificación Dinámica de Clientes (Loyalty Tiers)

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la clasificación en memoria (Caffeine) con atributos obligatorios, dominios válidos y determinismo; cubrir sincronización vía RabbitMQ |
| **QA** | Matías Regaló | Automatizar con Karate los escenarios de clasificación exitosa, payload incompleto, valores fuera de dominio y consistencia de resultado; verificar que el mismo payload produce siempre el mismo nivel de fidelidad |

### HU-11 — Cálculo de Carrito con Descuentos Aplicados

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias e integración de `POST /api/v1/engine/calculate` (DTOs, autenticación S2S, aplicación de prioridad y tope); garantizar que los contratos de respuesta estén documentados antes del inicio de la automatización QA |
| **QA** | Matías Regaló | Automatizar con Karate los escenarios de cálculo exitoso, carrito sin descuentos y carrito inválido; verificar el orden de aplicación de descuentos, el límite por tope y el control de acceso S2S; ejecutar con k6 las pruebas de carga sobre el endpoint y validar los umbrales p95 ≤ 500 ms y tasa de error < 1% |

### HU-12 — Historial de Descuentos Aplicados

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del registro de logs de transacciones y la retención a 7 días; garantizar que no se almacene PII del comprador final |
| **QA** | Matías Regaló | Automatizar con SerenityBDD+Cucumber los escenarios de visualización del historial filtrado por los últimos 7 días; verificar el resaltado de transacciones truncadas por tope y la ausencia de datos personales |

### HU-13 — Trazabilidad de Cambios de Reglas (Auditoría)

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias de la generación de eventos de auditoría ante cada CREATE/UPDATE/DELETE de reglas; verificar que la estructura del log incluye todos los campos requeridos |
| **QA** | Matías Regaló | Automatizar con SerenityBDD+Cucumber el escenario de consulta del panel de auditoría por Super Admin; verificar los filtros por tipo de regla, usuario y rango de fechas |

### HU-14 — Activar/Desactivar Reglas

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del cambio de estado de reglas y la propagación inmediata al Motor vía RabbitMQ; verificar que una regla desactivada no participa en evaluaciones |
| **QA** | Matías Regaló | Automatizar con Karate el endpoint de cambio de estado (PATCH /rules/:id/status): desactivar una regla activa y verificar que una llamada posterior a POST /engine/calculate no la aplica (descuento ausente en la respuesta); reactivar la regla y verificar que vuelve a aplicarse |

### HU-15 — Registro de Ecommerces (Onboarding)

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del registro de ecommerces, validación de unicidad del slug y la lógica de desactivación en cascada sobre usuarios y API Keys asociadas |
| **QA** | Matías Regaló | Automatizar con Karate los endpoints de gestión de ecommerces (POST /ecommerces, PATCH /ecommerces/:id/status): registro exitoso, rechazo por slug duplicado y desactivación en cascada; verificar que los tokens JWT de usuarios vinculados retornan HTTP 401 y que las API Keys del ecommerce son rechazadas por el Motor tras la desactivación |

### HU-16 — Acceso Total (Super Admin)

| Rol | Responsable | Actividades de prueba |
|---|---|---|
| **DEV** | Agustín Mites / Luis Ospino | Escribir pruebas unitarias del rol SUPER_ADMIN, su acceso global sin restricción de `ecommerce_id` y las operaciones CRUD sobre STORE_ADMINs y usuarios de cualquier ecommerce |
| **QA** | Matías Regaló | Automatizar con Karate las operaciones CRUD sobre STORE_ADMINs y usuarios de cualquier ecommerce bajo token SUPER_ADMIN (HTTP 200 sin restricción de `ecommerce_id`); verificar el listado global de ecommerces sin filtro; automatizar con SerenityBDD+Cucumber la verificación visual del panel global y la navegación sin restricciones entre ecommerces en el Dashboard |

---

## 9. Cronograma y Estimación

| Sprint | Actividad | Duración estimada |
|---|---|---|
| Sprint 0 | Diseño del plan de pruebas | 1 día |
| Sprint 0 | Diseño de casos de prueba (HU-01 a HU-16) | 2 días |
| Sprint 1 | Automatización HU-01, HU-02, HU-03, HU-04 (SerenityBDD + Cucumber + Karate — Autenticación y Usuarios) | 1.5 días |
| Sprint 1 | Automatización HU-05 (SerenityBDD + Cucumber + Karate — API Keys) | 1 día |
| Sprint 1 | Automatización HU-06 (Karate — Reglas de temporada) | 0.5 días |
| Sprint 1 | Automatización HU-09 (SerenityBDD + Cucumber + Karate — Límite y Prioridad) | 1 día |
| Sprint 2 | Automatización HU-10 (Karate — Clasificación de Clientes) | 0.5 días |
| Sprint 2 | Automatización HU-11 (Karate — Cálculo de Carrito) | 1 día |
| Sprint 2 | Pruebas de rendimiento k6 (HU-11) | 0.5 días |
| Sprint 2 | Automatización HU-12, HU-13 (SerenityBDD + Cucumber — Historial, Auditoría) + HU-14 (Karate — Activación de reglas) | 1 día |
| Sprint 2 | Automatización HU-15 (Karate — Onboarding) + HU-16 (Karate + SerenityBDD + Cucumber — Super Admin) | 0.5 días |
| Sprint 2 | Regresión + reporte final | 0.5 días |
| **TOTAL** | | **11 días** |

---

## 10. Entregables de Prueba

| # | Entregable | Formato | Descripción |
|---|---|---|---|
| 1 | Plan de pruebas | `TEST_PLAN.md` | Documento que estructura el plan de pruebas con los siguientes elementos: Identificación del Plan (nombre, sistema, versión, fecha y equipo), Contexto (sistema bajo prueba y problema de negocio), Alcance (HU validadas y excluidas del ciclo), Estrategia (SerenityBDD + Cucumber para funcional, Karate para API y k6 para rendimiento), Criterios de Entrada y Salida, Entorno de Pruebas, Herramientas y su propósito, Roles y Responsabilidades (QA vs DEV), Cronograma y Estimación (esfuerzo en relación con Story Points), Entregables de Prueba y Riesgos y Contingencias |
| 2 | Casos de prueba | `TEST_CASES.md` | Matriz de casos de prueba organizada por Historia de Usuario. Por cada caso incluye: HU asociada, ID del caso (ej. TC-001), escenario en lenguaje Gherkin (Dado / Cuando / Entonces), precondiciones, datos de entrada, pasos de ejecución, resultado esperado, resultado obtenido y estado (Pasó / Falló / Sin ejecutar), prioridad (Crítico / Alto / Medio / Bajo) y registro de cada caso como sub-tarea dentro de su HU en GitHub Projects. 
| 3 | Repositorio SerenityBDD + Cucumber | Archivos `.feature` | Escenarios Cucumber vinculados a los flujos E2E críticos del Dashboard que requieren navegador real (HU-01, HU-03, HU-04, HU-05, HU-09, HU-12, HU-13, HU-16) |
| 4 | Repositorio Karate | Archivos `.feature` | Scripts Karate para validación de endpoints REST del Motor de Descuentos y del Dashboard de configuración (HU-01, HU-02, HU-03, HU-04, HU-05, HU-06, HU-09, HU-10, HU-11, HU-14, HU-15, HU-16) |
| 5 | Repositorio k6 | Archivos `.js` (k6) | Scripts de carga y estrés para el endpoint de cálculo de carrito (HU-11) |
| 6 | Reporte SerenityBDD | HTML generado | Reporte detallado de ejecución de pruebas funcionales del Dashboard con evidencias paso a paso |
| 7 | Reporte Karate | HTML generado | Reporte detallado de ejecución de pruebas de API REST del Motor de Descuentos y del Dashboard de configuración (HU-01, HU-02, HU-03, HU-04, HU-05, HU-06, HU-09, HU-10, HU-11, HU-14, HU-15, HU-16) |
| 8 | Reporte k6 | HTML / JSON | Reporte de métricas de rendimiento (latencia p95, throughput, tasa de error) |
| 9 | Registro de defectos | GitHub Issues | Defectos encontrados con severidad, pasos de reproducción y evidencia adjunta |
| 10 | Informe final de pruebas | Markdown / PDF | Resumen ejecutivo del ciclo: cobertura, métricas, defectos y recomendaciones |

---

## 11. Riesgos y Contingencias

### 11.1 Riesgos de proyecto

| Descripción                                              | Probabilidad | Impacto | Riesgo | Mitigación principal                                                                                                                       |
| -------------------------------------------------------- | ------------ | ------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Retrasos en la entrega del MVP                           | Alto            | Alto       | Alto      | Comunicar avances y antes de la fecha limite notificar a los clientes que se extenderá la fecha de entrega                                 |
| Cambios frecuentes en el alcance del MVP                 | Alto            | Alto       | Alto      | Evaluar impacto en tiempo/costo y aprobar solo cambios críticos                                                                            |
| Definición ambigua de criterios de aceptación            | Medio            | Alto        | Alto      | Definir criterios de aceptación claros y validar con QA antes de iniciar el sprint.                                                        |
| Problemas de comunicación entre QA y DEV                 | Bajo            | Medio       | Bajo      | Realizar reuniones cortas diarias para asignar responsables y fechas de solución.                                                          |
| Bajo uso inicial del MVP por parte de comercios clientes | Medio            | Alto       | Alto      | Acompañar a los comercios en el uso del sistema durante las primeras 2 semanas y revisar cada semana el nivel de uso para aplicar mejoras. |

### 11.2 Riesgos de producto

| Descripción                                                                            | Probabilidad | Impacto | Riesgo | Mitigación principal                                                                                                      |
| -------------------------------------------------------------------------------------- | ------------ | ------- | ------ | ------------------------------------------------------------------------------------------------------------------------- |
| Cálculo incorrecto del descuento total por orden de aplicación o acumulación de reglas | Alto            | Alto       | Alto      | Definir reglas de cálculo claras y hacer pruebas completas con casos reales antes de publicar.                            |
| Validaciones incompletas o inconsistentes de datos de entrada                          | Alto            | Alto       | Alto      | Validar datos, bloqueando valores vacíos, negativos o fuera de rango.                                                     |
| Cálculo e inyección de descuentos en el checkout                                       | Medio            | Alto       | Alto      | Probar el sistema con alta concurrencia antes y optimizar consultas o procesos lentos detectados.                         |
| Interfaz de configuración poco clara para usuarios del comercio                        | Medio            | Alto       | Alto      | Simplificar la pantalla principal, usar etiquetas claras y validar la usabilidad con pruebas rápidas con usuarios reales. |
| Cambios de reglas sin trazabilidad | Medio | Medio | Medio | Registrar quién, cuándo y qué regla se modificó, y mostrar un historial simple de cambios dentro del módulo |
| Exposición accidental de API Keys en logs o respuestas del sistema | Bajo | Alto | Medio | Implementar enmascaramiento `****XXXX` en toda interfaz y validar en pruebas que el valor completo no aparece en ninguna respuesta ni log |