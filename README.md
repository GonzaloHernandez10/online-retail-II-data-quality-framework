# :memo: Proyecto: Framework de Calidad y Gobernanza de Datos - Online Retail II
Rol: Data Analyst | Stack: Python (Pandas), Power BI, Framework de Calidad y Gobernanza, Business Intelligence.

# :book: 1. Contexto y Problema de Negocio

La empresa “Online Retail II”, ubicada en Reino Unido, reportó una cifra de ventas brutas entre el 01/12/2009 y el 09/12/2011, pero se sospecha que ese número está "inflado" por errores de sistema y registros contables que no son ventas reales. 

Impacto: Decisiones comerciales basadas en datos inflados, riesgos en la planeación de inventarios y pérdida de credibilidad operativa.

# :dart: 2. Objetivos Estratégicos

Framework de calidad y gobernanza de datos: Crear una estructura lógica, basada en reglas y políticas de negocio que clasifique cada registro del dataset.
Mitigación de Riesgo: Identificar y cuantificar las desviaciones financieras mediante técnicas de limpieza y validación lógica.
Análisis de Causa Raíz (RCA): Clasificar errores (Error Tagging) para explicar al negocio por qué falló la captura de datos (¿Fue una devolución? ¿Fue un ajuste manual? ¿Fue un error de sistema?)
Conciliación financiera: Generar el reporte final (Bridge Report) que concilie el dato bruto inicial con el dato validado. 

# :gear: 3. Arquitectura de Calidad de Datos

A. Reglas de Negocio
Completitud: Campos críticos (Invoice, Description, Quantity,  Price) no pueden ser nulos. Si falla, se asigna el código de error CRITICAL_ERROR. No se consideran ingresos operativos útiles para el análisis.
Integridad de ingresos: Sólo las transacciones con Quantity > 0 y Price > 0 se consideran ingresos operativos útiles para el análisis. Si falla, se asigna el código de error SYSTEM_ADJUSTMENT.
Mitigación de riesgo de precios: Los precios (Price) negativos o en cero son anomalías del sistema, no errores del cliente. Se asigna el código de error INVALID_UNIT_PRICE y su valor se imputa a cero. 
Trazabilidad de devoluciones: Toda transacción que comience con 'C' en Invoice debe ser restada de la venta bruta. Se asigna el código de error COMMERCIAL_CANCELATION.
Deduplicación: Identificación de registros idénticos por campo crítico e Invoice, conservando solo la primera instancia (Timestamp más antiguo).

# :bug: 4. Taxonomía de errores
## Códigos de Error y Acciones

| CÓDIGO DE ERROR          | DESCRIPCIÓN                         | TIPO        | ACCIÓN                 |
|--------------------------|-------------------------------------|-------------|------------------------|
| CRITICAL_ERROR           | Fallo en campo crítico              | Crítico     | Eliminar del análisis  |
| SYSTEM_ADJUSTMENT        | Quantity o Price menor que 0        | Crítico     | Eliminar del análisis  |
| INVALID_UNIT_PRICE       | Price menor que 0                   | No crítico  | Imputar a 0            |
| COMMERCIAL_CANCELATION   | Transacción cancelada               | No crítico  | Segregar               |

# :building_construction: 5. Metodología de Implementación

1. Data Profiling: Diagnóstico inicial del DataSet. Se verifican valores nulos, formatos, campos críticos y valores duplicados.
2. Data Labeling & Categorization: En base a la taxonomía de errores y las valoraciones obtenidas en el Data Profiling, se le asigna a cada registro un código de error, y si es el caso, una categoría. Creación de columnas de control:
  - Error_Code
  - Category
3. Data Cleaning & Imputation: En base a las reglas de negocio, se separan los registros según la validación que aplique sin modificar la fuente de datos original. Obtención de un dataset con Dirty Data.
4. Cálculo de métricas: Recálculo de ventas netas basado en los datos limpios.
5. Obtención de datos para el RCA.
6. Obtención de datos para el Bridge Report.
7. Exportación de los datos.

# :mag_right: 6. Análisis de Causa Raíz (RCA) - Hallazgos
Tras auditar los registros excluidos, se identificaron las siguientes causas que podrían estar provocando la inflación en las ventas netas:
1. Tasa de error transaccional: Se identificaron los siguientes porcentajes los cuales indican el tamaño de errores presentados:
  - Error crítico: El 0.27% de los registros presentan error crítico, lo que indica un problema de punto de venta ya que no se sabe que se está vendiendo ni por cuanto.
  - Ajustes de sistema: El 0.09% de los registros presentan un error de ajuste de sistema. Este error es el más peligroso ya que son cantidades de venta negativas pero sin rastro de cancelación.
2. Cancelaciones: El 1.72% de los registros fueron cancelados, por lo que no se toman como activos estratégicos para el cálculo de ventas netas. 
3. Duplicados: El 0.99% de los registros son valores duplicados. Se considera registro duplicado aquellos que presentan valores duplicados en Invoice, Description, Quantity y Price. Dichos registros no se consideran activos estratégicos para el cálculo de ventas netas. 

# :chart_with_upwards_trend: 7. Impacto Financiero (Bridge Report)
Tras aplicar un proceso de Data Quality al DataSet original y obtener datos válidos y saludables se obtuvo que:

| Hito financiero            | Impacto económico |
|----------------------------|-------------------|
| Ingreso sanitizado         | $10,641,861.33    |
| (-) Cancelaciones          | $893,971.33       |
| Venta neta sanitizada      | $9,747,890.00     |
| (-) Inconsistencias*       | $124.07           |
| Ingreso bruto inicial      | $9,747,765.93     |

*NOTA: Las inconsistencias representan valores duplicados o valores monetarios residuales (venta neta sanitizada - ingreso bruto inicial). Y es la diferencia de conciliación por remoción de ruido técnico.

# :bulb: 8. Resultados y Entregables
A través de este framework de calidad se logró una purificación del 3.07% de los registros (suma de errores, duplicados y ajustes) los cuales inflaban las ventas netas . Esta limpieza no solo mejoró la precisión técnica, sino que permitió una conciliación financiera de la varianza entre el dato bruto y el auditado que quedó justificada y trazada.
DataSet "Golden Standard": DataSet curado listo para consumo en herramientas de visualización como Power BI.
Dataset “Dirty Data”: DataSet con los registros rechazados listos para su auditoría por parte del equipo pertinente. 
DataSet con cancelaciones: DataSet con los registros marcados como operaciones transaccionales cancelas listo para su análisis.  
Bridge Report: Comparativa antes y después de las ventas netas.
Reporte de causa raíz: Indicadores del posible origen de las fallas y errores encontrados en el DataSet original.  

# :arrow_forward: 9. Próximos pasos
Fase 2: Para la fase dos se tiene contemplado la visualización dinámica en Power BI com el objetivo de visualizar: 
1. Análisis de la conciliación financiera con las siguientes métricas clave:
  - Venta bruta sanitizada
  - Monto de devoluciones 
  - Venta neta auditada
El objetivo es cuánto dinero se "evapora" entre la intención de compra y la venta efectiva.
2. Análisis de pareto de la "Fuga de Valor" con las siguientes métricas clave:
  - Ventas netas de las cancelaciones
  - Campo Description

