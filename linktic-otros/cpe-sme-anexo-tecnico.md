# CPE — Anexo técnico SM&E (Computadores para Educar)

> Ruta local: `/Users/johnhernandez/Documents/Linktic/CPE`

## Descripción / propósito

La carpeta **no contiene código**: solo un documento, `ANEXO 06. ANEXO TÉCNICO (2) (1).pdf` (37 páginas, ~639 KB), de **Computadores para Educar (CPE)**, Oficina Asesora de Planeación, Bogotá, febrero de 2022.

Título: *Condiciones técnicas para el desarrollo e implementación del módulo de analítica y reportes de información e integración del Sistema de Monitoreo y Evaluación para la Innovación Educativa SM&E (Fase II del proyecto)*.

Es el anexo técnico de un proceso de contratación (probablemente referencia para una propuesta o para un proyecto similar de analítica/reportería).

## Contenido del anexo

- **Contexto**: política *Tecnologías para Aprender* (CONPES 3988, 2020), cuarto pilar "Fortalecimiento del monitoreo y evaluación del uso, acceso e impacto de las tecnologías digitales en la educación". Entidades del ecosistema: MEN, MinTIC, DANE, ICFES, entre otras.
- **Alcance de la Fase II**:
  1. Entender y articular la Fase I (módulos transaccionales, agente de monitoreo).
  2. Refinar, desarrollar y poner en producción el **módulo de analítica**.
  3. Diseñar y desarrollar un **módulo de reportes e indicadores (dashboards)**: Índice de Innovación Educativa (MEN) e Índice de Evolución Digital (CPE), con dimensiones, indicadores, línea base y visualización geográfica; capacidad de definir nuevos indicadores.
  4. Integración e interoperabilidad con el **Observatorio Colombiano de Innovación Educativa con Uso de TIC** y otras fuentes.
  5. Soporte técnico y mantenimientos; 6. apropiación y gestión del cambio; 7. nuevos requerimientos.
- **Módulos del SM&E**: agente de monitoreo y evaluación, encuestas, administración del sistema, reportes e indicadores.
- **Arquitectura**: orientada a servicios (SOA / microservicios / API), arquitectura cloud ya implementada, marco de interoperabilidad de MinTIC.
- **Metodología**: ágil (células / Agile Teams).
- **Equipo mínimo**: gerente de proyecto y perfiles de ingeniería; certificaciones Azure (Administrator, Developer, DevOps Engineer, Solutions, Fundamentals) entre las valoradas.
- **Plazo**: desde el acta de inicio hasta el 31 de diciembre de 2022. El contratista provee ambientes de desarrollo y pruebas; CPE provee producción.
- Secciones finales: obligaciones generales y específicas, forma de pago y garantías.

## Stack y versiones

No aplica (documento). Del anexo se infiere un entorno cloud Azure y servicios vía API/JSON.

## Arquitectura y estructura de carpetas

```
CPE/
  ANEXO 06. ANEXO TÉCNICO (2) (1).pdf
```

## Funcionalidades / endpoints / pantallas

No aplica.

## Datos e integraciones

Descritas en el documento (Observatorio de Innovación Educativa, entidades del ecosistema).

## Cómo ejecutarlo localmente

No aplica.

## Variables de entorno

Ninguna.

## Pruebas

No aplica.

## Estado git

No versionado (solo bajo el repositorio del home, sin commits).

## Notas

Por su tema (módulo de analítica y reportes de indicadores), es material de referencia afín a la línea de reportería del equipo (generador dinámico de reportes, tableros BI).
