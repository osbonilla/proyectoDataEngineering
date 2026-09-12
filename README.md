# Geohabilitación de Pipelines de Datos

## Proyecto de Ingeniería de Datos - USFQ

**Evelyn Nathaly Bermeo Granda**

**Oldrin Santiago Bonilla Cáceres**

**Reporte ejecutivo — Proyecto de Ingeniería de Datos Geoespaciales**

**Tecnologías:** ArcGIS Data Pipelines · ArcGIS Velocity · ArcGIS Enterprise

> Conectando pipelines de datos con el territorio para transformar información dispersa en decisiones operativas.

---

## Índice

- [Resumen ejecutivo](#resumen-ejecutivo)
- [Objetivo](#objetivo)
- [Concepto: Geohabilitación de datos](#concepto-geohabilitación-de-datos)
- [Arquitectura conceptual](#arquitectura-conceptual)
- [Casos de uso](#casos-de-uso)
  - [Caso 1: Gestión de incidencias en luminarias](#caso-1-gestión-de-incidencias-en-luminarias)
  - [Caso 2: Integración y estandarización de proyectos nacionales](#caso-2-integración-y-estandarización-de-proyectos-nacionales)
  - [Caso 3: Monitoreo de embarcaciones en tiempo real](#caso-3-monitoreo-de-embarcaciones-en-tiempo-real)
  - [Comparación de los tres casos](#comparación-de-los-tres-casos)
- [Arquitectura e infraestructura](#arquitectura-e-infraestructura)
- [Patrones de procesamiento](#patrones-de-procesamiento)
- [Resultados y valor generado](#resultados-y-valor-generado)
- [Conclusiones](#conclusiones)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Documentación oficial](#documentación-oficial)

---

## Resumen ejecutivo

Este proyecto demuestra cómo la Ingeniería de Datos puede incorporar una dimensión geoespacial a los flujos de información empresarial, conectando sistemas transaccionales, fuentes externas y datos en tiempo real con el territorio.

Se desarrollaron **tres casos de uso**, cada uno representando un patrón de datos y una necesidad operativa distinta:

- **Gestión de incidencias en campo**: conecta un ERP con el sistema geoespacial para dar seguimiento al ciclo de vida de una falla, desde su detección hasta su resolución.
- **Integración multi-departamental**: consolida información geográfica de ocho áreas de una organización y de fuentes externas en una sola fuente de verdad.
- **Monitoreo en tiempo real**: procesa la posición de embarcaciones en la costa para apoyar decisiones de seguridad.

En los tres casos, **ArcGIS Enterprise** actúa como infraestructura geoespacial común, mientras que **ArcGIS Data Pipelines** y **ArcGIS Velocity** cumplen el rol de motores de integración batch y de procesamiento en tiempo real, respectivamente.

---

## Objetivo

Demostrar cómo la geohabilitación de pipelines de datos permite integrar y procesar información proveniente de sistemas empresariales, fuentes externas y flujos en tiempo real, mediante procesos batch y streaming, generando información geoespacial confiable para el monitoreo y la toma de decisiones.

**Objetivos específicos:**

- Integrar un sistema ERP con el entorno geoespacial para gestionar y visualizar el ciclo completo de una incidencia de campo, desde su detección hasta su resolución.
- Integrar, transformar y estandarizar información geográfica proveniente de múltiples departamentos y fuentes externas, consolidándola en un único Feature Layer para su análisis y visualización.
- Procesar y visualizar datos en tiempo real sobre la ubicación y movimiento de embarcaciones, generando información geoespacial que apoye el monitoreo y la toma de decisiones.
- Demostrar la integración de ArcGIS Enterprise, ArcGIS Data Pipelines y ArcGIS Velocity como una arquitectura común para el procesamiento de datos tanto batch como en tiempo real.

---

## Concepto: Geohabilitación de datos

**Geohabilitar** un pipeline de datos significa incorporarle una referencia espacial para que la información —sin importar su origen— pueda ubicarse, visualizarse y analizarse sobre el territorio.

Un dato aislado en una tabla de un ERP o en un archivo JSON tiene valor limitado. El mismo dato, geohabilitado, permite:

- Ubicar un evento o un objeto en el espacio.
- Integrarse con otras capas de información geográfica.
- Visualizarse en escenas, mapas y dashboards.
- Convertirse en insumo directo para la toma de decisiones.

La narrativa que conecta a los tres casos de este proyecto sigue el mismo patrón conceptual:

```mermaid
flowchart LR
    A[Problema] --> B[Datos distribuidos o en tiempo real]
    B --> C[Ingeniería de Datos]
    C --> D[Geohabilitación]
    D --> E[Información espacial]
    E --> F[Visualización]
    F --> G[Decisión]
    G --> H[Acción]

    classDef step fill:#eef2ff,stroke:#4f46e5,color:#312e81;
    class A,B,C,D,E,F,G,H step;
```

---

## Arquitectura conceptual

Los tres casos comparten una misma lógica: distintas fuentes alimentan un motor de procesamiento, y el resultado geohabilitado se consume mediante visualizaciones orientadas a la decisión.

```mermaid
flowchart TB
    subgraph FUENTES[Fuentes de datos]
        ERP[ERP empresarial]
        DEP[Departamentos + JSON / archivos]
        RT[Fuente en tiempo real]
    end

    subgraph ENGINE[ArcGIS Enterprise]
        DP[ArcGIS Data Pipelines]
        VEL[ArcGIS Velocity]
    end

    subgraph CONSUMO[Visualización y decisión]
        ESC[Escena web]
        DASH[Dashboard]
        MAPA[Mapa web]
    end

    ERP --> DP
    DEP --> DP
    RT --> VEL

    DP --> ESC
    DP --> DASH
    VEL --> MAPA

    classDef fuente fill:#eef2ff,stroke:#4f46e5,color:#312e81;
    classDef proceso fill:#fff7ed,stroke:#ea580c,color:#7c2d12;
    classDef salida fill:#ecfdf5,stroke:#059669,color:#064e3b;

    class ERP,DEP,RT fuente;
    class DP,VEL proceso;
    class ESC,DASH,MAPA salida;
```

- **ArcGIS Data Pipelines** resuelve los patrones batch y programados: integración del ERP (Caso 1) y de fuentes multi-departamentales (Caso 2).
- **ArcGIS Velocity** resuelve el patrón de tiempo real: posición de embarcaciones (Caso 3).
- **ArcGIS Enterprise** provee la infraestructura común de almacenamiento, publicación y visualización para ambos motores.

---

## Casos de uso

### Caso 1: Gestión de incidencias en luminarias

**Roles involucrados**

| Rol | Necesidad |
|---|---|
| Responsable de Operaciones | Identificar las luminarias afectadas y conocer su estado |
| Técnico de Campo | Localizar la incidencia, atenderla y reportarla como solucionada |

**El reto**

Conectar el ERP con el sistema geoespacial para reflejar cada cambio en el territorio, desde la falla hasta su resolución.

**La solución**

Un pipeline en ArcGIS Data Pipelines ingesta información del ERP: historial de fallas, luminarias, transformadores y técnicos disponibles. El flujo transforma e integra estas fuentes y actualiza los feature layers que alimentan una **escena web** de monitoreo.

Desde esta escena, el responsable de operaciones identifica las luminarias afectadas y coordina la atención con el técnico de campo. Una vez resuelta la incidencia, el técnico la reporta como solucionada y, mediante un **webhook**, el estado se propaga de regreso hacia los feature layers, actualizando automáticamente el visor de monitoreo.

```mermaid
flowchart TD
    ERP[(ERP empresarial)]
    ERP --> HF[Historial de fallas]
    ERP --> LUM[Luminarias]
    ERP --> TRA[Transformadores]
    ERP --> TEC[Técnicos disponibles]

    HF --> DP[ArcGIS Data Pipelines]
    LUM --> DP
    TRA --> DP
    TEC --> DP

    DP -->|Ingesta, transformación e integración| FL[Feature Layers]
    FL --> ESC[Escena web de monitoreo]

    ESC --> RO[Responsable de Operaciones]
    ESC --> TC[Técnico de Campo]
    TC -->|Atiende y reporta solución| WH[[Webhook]]
    WH --> FL

    classDef fuente fill:#eef2ff,stroke:#4f46e5,color:#312e81;
    classDef proceso fill:#fff7ed,stroke:#ea580c,color:#7c2d12;
    classDef salida fill:#ecfdf5,stroke:#059669,color:#064e3b;
    classDef rol fill:#fefce8,stroke:#ca8a04,color:#713f12;

    class ERP,HF,LUM,TRA,TEC fuente;
    class DP,WH proceso;
    class FL,ESC salida;
    class RO,TC rol;
```

**Evidencia visual**

![Escena web con el estado de las luminarias](www/caso-01/escena-luminarias.png)

*Escena web utilizada por el responsable de operaciones para monitorear el estado de las luminarias.*

![Visor de estadísticas de integración del ERP](www/caso-01/integracion-erp.png)

![Pipeline desarrollado en ArcGIS Data Pipelines](www/caso-01/pipeline-data-pipelines.png)

![Arquitectura del Caso 1](www/caso-01/arquitectura.png)

---

### Caso 2: Integración y estandarización de proyectos nacionales

**Roles involucrados**

| Rol | Necesidad |
|---|---|
| Coordinador de Proyectos | Conocer diariamente el estado de los proyectos a nivel nacional |
| Especialista en Geomática e Ingeniería de Datos | Integrar y estandarizar la información de los distintos departamentos |

**El reto**

La información de proyectos está distribuida entre ocho departamentos —Obras Públicas, Patrimonio, Riego, Gestión Ambiental, Servicios Sociales, Planificación Territorial, Desarrollo Económico y Participación Ciudadana— además de fuentes externas como archivos JSON. Cada fuente utiliza su propia estructura, lo que dificulta consolidar, estandarizar y automatizar una visión nacional de los proyectos.

**La solución**

Un flujo en ArcGIS Data Pipelines ingesta la información geográfica de los ocho departamentos y de fuentes externas, estandariza sus estructuras y atributos, y actualiza diariamente un **feature layer consolidado**. A partir de esta fuente única se alimenta un **dashboard** de monitoreo nacional.

```mermaid
flowchart TD
    subgraph FUENTES[Fuentes departamentales]
        OP[Obras Públicas]
        PAT[Patrimonio]
        RIE[Riego]
        AMB[Gestión Ambiental]
        SS[Servicios Sociales]
        PT[Planificación Territorial]
        DE[Desarrollo Económico]
        PC[Participación Ciudadana]
    end
    JSON[(JSON y otras fuentes externas)]

    OP --> DP[ArcGIS Data Pipelines]
    PAT --> DP
    RIE --> DP
    AMB --> DP
    SS --> DP
    PT --> DP
    DE --> DP
    PC --> DP
    JSON --> DP

    DP -->|Estandarización e integración| FL[Feature Layer consolidado]
    FL --> DASH[Dashboard nacional]
    DASH --> CP[Coordinador de Proyectos]

    classDef fuente fill:#eef2ff,stroke:#4f46e5,color:#312e81;
    classDef proceso fill:#fff7ed,stroke:#ea580c,color:#7c2d12;
    classDef salida fill:#ecfdf5,stroke:#059669,color:#064e3b;
    classDef rol fill:#fefce8,stroke:#ca8a04,color:#713f12;

    class OP,PAT,RIE,AMB,SS,PT,DE,PC,JSON fuente;
    class DP proceso;
    class FL,DASH salida;
    class CP rol;
```

**Evidencia visual**

![Dashboard de monitoreo nacional de proyectos](www/caso-02/dashboard-proyectos.png)

*Dashboard alimentado diariamente por el feature layer consolidado.*

![Pipeline de integración completo](www/caso-02/pipeline-integracion.png)

![Arquitectura del Caso 2](www/caso-02/arquitectura.png)

---

### Caso 3: Monitoreo de embarcaciones en tiempo real

**Roles involucrados**

| Rol | Necesidad |
|---|---|
| Responsable de Seguridad Nacional | Monitorear en tiempo real las embarcaciones para detectar posibles actividades ilícitas |
| Arquitecto de Soluciones | Integrar los datos mediante un pipeline en tiempo real |

**El reto**

Visualizar y monitorear en un mapa web la trayectoria de las embarcaciones en la costa ecuatoriana, con el fin de apoyar la seguridad y la toma de decisiones.

**La solución**

El flujo se implementó con ArcGIS Velocity, que recibe de forma continua la ubicación de las embarcaciones, la procesa y mantiene actualizada su representación geoespacial. La información resultante alimenta un **mapa web** donde se observa la ubicación, el movimiento y la trayectoria de cada embarcación conforme se actualiza en tiempo real.

```mermaid
flowchart TD
    RT[(Fuente de datos en tiempo real)]
    RT --> VEL[ArcGIS Velocity]

    VEL -->|Ingesta, procesamiento y actualización continua| GEO[Información geoespacial actualizada]
    GEO --> MAPA[Mapa web]
    MAPA --> RSN[Responsable de Seguridad Nacional]

    AS[Arquitecto de Soluciones] --- VEL

    classDef fuente fill:#eef2ff,stroke:#4f46e5,color:#312e81;
    classDef proceso fill:#fff7ed,stroke:#ea580c,color:#7c2d12;
    classDef salida fill:#ecfdf5,stroke:#059669,color:#064e3b;
    classDef rol fill:#fefce8,stroke:#ca8a04,color:#713f12;

    class RT fuente;
    class VEL proceso;
    class GEO,MAPA salida;
    class AS,RSN rol;
```

**Evidencia visual**

![Mapa web con la trayectoria de las embarcaciones](www/caso-03/mapa-embarcaciones.png)

*Mapa web con la ubicación y trayectoria de las embarcaciones, actualizado en tiempo real.*

![Flujo desarrollado en ArcGIS Velocity](www/caso-03/velocity-pipeline.png)

![Arquitectura del Caso 3](www/caso-03/arquitectura.png)

---

### Comparación de los tres casos

| **Caso** | **Necesidad** | **Patrón de datos** | **Tecnología** | **Flujo de procesamiento** | **Resultado** |
|---|---|---|---|---|---|
| **1 · Luminarias** | Gestión del ciclo de incidencias en campo | Batch + eventos | ArcGIS Data Pipelines | ERP → ETL → Feature Layer → Webhook | Escena web actualizada |
| **2 · Proyectos nacionales** | Integración y homogeneización de múltiples fuentes | Batch programado | ArcGIS Data Pipelines | Feature Layers + JSON → ETL → Feature Layer consolidado | Dashboard de monitoreo |
| **3 · Embarcaciones** | Monitoreo de ubicación y movimiento | Streaming / tiempo real | ArcGIS Velocity | Feed → procesamiento continuo → salida geoespacial | Mapa web en tiempo real |

---

## Arquitectura e infraestructura

Los tres casos operan sobre una misma infraestructura geoespacial empresarial. Esta sección describe, de forma conceptual, cómo se organiza esa infraestructura.

### ArcGIS Enterprise: arquitectura base

Plataforma geoespacial empresarial que centraliza el almacenamiento, la publicación de servicios y la gestión de contenido, y sirve como base sobre la cual operan tanto los procesos batch como los de tiempo real.

![Arquitectura base de ArcGIS Enterprise](www/arquitectura/enterprise-base.png)

### Enterprise + Data Pipelines

Incorpora la capacidad de diseñar flujos de ingesta, transformación e integración de datos —batch y programados— directamente sobre la infraestructura de Enterprise, publicando los resultados como feature layers listos para consumo.

![Arquitectura de ArcGIS Enterprise con Data Pipelines](www/arquitectura/enterprise-data-pipelines.png)

### Enterprise + Velocity

Añade la capacidad de recibir, procesar y actualizar información en tiempo real, manteniendo sincronizada la representación geoespacial de fenómenos dinámicos.

![Arquitectura de ArcGIS Enterprise con Velocity](www/arquitectura/enterprise-velocity.png)

### Arquitectura integrada

Combina Enterprise, Data Pipelines, Velocity e Image Server en una sola plataforma capaz de atender distintos patrones de datos —batch, programado, empresarial, geográfico, tiempo real y procesamiento de imágenes— sin generar silos tecnológicos.

![Arquitectura integrada de la plataforma geoespacial](www/arquitectura/enterprise-integrated.png)

---

## Patrones de procesamiento

| Patrón | Descripción | Casos relacionados |
|---|---|---|
| **Batch** | Procesamiento de datos en lotes, ejecutado de forma periódica | Caso 1 · Caso 2 |
| **Automatización** | Flujos que actualizan la información sin intervención manual repetitiva | Caso 1 · Caso 2 |
| **Integración de fuentes** | Consolidación de datos provenientes de sistemas y formatos distintos (ERP, feature layers departamentales, JSON) | Caso 1 · Caso 2 |
| **Tiempo real** | Procesamiento continuo de eventos para mantener la información siempre actualizada | Caso 3 |

---

## Resultados y valor generado

| Resultado | Caso 1 · Luminarias | Caso 2 · Proyectos | Caso 3 · Embarcaciones |
|---|:---:|:---:|:---:|
| Integración de fuentes heterogéneas | ✔️ | ✔️ | |
| Automatización de procesos | ✔️ | ✔️ | |
| Estandarización de información | | ✔️ | |
| Centralización de datos | | ✔️ | |
| Actualización de información geoespacial | ✔️ | ✔️ | ✔️ |
| Visualización operacional | ✔️ | ✔️ | ✔️ |
| Procesamiento en tiempo real | | | ✔️ |
| Soporte a la toma de decisiones | ✔️ | ✔️ | ✔️ |
| Conexión entre sistemas de datos y el territorio | ✔️ | ✔️ | ✔️ |

---

## Conclusiones

Los tres casos demuestran cómo una arquitectura de datos geoespaciales puede responder a distintas necesidades y velocidades de información: gestionar incidencias de campo, integrar información de múltiples áreas y monitorear eventos en tiempo real.

La geohabilitación aporta el contexto espacial a los datos, permitiendo pasar de información dispersa y aislada a información integrada, contextualizada y útil para la toma de decisiones.

ArcGIS Data Pipelines permite automatizar los procesos de integración y transformación batch, mientras que ArcGIS Velocity incorpora la ingesta y el procesamiento de datos en tiempo real.

En conjunto, estas capacidades pueden integrarse dentro del ecosistema ArcGIS Enterprise, conformando una arquitectura geoespacial capaz de soportar procesos batch y streaming para diferentes necesidades operativas.

El valor no está únicamente en integrar los datos, sino en agregarles contexto espacial y entregarlos con la velocidad necesaria para convertirlos en decisiones.

> **El dato informa. El territorio contextualiza. La Ingeniería de Datos lo conecta.**

---

## Estructura del repositorio

```text
.
├── README.md
│
└── www/
    ├── caso-01/
    │   ├── escena-luminarias.png
    │   ├── integracion-erp.png
    │   ├── pipeline-data-pipelines.png
    │   └── arquitectura.png
    │
    ├── caso-02/
    │   ├── pipeline-integracion.png
    │   ├── dashboard-proyectos.png
    │   └── arquitectura.png
    │
    ├── caso-03/
    │   ├── velocity-pipeline.png
    │   ├── mapa-embarcaciones.png
    │   └── arquitectura.png
    │
    └── arquitectura/
        ├── enterprise-base.png
        ├── enterprise-data-pipelines.png
        ├── enterprise-velocity.png
        └── enterprise-integrated.png
```

---

## Documentación oficial

Recursos oficiales de Esri para profundizar en las tecnologías utilizadas en este proyecto:

### ArcGIS Data Pipelines

- [Documentación técnica de ArcGIS Data Pipelines](https://doc.esri.com/es/arcgis-data-pipelines/latest/index.html) — instalación, primeros pasos y tareas esenciales.
- [Recursos y tutoriales de ArcGIS Data Pipelines](https://www.esri.com/es-es/arcgis/products/arcgis-data-pipelines/resources) — tutoriales, preguntas frecuentes y blog del producto.

### ArcGIS Velocity

- [Documentación técnica de ArcGIS Velocity](https://doc.esri.com/en/arcgis-velocity) — configuración, primeros pasos y tareas esenciales (en inglés).
- [Recursos y tutoriales de ArcGIS Velocity](https://www.esri.com/es-es/arcgis/products/arcgis-velocity/resources) — tutoriales, opciones de implementación y comunidad.