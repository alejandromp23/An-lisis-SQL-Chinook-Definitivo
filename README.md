# SQL Music Store Analysis

## Objetivo
Análisis de datos de una tienda musical utilizando SQL y MySQL.

---

## Tecnologías utilizadas
- SQL
- MySQL

---

# Promedio de precios

SELECT AVG(Total) AS 'Precio promedio'
	FROM invoice;
---

### Insight

El precio promedio equivale a 5.65.

---

# Promedio de precios por país

SELECT BillingCountry AS 'País', AVG(Total) AS 'Precio promedio'
	FROM invoice
	GROUP BY BillingCountry
	ORDER BY `Precio promedio` DESC;
---

### Insight

El precio promedio por país pone en primer lugar a Chile con un precio de 6.7 
y, en último lugar, a Argentina con un precio de 5.4.

---

# ¿Qué países ingresan más dinero?

SELECT BillingCountry AS 'País', SUM(Total) AS 'Total ingresado',
    RANK() OVER(ORDER BY SUM(Total) DESC) AS 'Ranking ventas'
    FROM invoice
    GROUP BY BillingCountry
    LIMIT 10;
---

### Insight 

El país con mayores ingresos ha sido USA, con un total de 523, seguido por Canadá, con 303.

![Países con más ingresos](Screenshots/Ingresos_País.jpeg)

---

# ¿Cuáles han sido los ingresos por año?

SELECT YEAR(InvoiceDate) AS 'Año', COUNT(*) AS 'Número de facturas',
    SUM(total) AS 'Total ingresado',
    SUM(SUM(total)) OVER (ORDER BY YEAR(InvoiceDate) ASC) AS 'Ingreso Acumulado'
    FROM invoice
    GROUP BY YEAR(InvoiceDate);
---

### Insight 

Encontramos resultados muy pares entre los distintos años expuestos. Además, aunque la cantidad vendida
sea la misma, o muy cercana (en el caso de 2025), los ingresos varian en relación a los años. Esto se debe a que
los precios unitarios varian, es decir, las cantidades vendidas varian en relación a los artistas y, como sabemos, 
tenemos distintos precios según artista.


![Ingresos por año](Screenshots/Ingresos_Año.jpeg)

---

# ¿Qué clientes han gastado más dinero que el promedio de gasto total?

WITH clientes AS( 
SELECT c.customerId AS 'ID', CONCAT(c.LastName, ', ', c.FirstName) AS 'Cliente', 	c.city AS 'Ciudad', COUNT(i.invoiceId) AS 'Compras', 
	SUM(i.total) AS 'Total gastado',
	RANK() OVER (ORDER BY SUM(i.total) DESC) AS 'Ranking gasto'
	FROM customer c INNER JOIN invoice i
        ON c.customerId  = i.customerId
        GROUP BY c.customerId, CONCAT(c.LastName, ', ', c.FirstName)
)

SELECT * FROM clientes
     WHERE `Total gastado` > (SELECT AVG(`Total gastado`)
				FROM clientes);
---

### Insight

En esta consulta encontramos que los clientes con un gasto superior al promedio total han comprado
una cantidad de 7, con un gasto máximo de 49.62 y un mínimo de 39.62 entre los clientes expuestos.

---

# ¿Cuáles son los géneros musicales más vendidos?

SELECT g.genreId AS 'ID', g.Name AS 'Género musical', COUNT(il.invoiceLineId) AS 'Ventas', SUM(il.UnitPrice * il.Quantity) AS 'Total ingresado',
 RANK () OVER(ORDER BY COUNT(il.invoiceLineId) DESC) AS 'Ranking ventas'
 FROM genre g INNER JOIN track t
   ON g.genreId  = t.genreId
   INNER JOIN invoiceLine il
   ON il.trackId = t.trackId
   GROUP BY g.genreId, g.name
   LIMIT 10;
    
Si nos fijamos, encontramos que el género número 7 del ranking (TV Shows) se encuentra en la misma 
situacion anterior que el artista 'Lost': recauda mayores ingresos que otros géneros superiores en ventas.
Por tanto, al igual que antes, este hecho se debe al tener un valor superior del precio unitario de venta.
---

### Insight

El género de Rock destaca claramente en ventas en comparación con el resto de géneros. De hecho, supera 
el doble de ventas del segundo género (Latin). Lo mismo ocurre entre el top 4 y el top 5.
Esto nos demuestra que los 4 géneros con más ventas ocupan la mayor parte de la cuota total del mercado.

![Géneros con más ventas](Screenshots/G_Ventas.jpeg)

---

# ¿Cuáles son los artistas más vendidos?

SELECT ar.ArtistId AS 'ID', ar.Name AS 'Artista', COUNT(il.invoiceLineId) AS 'Ventas', SUM(il.UnitPrice * il.Quantity) AS 'Total ingresado',
    RANK() OVER(ORDER BY COUNT(il.invoiceLineId) DESC) AS 'Ranking ventas'
    FROM artist ar INNER JOIN album al
    ON ar.artistId  = al.artistId
    INNER JOIN track t
    ON al.albumId = t.albumId
    INNER JOIN invoiceline il
    ON t.trackId = il.trackId
    GROUP BY ar.ArtistId, ar.name
    LIMIT 10;
    
Podemos apreciar como el Artista 'Lost' se encuentra en el puesto número 8 del ranking
y aun así obtiene mayores ingresos que el artista que se encuentra en el top 5 de ventas.
Esto se debe a que el precio de venta unitario del artista 'Lost' es mayor y, por tanto,
su recaudación es superior al número 5 del ranking de ventas.

Aquí podemos ver los precios de venta unitarios que sean iguales o mayores en relación a 'Lost':

WITH precio_artista AS (
SELECT ar.ArtistId AS 'ID', ar.name AS 'Artista', il.unitprice AS 'Precio Unitario'
    FROM artist ar INNER JOIN album al
    ON ar.artistId  = al.artistId
    INNER JOIN track t
    ON al.albumId = t.albumId
    INNER JOIN invoiceline il
    ON t.trackId = il.trackId
    GROUP BY ar.artistId, ar.name, il.UnitPrice
)

SELECT * FROM precio_artista
	WHERE `Precio unitario` >= (SELECT `Precio unitario` FROM precio_artista
					WHERE artista LIKE 'Lost');
                                    
Como podemos observar, no encontramos a ningún artista que se encuentre en el top 5-7 
que tenga un precio unitario superior al de 'Lost'. Por tanto, conocemos la causa 
por la cual 'Lost' recauda mayores ingresos en comparación con algunos artistas más vendidos.

Aquí podemos ver el artista top por país:

WITH top_artista AS(
SELECT i.billingcountry AS 'País', ar.ArtistId AS 'ID', ar.Name AS 'Artista', COUNT(il.invoiceLineId) AS 'Ventas', SUM(il.UnitPrice * il.Quantity) AS 'Total ingresado',
    RANK() OVER(PARTITION BY i.billingcountry ORDER BY COUNT(il.invoiceLineId) DESC) AS 'Ranking ventas'
    FROM artist ar INNER JOIN album al
    ON ar.artistId  = al.artistId
    INNER JOIN track t
    ON al.albumId = t.albumId
    INNER JOIN invoiceline il
    ON t.trackId = il.trackId
    INNER JOIN invoice i
    ON i.invoiceId = il.invoiceId
    GROUP BY i.BillingCountry, ar.artistId, ar.name
)

SELECT País, ID, Artista, Ventas, `Total ingresado` FROM top_artista
	WHERE `ranking ventas` = 1
        ORDER BY país;
---

### Insight

Podemos observar como entre los 10 artistas más vendidos hay una fuerte presencia de precios unitarios
más baratos (solo hay 1 artista con un precio unitario más elevado: 'Lost') en comparación con el total de precios de otros
artistas. Deberíamos averiguar si la curva de la demanda es inelástica, para ver si el precio es lo que incentiva a
las personas a comprar, o es el propio artista. Sería interesante analizar la sensibilidad al precio antes 
de tomar decisiones sobre cambios de precio.
Iron Maiden se posiciona como el artista más vendido con una variación superior del 30,1% en relación 
a las ventas segundo (U2). 
Entre el top 4 y el 5 hay una diferencia significativa superior en cantidad de ventas, lo que nos obliga a enfocarnos
en los 4 primeros artistas.

![Artistas con más ventas](Screenshots/Artista_+Ventas.jpeg)
![Artista top por país](Screenshots/Artista_Top.jpeg)

---

## Conclusión

Este proyecto permitió practicar consultas SQL intermedias aplicadas a análisis de negocio.
El análisis muestra que USA es el principal mercado en ingresos, Rock domina claramente las ventas por género y algunos artistas/géneros con menor volumen de ventas generan más ingresos debido a un precio unitario superior.