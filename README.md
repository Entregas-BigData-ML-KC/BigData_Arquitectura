### Definición de la estrategia del DAaaS

Se plantea realizar como producto final una aplicación web que prediga precios de diferentes discos de vinilo aplicando algoritmos de regresión. El usuario podrá buscar el título de un disco de forma manual o bien realizar una carga de un archivo .csv que contendrá los títulos de disco, entre otros datos, para lograr una predicción de un conjunto de discos.

### Arquitectura DAaaS
#### Fuentes de datos
Se utilizarán las siguientes páginas web para extraer **precios** y algunas estadísticas que podrán ser utilizadas por los algorimos de Machine Learning para calcular y estimar el precio de cada disco

- [https://vinilocoleccion.com/](https://vinilocoleccion.com/)
- [https://www.ebay.es/b/Vinilos-de-musica-edicion-de-- -coleccionista/176985/bn_7036815610](https://www.ebay.es/b/Vinilos-de-musica-edicion-de-coleccionista/176985/bn_7036815610)
- [https://www.fnac.es/s74/Discos-de-vinilo](https://www.fnac.es/s74/Discos-de-vinilo)
- [https://www.discogs.com/es/](https://www.discogs.com/es/)

#### Tecnologías involucradas

- Web Server
	- Nginx
	- Vue
	- Node-Express
- Python Back
	- Flask
	- Implementaciones de algoritmos de regresión en python
- Data Lake
	- Hadoop _ DataProc
- Data Warehouse
	- BigQuery
- Bases de Datos
	- Cloud SQL
- Storage
	- Google Cloud Storage
- Functions
	- Cloud Functions
	- Schedule Functions

### DAaaS Operation Model Design and Rollout

1. Se llevará a cabo un análisis de las fuentes de datos para determinar qué información extraeremos con un crawler. (Precio y estadísticas)
2. Extraeremos un fichero .csv por cada fuente de datos mediante un crawler programado para cada página web de referencia, que será guardado en Google Storage
3. Se comprobará que los datos son válidos y se procederá a automatizar este proceso cada 24 horas
4. Se procederá a preparar la ejecución del servidor web compuesto por nginx que ejecutará Vue para el front y Node-Express como back, comunicados mediante API REST
5. Se procederá a preparar la ejecución del deepback de python que ejecutará Flask, permitiendo establecer comunicación mediante API REST con node. Este deepback contendrá la implementación de Algoritmos de Machine Learning de Regresión.
6. Crearemos dos Cloud SQL en Google Cloud. Una será para uso privado (conexión con deepback de python) y otra será pública, conteniendo la información necesaria para ofrecer al usuario la funcionalidad indicada: predecir precios de diferentes discos de vinilo según el nombre del disco.
7. Dispondremos de BigQuery como almacén de datos:
	7.1 Se aplicarán Cloud Functions para la extracción de precios y estadísticas de los archivos .csv contenidos en el lago de datos
	7.2 Este proceso se repetirá cada vez que el crawler haga una nueva extración de datos (24h)
8. El resultado de la extracción de precios y estadísticas en bigQuery será guardado en un Cloud SQL privado al que sólo tendrá acceso el deepback de python para realizar labores de cálculo y predicción, cuyo resultado será copiado en el Cloud SQL "público" cuyo acceso será compartido con node con la finalidad de tener posibilidad de consultar "datos ya calculados", aliviando la carga de trabajo de Flask y con ello, reduciendo costes.

De esta manera, el usuario hace una petición al servidor web y éste puede consultar su Cloud SQL para comprobar si python ya ha procesado la predicción del precio del disco de vinilo. En caso negativo, node hará una llamada a Flask mediante Request Promise solicitando el cálculo de ese disco o una aproximación orientativa si no estuviese registrado ese título de disco, basada en los precios de otros discos del mismo artista. Podría devolver un *unknow* si no están disponible el título del disco y el artista o si el modelo de predicción no estuviese ajustado para un cálculo con unas mínimas garantías.

### IMAGEN DE LA ARQUITECTURA

![Arquitecura](url.jpeg)