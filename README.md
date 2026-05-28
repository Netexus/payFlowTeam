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

<img width="527" height="506" alt="1" src="https://github.com/user-attachments/assets/afb623e3-adb4-46bd-b4c3-73c685d8fa7f" />
<img width="524" height="509" alt="2" src="https://github.com/user-attachments/assets/c0340ec8-e5fb-4cd0-b9dd-4daac7ae70ff" />
<img width="500" height="503" alt="3" src="https://github.com/user-attachments/assets/673f0a0b-beef-40e2-af51-b85a88f24cf3" />
<img width="531" height="509" alt="4" src="https://github.com/user-attachments/assets/0fbdcdfb-6b47-4e81-88e9-13dafd0322ea" />
<img width="520" height="451" alt="5" src="https://github.com/user-attachments/assets/9f74b2e5-6ef6-478a-9387-14195518dab0" />


## Decisiones Arquitectónicas (ADRs)

> _Sección en construcción — los ADRs se redactarán progresivamente._

### ADR-01: Azure Event Hubs vs Azure Service Bus como punto de entrada

# ADR-01: Azure Event Hubs vs Azure Service Bus como punto de entrada

## Estado
Aceptado

## Contexto

PayFlow procesa transacciones digitales en tiempo real para pequeños y medianos comercios en Colombia, Ecuador y Perú. La plataforma presenta picos de hasta 260.000 transacciones diarias y actualmente enfrenta problemas de escalabilidad en momentos de alta demanda.

El sistema necesita un mecanismo de entrada capaz de recibir grandes volúmenes de eventos sin generar cuellos de botella, permitiendo desacoplar los productores de eventos del procesamiento posterior.

El equipo evaluó dos alternativas para el punto principal de entrada del sistema:

Azure Event Hubs
Azure Service Bus

## Decisión

Se decidió utilizar Azure Event Hubs como punto de entrada principal de eventos para la plataforma PayFlow.

Azure Event Hubs actuará como buffer distribuido de eventos, recibiendo el flujo de transacciones generadas por el sistema y permitiendo su procesamiento asíncrono por los componentes posteriores de Azure Functions, Service Bus y Cosmos DB.

Azure Service Bus no se utilizará como punto de entrada principal, sino como mecanismo complementario para flujos críticos y transacciones de alto valor.

## Consecuencias

### Positivas

Alta capacidad de ingestión de eventos en tiempo real
Mejor tolerancia a picos de tráfico
Desacoplamiento entre productores y consumidores
Escalabilidad horizontal para cargas variables
Integración nativa con Azure Functions
Adecuado para arquitecturas event-driven

### Negativas

 No ofrece las capacidades avanzadas de colas empresariales de Service Bus
 Requiere servicios complementarios para flujos críticos
 Necesita diseño cuidadoso de particiones y retención

## Alternativas consideradas

### Azure Service Bus

**Ventajas:**
Manejo avanzado de mensajes
Colas y tópicos
Dead-letter queue
Retries y control empresarial

**Desventajas:**
 Menor capacidad para ingestión masiva de eventos
Menor adecuación para escenarios de streaming de alto volumen
 Puede convertirse en cuello de botella en picos de tráfico

## Referencias
Azure Event Hubs Documentation
Azure Service Bus Documentation
Azure Event-Driven Architecture Patterns
















### ADR-02: Azure Functions vs Azure Stream Analytics

## Estado
Aceptado

## Contexto

PayFlow requiere validar transacciones, aplicar reglas antifraude básicas, enrutar operaciones de alto valor y registrar resultados finales en una base de datos de baja latencia.

El equipo necesita una capa de procesamiento que pueda reaccionar a eventos en tiempo real y ejecutar lógica personalizada sobre cada transacción.

Las alternativas evaluadas fueron:

Azure Functions
Azure Stream Analytics

## Decisión

Se decidió utilizar Azure Functions como componente principal de procesamiento del flujo de transacciones.

Azure Functions recibirá eventos desde Azure Event Hubs, ejecutará validaciones de formato y reglas antifraude básicas, enrutarará las transacciones de alto valor hacia Azure Service Bus y persistirá el resultado final en Cosmos DB.

Azure Stream Analytics no fue seleccionado como componente principal porque el caso requiere lógica personalizada, control por transacción y mayor flexibilidad en el procesamiento de eventos.

## Consecuencias

### Positivas

 Procesamiento serverless bajo demanda
 Integración directa con Event Hubs y Service Bus
 Flexibilidad para lógica personalizada en Node.js
 Escalado automático según carga
 Menor complejidad operativa

### Negativas

Puede requerir más código que una solución puramente declarativa
Depende de una buena estrategia de manejo de errores
El monitoreo y trazabilidad deben configurarse explícitamente

## Alternativas consideradas

### Azure Stream Analytics

**Ventajas:**
Muy útil para análisis de streaming
Configuración declarativa
Buen soporte para consultas sobre flujos

**Desventajas:**
Menor flexibilidad para lógica transaccional personalizada
No es la opción más directa para reglas de negocio por evento
Menos conveniente para enrutar casos específicos con lógica de aplicación

## Referencias

 Azure Functions Documentation
 Azure Stream Analytics Documentation
 Serverless computing patterns






















### ADR-03: Cosmos DB vs Azure SQL Database

## Estado
Aceptado

## Contexto

PayFlow necesita almacenar el estado final de las transacciones procesadas por la plataforma, con baja latencia, alta concurrencia y capacidad de crecimiento continuo.

El sistema maneja un volumen alto de eventos y necesita persistencia flexible para registros transaccionales, resultados de procesamiento y consultas operativas.

Las alternativas evaluadas fueron:

Azure Cosmos DB
Azure SQL Database

## Decisión

Se decidió utilizar Azure Cosmos DB como sistema principal de persistencia para las transacciones procesadas.

Cosmos DB permitirá almacenar y consultar los resultados del flujo de eventos con baja latencia, escalabilidad horizontal y un modelo de datos flexible, adecuado para una arquitectura orientada a eventos.

Azure SQL Database no fue seleccionada como opción principal porque el escenario prioriza distribución, elasticidad y alto volumen de escritura por encima de un modelo relacional tradicional.

## Consecuencias

### Positivas

Baja latencia en lectura y escritura
Escalabilidad horizontal
Alta disponibilidad distribuida
Modelo flexible para documentos JSON
Integración adecuada con arquitecturas event-driven
Buen ajuste para cargas variables e impredecibles

### Negativas

Modelado más cuidadoso de particiones
Costos ligados a capacidad y consumo
Menor naturalidad para consultas relacionales complejas
Requiere diseño explícito de partition key

## Alternativas consideradas

### Azure SQL Database

**Ventajas:**
 Modelo relacional maduro
 SQL tradicional
 Buen soporte para integridad relacional
 Útil para consultas complejas con joins

**Desventajas:**
Menor flexibilidad para datos semiestructurados
Escalabilidad menos natural para cargas muy variables
Menor adecuación para flujos de eventos de alta concurrencia

## Referencias

Azure Cosmos DB Documentation
Azure SQL Database Documentation
Cloud-native persistence patterns




### ADR-04: Service Bus vs Storage Queue para alto valor

## Estado
Aceptado

## Contexto

PayFlow procesa transacciones financieras en tiempo real y necesita tratar de forma especial las operaciones de alto valor, definidas en el proyecto como transacciones superiores a $5.000.000 COP.

Estas operaciones requieren mayor confiabilidad, priorización y control de errores que una cola básica.

Las alternativas evaluadas fueron:

Azure Service Bus
Azure Storage Queue

## Decisión

Se decidió utilizar Azure Service Bus para encolar y procesar las transacciones de alto valor.

Azure Service Bus ofrece capacidades empresariales de mensajería que permiten priorizar operaciones críticas, controlar reintentos, manejar errores mediante dead-letter queues y desacoplar la lógica de procesamiento.

Azure Storage Queue no fue seleccionada como mecanismo principal para este caso debido a sus limitaciones en control avanzado de mensajes y priorización.

## Consecuencias

### Positivas

Mayor confiabilidad en el procesamiento
Mejor control de mensajes críticos
Retries automáticos
Dead-letter queue para errores
Integración nativa con Azure Functions
Adecuado para flujos financieros sensibles

### Negativas

 Mayor complejidad de configuración
 Costos algo superiores a una cola básica
 Requiere una administración más cuidadosa

## Alternativas consideradas

### Azure Storage Queue

**Ventajas:**
Más simple de configurar
Costo más bajo
Buena integración con Azure

**Desventajas:**
Menor soporte para mensajería empresarial
Sin priorización avanzada
Menor control para escenarios financieros críticos

## Referencias

Azure Service Bus Documentation
Azure Queue Storage Documentation
Messaging reliability patterns


### ADR-05: Azure Monitor vs solución de observabilidad de terceros

<img width="934" height="818" alt="image" src="https://github.com/user-attachments/assets/d638f619-b7fe-428a-be6e-646eaac90190" />



## Implementación — Flujo crítico

> _Sección actualizada con evidencias de implementación
del flujo crítico y monitoreo cloud._

| Paso | Acción                                                                  | Servicio                             | Estado          |
|------|-------------------------------------------------------------------------|--------------------------------------|-----------------|
|  1   | Script Python genera eventos de transacción y los publica en Event Hubs | Azure Event Hubs                     | ✅ Implementado  |
|  2   | Function valida formato y aplica regla antifraude básica                | Azure Functions — validarTransaccion | ✅ Implementado |
|  3   | Si monto > $5M COP, enruta a cola de alto valor en Service Bus          | Azure Service Bus                    | ✅ Implementado |
|  4   | Function registra resultado final en Cosmos DB                          | Cosmos DB                            | ✅ Implementado |
|  5   | Azure Monitor muestra métricas del flujo en tiempo real                 | Azure Monitor                        | ✅ Implementado |

## Generación de eventos con Python y Event Hubs

Se implementó un script en Python encargado de simular transacciones bancarias en tiempo real.

El script genera eventos aleatorios con información como:

- cliente

- monto

- tipo de transacción


Posteriormente, los eventos son enviados a Azure Event Hubs utilizando el SDK oficial de Azure para Python.

Estas transacciones activan automáticamente el flujo serverless implementado en Azure:

Azure Function procesa el evento.
Se valida si la transacción es de alto valor.
Las transacciones mayores a $5.000.000 COP se envían a Azure Service Bus.
Todas las tr ansacciones son almacenadas en Cosmos DB.
Azure Monitor y Application Insights registran las métricas y logs del sistema.

## Evidencias obtenidas

Durante la ejecución del script se validó:

✅ Envío correcto de eventos a Event Hubs

✅ Activación automática de Azure Functions

✅ Registro de transacciones en Cosmos DB

✅ Enrutamiento de transacciones de alto valor a Service Bus

✅ Generación de métricas y logs en Azure Monitor y Application Insights


<img width="867" height="961" alt="image" src="https://github.com/user-attachments/assets/e45bbb6c-9c07-4a2e-940e-43805a4a74b8" />






## Evidencia — Azure Functions

La Azure Function procesa transacciones financieras,
valida el monto y enruta operaciones de alto valor.

<img width="921" height="214" alt="image" src="https://github.com/user-attachments/assets/390c7a26-9a38-4aea-9878-380cfe4d3b72" />

## Evidencia — Azure Service Bus

Las transacciones superiores a $5.000.000 COP
son enviadas automáticamente a la cola
de alto valor.

<img width="921" height="253" alt="image" src="https://github.com/user-attachments/assets/311554a2-3b2b-4857-9e74-51b19f7aa36d" />

## Evidencia — Cosmos DB

Las transacciones procesadas son almacenadas
en Cosmos DB para persistencia y consulta.

<img width="921" height="320" alt="image" src="https://github.com/user-attachments/assets/72082c38-71e4-4e26-8a6c-146f88f6bcde" />

## Evidencia — Azure Monitor

Azure Monitor y Application Insights permiten
visualizar logs, solicitudes, errores y métricas
del sistema en tiempo real.

<img width="921" height="346" alt="image" src="https://github.com/user-attachments/assets/68df3d1e-8b4d-4b37-aac2-93f72c55f3b7" />

## Consulta de trazas (Logs)

<img width="921" height="415" alt="image" src="https://github.com/user-attachments/assets/36235029-4937-4c70-9d50-a38e62aa3698" />

## Consulta de solicitudes

<img width="921" height="239" alt="image" src="https://github.com/user-attachments/assets/560f9ecf-129e-4396-ba45-ccf1a21c86a5" />

## Consulta de errores

<img width="921" height="280" alt="image" src="https://github.com/user-attachments/assets/694e78e7-bc7a-4847-94fe-600d933281b3" />

## Consulta de operaciones exitosas

<img width="921" height="196" alt="image" src="https://github.com/user-attachments/assets/7c52670d-7133-4130-8a22-6303468b4eb8" />


## Conclusiones

El desarrollo del caso PayFlow permitió diseñar e implementar una arquitectura cloud orientada a eventos utilizando servicios de Microsoft Azure, enfocada en el procesamiento de transacciones financieras en tiempo real.

La solución propuesta logró desacoplar el flujo transaccional mediante Azure Event Hubs, Azure Functions, Service Bus y Cosmos DB, mejorando la escalabilidad, resiliencia y capacidad de procesamiento ante altos volúmenes de eventos.

Además, la integración de Azure Monitor y Application Insights permitió incorporar observabilidad centralizada, facilitando el monitoreo de métricas, logs y errores del sistema en tiempo real.

El uso del modelo C4 y de los ADRs permitió documentar la arquitectura y justificar técnicamente las decisiones tomadas durante el proyecto, alineando la solución con buenas prácticas de arquitecturas event-driven y computación serverless en entornos cloud.

---

*Última actualización: 28 de abril de 2026*
