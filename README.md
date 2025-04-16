# Prueba_Tuya


/*
¿Qué hace esta consulta?
- Unir las tablas TRANSACCIONES, CLIENTES y CATEGORIAS_CONSUMO.
- Cuenta el número total de transacciones por categoría para cada cliente.
- Obtiene la última fecha de transacción por categoría.
- Agrupa los datos por cliente y categoría.
*/

SELECT 
    C.[NOMBRE],
    C.[IDENTIFICACIÓN],
    C.[TIPO_DOCUMENTO],
    C.[CLASIFICACION],
    C.[TIPO TARJETA],
    C.[FEÇHA_APERTURA_TARJETA],
    C.[ESTADO_TARJETA],
    CC.[NOMBRE_CATEGORIA],
    T.[CODIGO_CATEGORIA],
    COUNT(T.[ID_TRANSACCION]) AS TOTAL_TRANSACCIONES,
    MAX(T.[FECHA_TRANSACCION]) AS ULTIMA_TRANSACCION
FROM [PRUEBA_TUYA].[dbo].[TRANSACCIONES$] T
JOIN [PRUEBA_TUYA].[dbo].[CLIENTES$] C ON T.[IDENTIFICACION] = C.[IDENTIFICACIÓN]
JOIN [PRUEBA_TUYA].[dbo].[CATEGORIAS_CONSUMO$] CC ON T.[CODIGO_CATEGORIA] = CC.[CODIGO_CATEGORIA]
GROUP BY C.[NOMBRE], C.[IDENTIFICACIÓN], C.[TIPO_DOCUMENTO], C.[CLASIFICACION], 
         C.[TIPO TARJETA], C.[FEÇHA_APERTURA_TARJETA], C.[ESTADO_TARJETA], CC.[NOMBRE_CATEGORIA], T.[CODIGO_CATEGORIA]
;


/*
¿Qué hace esta consulta?
- La subconsulta Preferencias agrupa las transacciones por cliente y categoría.
- Usa COUNT() para calcular la cantidad de transacciones por categoría.
- Usa MAX() para obtener la última transacción dentro de cada categoría.
- Aplica ROW_NUMBER() para asignar un ranking basado en el número de transacciones.
- Finalmente, filtra solo la primera categoría preferida (RANKING = 1).
*/

WITH Preferencias AS (
						SELECT 
							C.[NOMBRE],
							C.[IDENTIFICACIÓN],
							C.[TIPO_DOCUMENTO],
							CC.[NOMBRE_CATEGORIA],
							T.[CODIGO_CATEGORIA],
							COUNT(T.[ID_TRANSACCION]) AS TOTAL_TRANSACCIONES,
							MAX(T.[FECHA_TRANSACCION]) AS ULTIMA_TRANSACCION,
							ROW_NUMBER() OVER (PARTITION BY C.[IDENTIFICACIÓN] ORDER BY COUNT(T.[ID_TRANSACCION]) DESC) AS RANKING
						FROM [PRUEBA_TUYA].[dbo].[TRANSACCIONES$] T
						JOIN [PRUEBA_TUYA].[dbo].[CLIENTES$] C ON T.[IDENTIFICACION] = C.[IDENTIFICACIÓN]
						JOIN [PRUEBA_TUYA].[dbo].[CATEGORIAS_CONSUMO$] CC ON T.[CODIGO_CATEGORIA] = CC.[CODIGO_CATEGORIA]
						GROUP BY C.[NOMBRE], C.[IDENTIFICACIÓN], C.[TIPO_DOCUMENTO], CC.[NOMBRE_CATEGORIA], T.[CODIGO_CATEGORIA]
						)
SELECT 
* 
FROM Preferencias 
WHERE RANKING = 1
;

/*La variable RANKING puede variar según se desee obtener las N primeras categorías preferidas en las que tuvo más transacciones*/


/*
¿Qué hace este ajuste?
- Filtra las transacciones dentro de un rango de fechas.
- Permite analizar preferencias de consumo dentro de un período específico.
*/

WITH Preferencias AS (
						SELECT 
							C.[NOMBRE],
							C.[IDENTIFICACIÓN],
							C.[TIPO_DOCUMENTO],
							CC.[NOMBRE_CATEGORIA],
							T.[CODIGO_CATEGORIA],
							COUNT(T.[ID_TRANSACCION]) AS TOTAL_TRANSACCIONES,
							MAX(T.[FECHA_TRANSACCION]) AS ULTIMA_TRANSACCION,
							ROW_NUMBER() OVER (PARTITION BY C.[IDENTIFICACIÓN] ORDER BY COUNT(T.[ID_TRANSACCION]) DESC) AS RANKING
						FROM [PRUEBA_TUYA].[dbo].[TRANSACCIONES$] T
						JOIN [PRUEBA_TUYA].[dbo].[CLIENTES$] C ON T.[IDENTIFICACION] = C.[IDENTIFICACIÓN]
						JOIN [PRUEBA_TUYA].[dbo].[CATEGORIAS_CONSUMO$] CC ON T.[CODIGO_CATEGORIA] = CC.[CODIGO_CATEGORIA]
						WHERE T.[FECHA_TRANSACCION] BETWEEN '2023-01-01' AND '2023-03-31'
						GROUP BY C.[NOMBRE], C.[IDENTIFICACIÓN], C.[TIPO_DOCUMENTO], CC.[NOMBRE_CATEGORIA], T.[CODIGO_CATEGORIA]
					)
SELECT 
* 
FROM Preferencias WHERE RANKING = 1
;
