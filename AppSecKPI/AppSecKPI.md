# AppSec KPIs & Metrics
### [English Version](./AppSecKPI-EN.md)


- En AppSec, muchas veces se invierte tiempo en implementar controles y procesos, pero no se mide adecuadamente su impacto. Sin métricas claras, es difícil demostrar valor, justificar decisiones o identificar oportunidades de mejora.
- En este articulo quiero compartir algunas de las métricas clave que estuve explorando dentro del programa de Security Champions, basadas en experiencia real y aplicadas con datos concretos.

## KPIs fundamentales en AppSec

1. Number of Active Critical Vulnerabilities (NACV)

- Este KPI evalúa el riesgo activo para el negocio basado en la cantidad de vulnerabilidades críticas abiertas en un momento dado. Permite responder a preguntas como: ¿Estamos mejorando o acumulando deuda tecnica? ¿Cuántas vulnerabilidades críticas tenemos abiertas? ¿Cuántas de ellas están en producción? ¿Cuántas de ellas están en producción y no tienen un plan de remediación? etc.

2. Time to Remedy (TTR)

- Mide el tiempo promedio entre la creación de una vulnerabilidad y su remediación. Refleja la eficiencia del equipo en responder a los hallazgos y minimizar el tiempo de exposición.
- Este KPI es fundamental para entender la velocidad de respuesta del equipo ante vulnerabilidades.
- Nos deja identificar cuellos de botella en el proceso de remediación y evaluar la efectividad de las capacitaciones o revisiones de código. Un TTR bajo indica un equipo ágil y proactivo, mientras que un TTR alto puede señalar problemas en la gestión de vulnerabilidades o falta de recursos.
- la ecuación para medir un TTR bajo es la siguiente: **TTR = (Fecha_Resuelto - Fecha_Creación).days**
    - Esto es un aproximado, la realidad es que depende de la lógica de negocio por defondo de cada empresa, pero en general se espera que el TTR sea bajo.

3. Number of Vulnerabilities per Team (NVT)

- Este indicador nos ayuda a identificar en qué equipos o plataformas se concentran las vulnerabilidades, lo que permite priorizar capacitaciones, revisiones o refactorizaciones.
- Un NVT alto en un equipo específico puede indicar la necesidad de una revisión más profunda de su código o procesos. También puede señalar la falta de capacitación o recursos en ese equipo.
- Este KPI es útil para identificar patrones en la aparición de vulnerabilidades y para dirigir esfuerzos de capacitación o mejora de procesos hacia los equipos que más lo necesitan.
- En resumen que equipo necesita más ayuda y cual es su grado de madurez en cuanto a seguridad.

4. Time to Remedy per Criticity (TTRC)

- Similar al TTR, pero desglosado por nivel de severidad. No todas las vulnerabilidades deberían tener el mismo tiempo de respuesta. Este KPI permite entender si las más críticas están siendo tratadas con la urgencia adecuada.
- Estas metricas ya dependen mucho de la cultura de la empresa y el equipo, pero en general se espera que las vulnerabilidades críticas tengan un TTR mucho más bajo que las de baja severidad.
    - Umbrales ideales:

        - Critical: ≤ 30 días
        - High/Major: ≤ 90 días
        - Normal/Medium: ≤ 120 días
        - Low: ≤ 180 días o incluso un año

### Implementaciones tipicas
- Todos estos KPIs pueden calcularse a partir de queries con jira, confluence, o con cualquier herramienta de ticketing que use la empresa.
- También se pueden implementar dashboards en herramientas como Grafana o Kibana para visualizar estos KPIs en tiempo real.
- Para mí la mejor forma es poder extraer la información en un CSV de forma que se pueda extraer la información de diferentes fuentes para tenerla en el mismo formato o si diferentes fuentes tienen diferentes formatos, poder estandarizarlo.
- Con `pandas` podemos combinar y transformar y filtrar los datos.
- Podemos crear graficos con `matplotlib` para visualizar los KPIs.


### Para tener en cuenta
- Estas metricas **no sirven si no hay suficientes datos** o si están incompletos o mal estructurados.
- Por eso es importante definir un proceso claro para la recolección y almacenamiento de datos, así de esta manera podemos asegurar que los KPIs sean precisos y útiles.
 - Generalmente en los programas de security champions y SSDLC se determina una politica de seguridad y se define un proceso claro para la recolección y almacenamiento de datos, así de esta manera podemos asegurar que los KPIs sean precisos y útiles.

## Ejemplo de implementación
 - [Repositorio](https://github.com/G4sp4rCS/AppSec-KPI-metrics)
 - Hice una pequeña implementación en formato de script con fines demostrativos.