# PayFlow — Procesamiento de Eventos en Tiempo Real

> **Tecnológico de Antioquia — Institución Universitaria**  
> Computación en la Nube | Semestre 2026-1  
> Profesor: Julian David Florez Sanchez  
> Caso 3 · Trabajo Grupal — Análisis e Implementación de Arquitectura Cloud

---

## Información del proyecto

| Campo      | Detalle                                          |
|------------|--------------------------------------------------|
| Caso       | 03 — Procesamiento de Eventos en Tiempo Real     |
| Empresa    | PayFlow (fintech colombiana)                     |
| Plataforma | Microsoft Azure (Free Tier / Azure for Students) |
| Inicio     | 21 de abril de 2026                              |
| Entrega    | 14 de mayo de 2026 — 11:59 p.m.                  |
| Valor      | 20% de la nota final del curso                   |

---

## Integrantes

| Nombre                        | Rol                      | Responsabilidad principal                            |
|-------------------------------|---------------------------------------------------------------------------------|
| Monica Atehortua              | Arquitecto de solución   | Diseño del modelo C4, ADR-01 y ADR-02                |
| Luis Martínez                 | Desarrollador backend    | Azure Functions en Node.js y script generador Python |
| David Torres                  | Ingeniero de datos       | Cosmos DB, Service Bus y configuración de Event Hubs |
| Brayan Ceballos               | Ingeniero de operaciones | Azure Monitor, App Insights, ADR-05 y evidencias     |

---

## Descripción del caso

PayFlow es una fintech colombiana fundada en 2020 que ofrece una plataforma de pagos digitales para pequeños y medianos comercios. Opera como intermediario entre comercios, adquirentes bancarios y redes de pago (Visa, Mastercard, PSE), procesando transacciones de compra, reembolsos, pagos de servicios y transferencias entre cuentas.

**Datos clave:**
- 28.000 comercios activos
- 85.000 transacciones diarias en promedio
- Hasta 260.000 transacciones en temporada alta
- Presencia en Colombia, Ecuador y Perú

### Problemas identificados

| Problema                             | Impacto                                            |
|--------------------------------------|----------------------------------------------------|
| Cuello de botella en picos           | Límite de 40 tx/seg → timeouts en comercios        |
| Sin separación de flujos críticos    | Micropagos bloquean transacciones de alto valor    |
| Detección de fraude reactiva         | El fraude se detecta después de autorizar          |
| Observabilidad limitada              | Las alertas llegan por WhatsApp, no por monitoreo  |
| Acoplamiento validación-notificación | Fallos en webhook revierten autorizaciones válidas |

---

## Stack tecnológico

| Servicio Azure               | Responsabilidad                                     | Tier                |
|------------------------------|-----------------------------------------------------|---------------------|
| Azure Event Hubs             | Punto de entrada — buffer distribuido de eventos    | Basic               |
| Azure Functions              | Validación, antifraude, enrutamiento y registro     | Consumption Plan    |
| Azure Service Bus            | Cola de alta prioridad para transacciones > $5M COP | Basic               |
| Cosmos DB                    | Persistencia del estado final de transacciones      | Free tier           |
| Azure Monitor + App Insights | Observabilidad, métricas y alertas automáticas      | Gratuito (5 GB/mes) |

---

## Estructura del repositorio

```
PayFlow/
├── README.md               # Documento principal
├── assets/                 # Diagramas C4 y capturas de pantalla
│   └── 
└── src/                    # Código fuente
    └── 
```

---

## Contenido de la documentación

- [ ] Modelo C4 — C1 Contexto
- [ ] Modelo C4 — C2 Contenedores
- [ ] Modelo C4 — C3 Componentes
- [ ] ADR-01: Azure Event Hubs vs Service Bus como punto de entrada
- [ ] ADR-02: Azure Functions vs Azure Stream Analytics
- [ ] ADR-03: Cosmos DB vs Azure SQL Database
- [ ] ADR-04: Service Bus vs Storage Queue para alto valor
- [ ] ADR-05: Azure Monitor vs solución de terceros
- [ ] Implementación — Evidencias del flujo crítico
- [ ] Conclusiones

---

## Arquitecturas de referencia Microsoft

- [Event-driven architecture](https://learn.microsoft.com/es-es/azure/architecture/guide/architecture-styles/event-driven)
- [Azure Event Hubs](https://learn.microsoft.com/es-es/azure/event-hubs/event-hubs-about)
- [Azure Functions — trigger Event Hubs](https://learn.microsoft.com/es-es/azure/azure-functions/functions-bindings-event-hubs)
- [Azure Service Bus](https://learn.microsoft.com/es-es/azure/service-bus-messaging/service-bus-messaging-overview)
- [Azure Cosmos DB](https://learn.microsoft.com/es-es/azure/cosmos-db/introduction)
- [Azure Monitor](https://learn.microsoft.com/es-es/azure/azure-monitor/overview)

---

## Modelo C4

> _Sección en construcción — los diagramas se agregarán en el siguiente commit._

### C1 — Contexto

<img width="736" height="547" alt="c1" src="https://github.com/user-attachments/assets/ca105b21-13a5-488d-9d1b-b4b0e833a5bc" />


### C2 — Contenedores

<img width="794" height="538" alt="c2" src="https://github.com/user-attachments/assets/2f423346-e19b-40c2-9c87-66a5bbafd883" />


### C3 — Componentes

<img width="1536" height="472" alt="c3" src="https://github.com/user-attachments/assets/3b694909-fdaf-4ab2-ab85-2a07846972cc" />


## Decisiones Arquitectónicas (ADRs)

> _Sección en construcción — los ADRs se redactarán progresivamente._

### ADR-01: Azure Event Hubs vs Azure Service Bus como punto de entrada

_(Pendiente)_

### ADR-02: Azure Functions vs Azure Stream Analytics

_(Pendiente)_

### ADR-03: Cosmos DB vs Azure SQL Database

_(Pendiente)_

### ADR-04: Service Bus vs Storage Queue para alto valor

_(Pendiente)_

### ADR-05: Azure Monitor vs solución de observabilidad de terceros

<img width="934" height="818" alt="image" src="https://github.com/user-attachments/assets/d638f619-b7fe-428a-be6e-646eaac90190" />



## Implementación — Flujo crítico

> _Sección actualizada con evidencias de implementación
del flujo crítico y monitoreo cloud._

| Paso | Acción                                                                  | Servicio                             | Estado          |
|------|-------------------------------------------------------------------------|--------------------------------------|-----------------|
|  1   | Script Python genera eventos de transacción y los publica en Event Hubs | Azure Event Hubs                     | ⏳ Pendiente    |
|  2   | Function valida formato y aplica regla antifraude básica                | Azure Functions — validarTransaccion | ✅ Implementado |
|  3   | Si monto > $5M COP, enruta a cola de alto valor en Service Bus          | Azure Service Bus                    | ✅ Implementado |
|  4   | Function registra resultado final en Cosmos DB                          | Cosmos DB                            | ✅ Implementado |
|  5   | Azure Monitor muestra métricas del flujo en tiempo real                 | Azure Monitor                        | ✅ Implementado |

Evidencia — Azure Functions

La Azure Function procesa transacciones financieras,
valida el monto y enruta operaciones de alto valor.

<img width="921" height="214" alt="image" src="https://github.com/user-attachments/assets/390c7a26-9a38-4aea-9878-380cfe4d3b72" />

Evidencia — Azure Service Bus

Las transacciones superiores a $5.000.000 COP
son enviadas automáticamente a la cola
de alto valor.

<img width="921" height="253" alt="image" src="https://github.com/user-attachments/assets/311554a2-3b2b-4857-9e74-51b19f7aa36d" />

Evidencia — Cosmos DB

Las transacciones procesadas son almacenadas
en Cosmos DB para persistencia y consulta.

<img width="921" height="320" alt="image" src="https://github.com/user-attachments/assets/72082c38-71e4-4e26-8a6c-146f88f6bcde" />

Evidencia — Azure Monitor

Azure Monitor y Application Insights permiten
visualizar logs, solicitudes, errores y métricas
del sistema en tiempo real.

<img width="921" height="346" alt="image" src="https://github.com/user-attachments/assets/68df3d1e-8b4d-4b37-aac2-93f72c55f3b7" />

Consulta de trazas (Logs)

<img width="921" height="415" alt="image" src="https://github.com/user-attachments/assets/36235029-4937-4c70-9d50-a38e62aa3698" />

Consulta de solicitudes

<img width="921" height="239" alt="image" src="https://github.com/user-attachments/assets/560f9ecf-129e-4396-ba45-ccf1a21c86a5" />

Consulta de errores

<img width="921" height="280" alt="image" src="https://github.com/user-attachments/assets/694e78e7-bc7a-4847-94fe-600d933281b3" />

Consulta de operaciones exitosas

<img width="921" height="196" alt="image" src="https://github.com/user-attachments/assets/7c52670d-7133-4130-8a22-6303468b4eb8" />


## Conclusiones

> _Sección pendiente — se completará al finalizar la implementación._

---

*Última actualización: 28 de abril de 2026*
