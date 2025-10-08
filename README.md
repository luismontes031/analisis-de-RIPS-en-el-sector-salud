# 🩺 Análisis de Datos del Sector Salud  
**Proyecto: RIPS y Auditoría**

El conjunto de datos contiene **más de 2000 registros** distribuidos en **7 tablas** relacionadas con usuarios, servicios, facturación y auditoría.  

Se utilizaron **Excel** y **Power BI** como principales herramientas de trabajo:

- **Excel:** limpieza, transformación, normalización y cruces de datos.  
- **Power BI:** creación de gráficos y paneles de análisis más complejos.

---

## 🧹 Limpieza y transformación de datos

Se realizó un proceso completo de depuración y estructuración:

**1. Cambio de columna `id_consulta` y los demas id_identificadores unicos a `general`**  
Se estandarizó el identificador principal para facilitar los cruces entre módulos.

**Motivos para usar enteros (INT o BIGINT) como llave primaria (PK):**
- Mayor **rendimiento**, ya que los enteros son más rápidos de comparar e indexar.  
- Menor **espacio de almacenamiento** que un texto (`VARCHAR`).  
- **Simplicidad** al manejar consecutivos autoincrementales (1, 2, 3...).


## 🔤 Estandarización de columnas

- Se transformaron todas las columnas según su tipo de dato.  
- Se eliminaron **espacios ocultos** usando la función `ESPACIOS`.  
- Se convirtieron los textos a **minúsculas** y se eliminaron tildes:

```excel
=MINUSC(SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(A2;"á";"a");"é";"e");"í";"i");"ó";"o");"ú";"u"))
En la columna valorfactura se reemplazó el carácter raro “≠” por texto descriptivo:

excel

=SUSTITUIR(A2;"≠";"diferente a")
📅 Validaciones y correcciones
Se validó que la fecha de ingreso sea siempre menor que la fecha de egreso (hospitalización):

excel

=SI(C2>D2;"Error";"OK")
Se concatenaron campos de tipo identificador (cc o id) con el número de documento:

excel

=CONCATENAR(A2;"-";B2)
Esto permitió reducir columnas y facilitar el manejo de identificadores únicos.

💰 Cruce entre facturación y servicios
Se detectó que la columna NumFactura no existía en los módulos de consulta, procedimientos, medicamentos y hospitalización.
Es importante incluirla para validar y cruzar información entre lo clínico y lo financiero.

Facturación: contiene ID_usuario + NumFactura

Otros módulos: solo tienen ID_usuario

Objetivo: llevar la columna NumFactura a los demás módulos, haciendo coincidir por ID_usuario.

excel

=SI.ERROR(ÍNDICE(AF_Facturacion!A:A;COINCIDIR(AF_A2;AF_Facturacion!B:B;0));"F0000")
Luego se reemplazaron valores no numéricos:

excel

=SI(ESNUMERO(A2);A2;0)
Y finalmente se clasificaron las facturas según asociación:

excel

=SI(O(A2="";A2=0);"Sin Usuario";"Con Usuario")
🧾 Análisis de facturas huérfanas y reales
Se filtraron las facturas sin usuario (huérfanas) y se creó una nueva hoja con esos registros.

Se generó una tabla dinámica y un gráfico de columnas para calcular el porcentaje del valor de facturas huérfanas por módulo.

Se repitió el proceso con las facturas reales (con usuario).

Fórmulas para cálculos agregados:
Facturas reales:

excel

=SUMAR.SI(facturas_reales!C:C;"CON USUARIO";facturas_reales!B:B)
Facturas huérfanas:

excel

=SUMAR.SI(facturas_huerfanas!C:C;"SIN USUARIO";facturas_huerfanas!B:B)
Con estos resultados se generó un resumen comparativo para visualizar la diferencia entre facturación real y facturación huérfana.
en excel se creo tabla dinamica y se represento en grafica:
Se construyeron gráficos comparativos que muestran:

Porcentaje de facturas huérfanas por módulo.

Distribución del valor total por tipo de usuario.

Proporción entre facturas reales y huérfanas.

📊 Análisis en Power BI
En Power BI se importó la hoja facturas_huerfanas, que contiene los registros de facturas no asociadas a ningún usuario.
Estas presentan riesgo de glosa por falta de correspondencia con un paciente.
En este proyecto se desarrollaron varias medidas DAX que permiten analizar las **facturas huérfanas** por usuario, identificando los módulos implicados, la cantidad de servicios asociados y la facturación total por cada caso. A continuación se detallan las medidas y su propósito:

---

### 🔹 `Resumen_Huerfanas_Detallado`

```DAX
Resumen_Huerfanas_Detallado =
ADDCOLUMNS(
    SUMMARIZE(
        facturas_huerfanas,
        facturas_huerfanas[ID_USUARIO]
    ),
    "Modulos_Lista",
        CONCATENATEX(
            DISTINCT(
                SELECTCOLUMNS(
                    FILTER(
                        facturas_huerfanas,
                        facturas_huerfanas[ID_USUARIO] = EARLIER(facturas_huerfanas[ID_USUARIO])
                    ),
                    "Modulo", facturas_huerfanas[MODULO]
                )
            ),
            [Modulo],
            " + ",
            [Modulo],
            ASC
        ),
    "Cantidad_Modulos",
        CALCULATE(
            DISTINCTCOUNT(facturas_huerfanas[MODULO]),
            facturas_huerfanas[ID_USUARIO] = EARLIER(facturas_huerfanas[ID_USUARIO])
        )
)
Descripción:
Esta medida crea una tabla resumen por usuario que muestra:

🔸 Modulos_Lista: una lista concatenada con los nombres de los módulos donde el usuario presenta facturas huérfanas (por ejemplo: Consultas + Procedimientos + Medicamentos).

Se construye con CONCATENATEX, que une los nombres de los módulos de manera ordenada y separada por “+”.

🔸 Cantidad_Modulos: calcula el número de módulos distintos asociados al mismo usuario usando DISTINCTCOUNT.

Con esta estructura es posible visualizar de forma resumida qué tan dispersas están las facturas de cada usuario entre los diferentes módulos del sistema.

🔹 Clasificacion
DAX
Copiar código
Clasificacion =
SWITCH(
    TRUE(),
    [Cantidad_Modulos] = 1, "Un solo módulo",
    [Cantidad_Modulos] = 2, "Dos módulos",
    [Cantidad_Modulos] = 3, "Tres módulos",
    [Cantidad_Modulos] >= 4, "Cuatro o más módulos"
)
Descripción:
Esta medida clasifica a cada usuario según la cantidad de módulos en los que presenta facturas huérfanas.
Permite segmentar fácilmente los casos en categorías analíticas:

Cantidad de módulos	Clasificación
1	Un solo módulo
2	Dos módulos
3	Tres módulos
4 o más	Cuatro o más módulos

De esta forma se facilita la priorización de casos complejos (usuarios con múltiples módulos afectados).

🔹 Facturacion_Total
DAX
Copiar código
Facturacion_Total =
SUM(facturas_huerfanas[VALOR_FACTURA])
Descripción:
Suma el valor total de las facturas huérfanas por usuario o por conjunto de registros seleccionados.
Esta medida se utiliza para identificar la magnitud económica del problema y apoyar la toma de decisiones financieras o auditorías internas.

conclusiones finales :



