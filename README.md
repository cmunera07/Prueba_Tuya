# Prueba_Tuya - Ingeniero de datos

## Aspirante: 
	Camilo Andres Munera Montoya


## Objetivo:
	Desarrollar prueba técnica para la convocatoria de Ingeniería de Datos


 ## Tecnologías:
<img src="https://upload.wikimedia.org/wikipedia/commons/c/c3/Python-logo-notext.svg" width="30"> <img src="https://upload.wikimedia.org/wikipedia/commons/d/d0/Google_Colaboratory_SVG_Logo.svg" width="30"> <img src="https://upload.wikimedia.org/wikipedia/commons/8/87/Sql_data_base_with_logo.png" width="30">






## Punto 1 - Procesamiento de archivos HTML en Python



## Punto 2 -  Preferencias de consumo

### 2.1. ¿Qué hace esta consulta?
	
- Une las tablas TRANSACCIONES, CLIENTES y CATEGORIAS_CONSUMO.
- Cuenta el número total de transacciones por categoría para cada cliente.
- Obtiene la última fecha de transacción por categoría.
- Agrupa los datos por cliente y categoría.


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


### 2.2. ¿Qué hace esta consulta?

- Crea la tabla temporal Preferencias que agrupa las transacciones por cliente y categoría.
- Usa la función COUNT() para calcular la cantidad de transacciones por categoría.
- Usa la función MAX() para obtener la última transacción dentro de cada categoría.
- Aplica un ROW_NUMBER() para asignar un ranking basado en el número de transacciones.
- Finalmente, filtra solo la primera categoría preferida (RANKING = 1).
- La variable RANKING puede variar según se desee obtener las N primeras categorías preferidas en las que tuvo más transacciones


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



### 2.3. ¿Qué hace esta consulta?

- Filtra las transacciones dentro de un rango de fechas.
- Permite analizar preferencias de consumo dentro de un período específico.
  

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




### Punto 3 -  Rachas

### 3.1. ¿Qué hace esta consulta?

* TABLA TEMPORAL: SaldosClasificados
	- Clasifica cada cliente según su nivel de saldo.
	- Genera una columna "nivel" que define en qué categoría de deuda se encuentra cada cliente.
	- Asegura que cada mes el cliente esté representado y verifica fecha de retiro en caso de que aplique.
	- Usa la función COALESCE() para asignar "N0" a clientes ausentes.
	- Filtra clientes cuya fecha de retiro es anterior al corte de mes.


			WITH SaldosClasificados AS (
						SELECT 
							h.[identificacion],
							h.[corte_mes],
							h.[saldo],
							CASE 
								WHEN h.[saldo] >= 0 AND h.[saldo] < 300000 THEN 'N0'
								WHEN h.[saldo] >= 300000 AND h.[saldo] < 1000000 THEN 'N1'
								WHEN h.[saldo] >= 1000000 AND h.[saldo] < 3000000 THEN 'N2'
								WHEN h.[saldo] >= 3000000 AND h.[saldo] < 5000000 THEN 'N3'
								WHEN h.[saldo] >= 5000000 THEN 'N4'
							END AS nivel
						FROM [PRUEBA_TUYA].[dbo].[historia$] h
						)
			SELECT 
			    sc.[identificacion],
			    sc.[corte_mes],
			    COALESCE(sc.[nivel], 'N0') AS nivel,
			    r.[fecha_retiro]
			FROM SaldosClasificados sc
			LEFT JOIN [PRUEBA_TUYA].[dbo].[retiros$] r ON sc.[identificacion] = r.[identificacion]
			WHERE (r.[fecha_retiro] IS NULL OR sc.[corte_mes] <= r.[fecha_retiro])
			;



### 3.2. ¿Qué hace esta consulta?

- Usa la función LAG() para validar si el cliente mantiene el mismo nivel que el mes anterior.
- Usa la función DATEDIFF() para validar cuando hay un cambio de racha en los niveles.
- Usa la SUM(rc.cambio_racha) OVER (....) para acumular el número de rachas que se dan en meses consecutivos
- Usa la función HAVING COUNT(r.[corte_mes]) >= N para filtrar solo las rachas que cumplen con el mínimo de meses requerido.



		WITH SaldosClasificados AS (
					SELECT 
						h.[identificacion],
						h.[corte_mes],
						h.[saldo],
						CASE 
							WHEN h.[saldo] >= 0 AND h.[saldo] < 300000 THEN 'N0'
							WHEN h.[saldo] >= 300000 AND h.[saldo] < 1000000 THEN 'N1'
							WHEN h.[saldo] >= 1000000 AND h.[saldo] < 3000000 THEN 'N2'
							WHEN h.[saldo] >= 3000000 AND h.[saldo] < 5000000 THEN 'N3'
							WHEN h.[saldo] >= 5000000 THEN 'N4'
						END AS nivel
					FROM [PRUEBA_TUYA].[dbo].[historia$] h
					)
			SELECT 
				rf.[identificacion],
				rf.[nivel],
				rf.[racha_id],
				COUNT(rf.[corte_mes]) AS total_meses,
				MAX(rf.[corte_mes]) AS fecha_fin
			FROM
				(
					SELECT 
						rc.[identificacion],
						rc.[nivel],
						rc.[corte_mes],
						SUM(rc.cambio_racha) OVER (PARTITION BY rc.[identificacion], rc.[nivel] ORDER BY rc.[corte_mes]) AS racha_id
					FROM
						(
						SELECT 
							sc.[identificacion],
							sc.[corte_mes],
							sc.[nivel],
							LAG(sc.[nivel]) OVER (PARTITION BY sc.[identificacion] ORDER BY sc.[corte_mes]) AS mes_anterior,
							CASE 
							WHEN DATEDIFF(MONTH, LAG(sc.[corte_mes]) OVER (PARTITION BY sc.[identificacion], sc.[nivel] ORDER BY sc.[corte_mes]), sc.[corte_mes]) = 1 
							THEN 0 ELSE 1 END AS cambio_racha
						FROM (
								SELECT 
									sc.[identificacion],
									sc.[corte_mes],
									COALESCE(sc.[nivel], 'N0') AS nivel,
									r.[fecha_retiro]
								FROM SaldosClasificados sc
								LEFT JOIN [PRUEBA_TUYA].[dbo].[retiros$] r ON sc.[identificacion] = r.[identificacion]
								WHERE (r.[fecha_retiro] IS NULL OR sc.[corte_mes] <= r.[fecha_retiro])
							) sc
					) rc
				) rf
			GROUP BY rf.[identificacion], rf.[nivel], rf.[racha_id]
			HAVING COUNT(rf.[corte_mes]) >= 3
		;


### 3.3. ¿Qué hace esta consulta?

- Selecciona la racha más larga de un cliente.
- Usa la función TOP N para devolver las n número de filas de acuerdo con el resultado esperado
- Usa la función WITH TIES para incluir todas las filas con los mejores resultados de acuerdo con la función TOP N y siempre debe ir acompañada de ORDER BY


		WITH SaldosClasificados AS (
					SELECT 
						h.[identificacion],
						h.[corte_mes],
						h.[saldo],
						CASE 
							WHEN h.[saldo] >= 0 AND h.[saldo] < 300000 THEN 'N0'
							WHEN h.[saldo] >= 300000 AND h.[saldo] < 1000000 THEN 'N1'
							WHEN h.[saldo] >= 1000000 AND h.[saldo] < 3000000 THEN 'N2'
							WHEN h.[saldo] >= 3000000 AND h.[saldo] < 5000000 THEN 'N3'
							WHEN h.[saldo] >= 5000000 THEN 'N4'
						END AS nivel
					FROM [PRUEBA_TUYA].[dbo].[historia$] h
					)
		SELECT 
		TOP 2 WITH TIES * 
		FROM (
				SELECT 
					rf.[identificacion],
					rf.[nivel],
					COUNT(rf.[corte_mes]) AS total_meses,
					MAX(rf.[corte_mes]) AS fecha_fin
				FROM
					(
						SELECT 
							rc.[identificacion],
							rc.[nivel],
							rc.[corte_mes],
							SUM(rc.cambio_racha) OVER (PARTITION BY rc.[identificacion], rc.[nivel] ORDER BY rc.[corte_mes]) AS racha_id
						FROM
							(
							SELECT 
								sc.[identificacion],
								sc.[corte_mes],
								sc.[nivel],
								LAG(sc.[nivel]) OVER (PARTITION BY sc.[identificacion] ORDER BY sc.[corte_mes]) AS mes_anterior,
								CASE 
								WHEN DATEDIFF(MONTH, LAG(sc.[corte_mes]) OVER (PARTITION BY sc.[identificacion], sc.[nivel] ORDER BY sc.[corte_mes]), sc.[corte_mes]) = 1 
								THEN 0 ELSE 1 END AS cambio_racha
							FROM (
									SELECT 
										sc.[identificacion],
										sc.[corte_mes],
										COALESCE(sc.[nivel], 'N0') AS nivel,
										r.[fecha_retiro]
									FROM SaldosClasificados sc
									LEFT JOIN [PRUEBA_TUYA].[dbo].[retiros$] r ON sc.[identificacion] = r.[identificacion]
									WHERE (r.[fecha_retiro] IS NULL OR sc.[corte_mes] <= r.[fecha_retiro])
								) sc
						) rc
					) rf
				GROUP BY rf.[identificacion], rf.[nivel], rf.[racha_id]
			) rachafinal
		--WHERE fecha_fin <= '2024-03-31'  -- Filtra por una fecha límite
		ORDER BY total_meses DESC, fecha_fin DESC
		;
