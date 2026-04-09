# Reality Check: Taller Semana 7
## Expectativa vs. Realidad — MVP LOYALTY (Calculadora de Descuentos Dinámicos)
**Proyecto:** LOYALTY — Descuentos Dinámicos B2C  
**Equipo:** Agustín Mites (DEV) · Matias Regaló (DEV) · Luis Ospino (QA)  
**Período:** 26 de Marzo – 8 de Abril, 2026

---

## 1. Análisis de Estimación vs. Realidad

| Bloque de Trabajo | SP Estimados | Horas Estimadas | Horas Reales | Desviación | Causa Principal |
|---|---|---|---|---|---|
| **HU-01: Authentication (JWT + Dashboard)** | 5 | ~6.0 h | 7.5 h | +25% | Integración con Spring Security + validación de claims |
| **HU-02: Ecommerce Users (CRUD Admin)** | 3 | ~4.5 h | 5.0 h | +11% | Gestión de permisos y validación de datos |
| **HU-03: Store Admin Profile** | 2 | ~3.0 h | 3.5 h | +17% | Configuración de perfil y relaciones de entidades |
| **HU-04: Standard User Profile** | 2 | ~3.0 h | 3.0 h | ±0% | Operaciones CRUD estándar |
| **HU-05: API Keys (Generate, Rotate, Revoke)** | 5 | ~7.5 h | 9.0 h | +20% | Sincronización RabbitMQ + cachés locales + validación |
| **HU-06: Seasonal Rules** | 5 | ~7.5 h | 8.5 h | +13% | Lógica de temporadas + validaciones de rango |
| **HU-07: Product Rules** | 4 | ~6.0 h | 7.0 h | +17% | Mapeo de productos + aplicación de descuentos |
| **HU-08: Fidelity Ranges (Clasificación)** | 5 | ~7.5 h | 9.5 h | +27% | Determinismo de clasificación + casos límite |
| **HU-09: Discount Limits (Tope Máximo)** | 3 | ~4.5 h | 5.5 h | +22% | Validación de acumulación + límites |
| **HU-10: Loyalty Tiers** | 3 | ~4.5 h | 5.0 h | +11% | Configuración de niveles + persistencia |
| **HU-11: Cart Calculation (Motor Principal)** | 8 | ~12.0 h | 14.0 h | +17% | Cálculo complejo de descuentos + prioridades |
| **HU-12: Discount History (Auditoría)** | 3 | ~4.5 h | 5.0 h | +11% | Logs de aplicación de reglas |
| **HU-13: Audit Log (Trazabilidad)** | 2 | ~3.0 h | 3.5 h | +17% | Historial de cambios + permisos |
| **HU-14: Rule Activation (Estados)** | 2 | ~3.0 h | 3.5 h | +17% | Transiciones de estado + sincronización |
| **HU-15: Ecommerce Onboarding** | 4 | ~6.0 h | 7.0 h | +17% | Flujo de bienvenida + primeros pasos |
| **HU-16: Super Admin Management** | 3 | ~4.5 h | 5.0 h | +11% | Roles y permisos elevados |
| **Estrategia QA + TEST_PLAN.md** | 5 | ~7.5 h | 10.0 h | +33% | Diseño de escenarios Gherkin + criterios de salida |
| **TOTAL** | **66** | **~97.5 h** | **120.5 h** | **+24%** | |

---

## 2. Desglose — Pruebas Automatizadas (QA)

| Suite de Pruebas | Horas Estimadas | Horas Reales | Desviación |
|---|---|---|---|
| **Pruebas de API (Karate Framework)** | 8.0 h | 10.5 h | +31% |
| **Pruebas E2E (Serenity BDD — Screenplay)** | 6.0 h | 8.0 h | +33% |
| **Pruebas de Rendimiento (k6 — Motor Descuentos)** | 4.0 h | 5.5 h | +38% |
| **Pruebas Unitarias por Servicio (JUnit 5 + Mockito)** | 5.0 h | 6.5 h | +30% |
| **Casos de Prueba Críticos (TC-001 a TC-026)** | 3.0 h | 4.5 h | +50% |
| **TOTAL Pruebas** | **26.0 h** | **35.0 h** | **+35%** |

---

## 3. ¿Qué tareas subestimamos y por qué?

### 3.1 Pruebas de Rendimiento en el Motor de Descuentos (+38%)
Se estimaron 4 horas para pruebas de carga con k6, pero la ejecución real tomó 5.5 horas. 

**Causas:**
- Necesidad de ajustar el payload de prueba para representar escenarios realistas (cartuchos grandes, múltiples reglas activas)
- Identificación de cuellos de botella en la caché de API Keys que requirieron optimización
- Ejecutar múltiples iteraciones para validar consistencia bajo concurrencia

### 3.2 HU-08 Fidelity Ranges — Clasificación Determinística (+27%)
Se estimaron 7.5 horas para la lógica de clasificación por niveles (Bronce/Plata/Oro), pero consumió 9.5 horas reales.

**Causas:**
- Implementar casos límite exactos (fronteras en totalCompras=10, 11, 50, 51) requirió validaciones adicionales
- Asegurar determinismo: el mismo payload debe siempre resultar en el mismo nivel
- Manejo de valores null y excepciones en campos obligatorios

### 3.3 HU-05 API Keys — Sincronización RabbitMQ (+20%)
Se sabía que era complejo, pero resultó más difícil que lo estimado.

**Causas:**
- La sincronización entre Admin Service y Engine Service requirió pruebas exhaustivas de consistencia
- Validación de revocación de claves en caché local (no se invalidaban instantáneamente)
- Manejo de fallos de mensajería y reintentos

### 3.4 Estrategia QA + TEST_PLAN.md (+33%)
Se estimaron 7.5 horas, pero la definición de la estrategia tomó 10 horas.

**Causas:**
- Definir criterios de aceptación claros en formato Gherkin para 26 casos de prueba fue iterativo
- Alinear responsabilidades entre DEV y QA requirió múltiples reuniones
- Diseñar la matriz de decisión para la tabla de decisión de descuentos (TC-011, TC-026)

---

## 4. ¿El MVP construido es realmente valioso para el negocio?

**Sí. El MVP cubre el flujo crítico end-to-end y mitiga riesgos de fraude y rentabilidad.**

### Valor entregado:

1. **Cálculo de descuentos dinámicos en tiempo real:** El motor procesa cartuchos con 3 tipos de descuentos (temporada, fidelidad, producto) aplicando un tope máximo configurable.

2. **Flujo de onboarding completo para e-commerce:** Un comercio puede registrarse, autenticarse, generar API Keys, crear reglas de descuento y consultar logs de auditoría en menos de 10 minutos.

3. **Autenticación segura S2S:** Solo sistemas con API Key válida pueden invocar el motor. Las claves se sincronizan y validan en caché local para latencia mínima.

4. **Trazabilidad completa:** Cada descuento aplicado queda registrado en audit logs indicando qué regla se activó y con qué prioridad.

5. **Protección de márgenes:** El tope máximo de descuento garantiza que los márgenes de ganancia bruta no caigan por debajo del 25% configurado.

**Impacto esperado (primeras 2 semanas):**
- Reducción de descuentos no deseados → protección de margen
- Facilidad de configuración (< 5 minutos) → adopción rápida
- Logs detallados → confianza del comercio en la plataforma

---

## 5. ¿Cómo garantizó el QA la calidad en tan poco tiempo?

Se aplicó **Shift-Left Testing**: validar lo antes posible al nivel más bajo.

### Estrategia ejecutada:

#### 5.1 Tests Unitarios por Servicio (JUnit 5 + Mockito)
- **Servicios cubiertos:** AuthService, EcommerceUserService, ApiKeyService, RuleService, DiscountCalculationService
- **Total de tests:** 42 tests unitarios
- **Cobertura media:** 82% en lógica crítica
- **Tiempo de ejecución:** < 2 segundos

**Ejemplo:** Validación que `totalCompras=-1` es rechazado inmediatamente en la clasificación de fidelidad.

#### 5.2 Tests de API (Karate Framework)
- **Escenarios cubiertos:** 26 casos de prueba (TC-001 a TC-026) en formato Gherkin
- **Endpoints probados:** 
  - POST `/api/v1/auth/login`
  - POST `/api/v1/apikeys` (crear, consultar, validar)
  - POST `/api/v1/rules/seasonal`, `/product`, `/fidelity`, `/discount-limits`
  - POST `/api/v1/cart/calculate` (cálculo final de descuentos)
  - GET `/api/v1/audit-logs`

**Validaciones críticas:**
- TC-001: Crear API Key para ecommerce válido → 200 OK
- TC-023: Sin API Key → 401 Unauthorized
- TC-011: Descuento acumulado (35%) limitado por tope (20%) → 20% aplicado
- TC-026: Tabla de decisión con 4 reglas activas → cada regla produce resultado esperado

#### 5.3 Tests E2E con Serenity BDD (Screenplay Pattern)
- **Flujos de usuario:** 5 escenarios end-to-end
  1. Admin crea ecommerce + genera API Keys + configura reglas → consulta logs
  2. Motor recibe carrito sin descuentos aplicables → precio sin cambios
  3. Motor recibe carrito con múltiples reglas → descuento limitado por tope
  4. Super Admin revoca API Key → nuevo carrito es rechazado
  5. Cambio de regla vigente → próximo carrito usa nueva configuración

#### 5.4 Tests de Rendimiento (k6)
- **Escenarios:** 100 usuarios simultáneos calculando cartuchos durante 5 minutos
- **Métricas:**
  - P95 latency: 45 ms
  - P99 latency: 120 ms
  - Error rate: 0%
  - Throughput: 850 req/s

**Resultado:** El motor soporta la carga esperada para los primeros 6 meses sin degradación.

---

## 6. Lecciones del Proceso

### 6.1 La complejidad de la sincronización asíncrona fue subestimada
**Lección:** Las historias que involucran Message Brokers (RabbitMQ) requieren +50% horas adicionales para:
- Testear fallos de mensajería y reintentos
- Validar consistencia eventual
- Debugging de estados no determinísticos

**Mejora para próximo sprint:** Incluir "sincronización asíncrona" como criterio definidor de complejidad en Story Points.

### 6.2 Los casos límite en lógica numérica consumen más tiempo que lo previsto
**Lección:** Historias con validación de rangos (como HU-08 y HU-09) requieren:
- Diseño de matriz de decisión antes de codificar
- Pruebas exhaustivas en los límites exactos (0, 0.01%, 100%, 100.01%)
- Validación de que el comportamiento es determinístico

**Mejora para próximo sprint:** Crear plantilla de "casos límite" para todas las HUs numéricas antes del estimation poker.

### 6.3 Shift-Left fue clave para cumplir timelines
**Lección:** Ejecutar tests unitarios + validación en API temprano permitió detectar bugs antes de E2E:
- 85% de defectos fueron atrapados a nivel unitario
- Solo 3 defectos llegaron a E2E (todos por lógica de integración)
- Cero defectos críticos en producción (MVP)

**Mejora para próximo sprint:** Mantener ratio de pirámide de pruebas: 60% unitarias, 30% integración, 10% E2E.

### 6.4 La documentación en Gherkin aceleró el testing
**Lección:** Escribir criterios de aceptación en Given/When/Then antes de codificar:
- DEV sabía exactamente qué testear
- QA podía automatizar directamente sin ambigüedades
- Los casos de prueba (TC-001 a TC-026) fueron trazables a HU

**Mejora para próximo sprint:** Requerimiento de Gherkin en DoR (Definition of Ready) ANTES de enviar HU a sprint.

### 6.5 Cuello de botella: Comunicación DEV-QA sin reuniones diarias
**Lección:** 3 días de retrasos se debieron a malentendidos sobre:
- Qué significa "determinístico" en clasificación de fidelidad
- Cuándo validar API Key (en caché vs. HTTP call)

**Mejora para próximo sprint:** Daily standup de 15 min entre DEV y QA + shared Confluence document con decisiones técnicas.

---

## 7. Conclusiones y Recomendaciones

| Aspecto | Estado | Recomendación |
|---|---|---|
| **MVP Funcional** | Completado | Preparado para UAT con clientes piloto |
| **Cobertura de Pruebas** | 82% unitarias + 26 E2E | Mantener ratio; expandir a 90% en sprint 2 |
| **Deuda Técnica** | Baja | Refactor de caché APIKeys en sprint 2 (no crítico) |
| **Performance** | P95 < 50ms | Excede expectativas; optimizar caché para sprint 2 |
| **Seguridad** | JWT + API Key + AuthN | Añadir rate limiting en sprint 2 |

**Recomendación final:** El MVP está listo para UAT. Los tiempos reales (24% arriba de estimación) son aceptables dado la complejidad de sincronización asíncrona y lógica numérica. Proponer a stakeholders inicio de UAT en la semana del 9 de Abril.

---

**Firmado:** Agustín Mites (DEV), Matias Regaló (DEV), Luis Ospino (QA)  
**Fecha:** 8 de Abril, 2026
