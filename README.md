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

📊 Análisis en Power BI
En Power BI se importó la hoja facturas_huerfanas, que contiene los registros de facturas no asociadas a ningún usuario.
Estas presentan riesgo de glosa por falta de correspondencia con un paciente.

Se construyeron gráficos comparativos que muestran:

Porcentaje de facturas huérfanas por módulo.

Distribución del valor total por tipo de usuario.

Proporción entre facturas reales y huérfanas.
