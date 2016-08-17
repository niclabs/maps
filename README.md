# Maps
Mapas de Chile para visualizaciones interactivas.

El repositorio contiene archivos .shp con mapas de las comunas, provincias y regiones de Chile, además de instrucciones para transformar estos archivos a formato [GeoJSON](http://geojson.org/) para su visualización.

## Dependencias

Se usa el programa [Topojson](https://github.com/mbostock/topojson) para exportar los mapas a formato GeoJson. Topojson además tiene como dependencias [Node.js](https://nodejs.org) y [npm](https://www.npmjs.com/).

## Datos de los mapas
Los mapas se crearon en base a los archivos disponibles en el portal de datos abiertos de la [Biblioteca Nacional del Congreso](https://www.bcn.cl/siit/mapas_vectoriales/index_html).

### Regiones
Los archivos de la carpeta *regiones* contienen los datos del mapa de las 15 regiones de Chile. Cada región tiene las siguientes propiedades:

| Nombre        | Descripción  |  
| ------------- |:-------------|
|  NOM_REG  | Nombre de la region |
| COD_REGI  | Número identificador de la región |

### Provincias
Los archivos de la carpeta *provincias* contienen los datos del mapa de las 54 provincias de Chile. Cada provincia tiene las siguientes propiedades:

| Nombre        | Descripción  |  
| ------------- |:-------------|
|  NOM_PROV  | Nombre de la provincia |
| COD_PROV  | Número identificador de la provincia |
| COD_REGI  | Número identificador de la región a la cual pertenece la provincia |
| REGION  | Nombre de la región a la cual pertenece la provincia |

### Comunas
Los archivos de la carpeta *comunas* contienen los datos del mapa de las 346 comunas de Chile. Cada comuna tiene las siguientes propiedades:

| Nombre        | Descripción  |  
| ------------- |:-------------|
|  NOM_COM  | Nombre de la comuna |
| CODIGO  | Número identificador de la comuna |
| REGION  | Nombre de la región a la cual pertenece la comuna |
| PROV  | Nombre de la provincia a la cual pertenece la comuna |

## Exportar
Para usar los mapas en visualizaciones interactivas es necesario exportarlos al formato GeoJson. Para exportar los mapas usaremos TopoJson, una extensión del formato GeoJSON que usa una representación más compacta para reducir el tamaño de archivo del mapa final.

Antes de empezar hay que revisar si topojson está instalado correctamente, ejecutar el comando `topojson --help` en la consola debería mostrar un mensaje con el número de versión y las opciones disponibles.
Para exportar, por ejemplo, el mapa de provincias hay que navegar a la carpeta *provincias* y ejecutar el siguiente comando:
```
topojson -o output.json -p --shapefile-encoding utf8 provincias.shp
```
En el comando anterior la opción `-o output.json` especifica el archivo de salida. La opción `-p` hace que se el archivo de salida contenga las propiedades del mapa original como los nombres e identificadores de cada provincia, sin está opción el mapa solo contendrá los poligonos correspondientes a cada provincia. La opción `--shapefile-encoding utf8` especifica que se debe usar el encoding utf-8 para escribir las propiedades. Finalmente `provincias.shp` es el archivo que contiene los datos del mapa original.

Cuando termine la ejecución del comando se creará el archivo output.json, podemos comprobar que este archivo se creó correctamente subiendolo a [geojson.io](http://geojson.io). Se deberían desplegar sobre el mapa los poligonos de las provincias de Chile, al hacer click en cualquier provincia se pueden ver sus propiedades.

### Reducir el tamaño del archivo
TopoJSON puede reducir el tamaño del archivo al reducir la calidad del mapa, esto es, la cantidad de puntos que contiene. Para esto usamos la opción `--simplify-proportion`, usando el mismo ejemplo anterior para reducir el tamaño del mapa de provincias usamos el siguiente comando:
```
topojson -o output.json -p --shapefile-encoding utf8 \
 --simplify-proportion 0.5 provincias.shp
```
El nuevo archivo archivo output.json tiene aproximadamente la mitad de puntos que el original. Se puede controlar la cantidad de puntos que se conservan en el mapa pasando como argumento un número entre 0 y 1. Nuevamente podemos probar el archivo subiéndolo a [geojson.io](http://geojson.io) para apreciar la diferencia en la calidad del mapa.

### Filtrar o renombrar propiedades
El flag `-p` se puede usar para seleccionar que propiedades se mantienen en el mapa o para renombrarlas. Por defecto TopoJSON elimina todas las propiedades del mapa original, al usar la opción `-p` sin argumentos se preservan todas las propiedades. Al pasar argumentos se puede especificar que propiedades preservar, por ejemplo `-p FOO` preservará la propiedad "FOO". Se pueden pasar múltiples propiedades separándolas con comas, por ejemplo `-p FOO,BAR`.

La opción `-p` también permite renombrar propiedades. AL pasar un argumento con el formato `-p nuevo=original` preservará la propiedad "original" y la renombrará a "nuevo" en el archivo de salida. Renombrar propiedades también funciona entregando múltiples propiedades como argumento. Por ejemplo la opción `-p foo=FOO,bar=BAR` renombrará con minúsculas las propiedades "FOO" y "BAR".

### Añadir propiedades desde archivos externos

TopoJSON permite añadir propiedades a un mapa tomando datos desde un archivo TSV o CSV usando la opción `-e`.
Usemos el archivo [example_data.csv](./example_data.csv) incluido en este repositorio para probar esta funcionalidad añadiendo una nueva propiedad al mapa de regiones. Desde el directorio *regiones* ingresar el siguiente comando:
```
topojson -o output.json -e ../example_data.csv \
--id-property=COD_REGI,+ID \
-p unemployment=+VALUE,name=NOM_REG \
--shapefile-encoding utf8 \
--simplify-proportion 0.5 regiones.shp
```
La opción `-e ../example_data.csv` especifica el archivo CSV con los datos. El archivo [example_data.csv](./example_data.csv) tiene dos columnas de datos, "ID" y "VALUE", la primera corresponde al número identificados de cada región y la segunda el valor de la propiedad para añadir al mapa.
La opción `--id-property=COD_REGI,+ID` le dice a topojson que la propiedad "COD_REGI" del mapa original y "ID" del archivo CSV son identificadores para los objetos del mapa, el programa usa estos identificadores para emparejar los valores del CSV con los del mapa. "ID" es precedido por un signo más (+) para especificar que los valores de la columna "ID" son números, de lo contrario serían interpretados como strings y no se podrían emparejar con los de "COD_REGI".
Finalmente la opción `-p nueva_prop=+VALUE,NOM_REG` especifica que el archivo de salida tendrá las propiedades "nueva_prop" que viene del valor de "VALUE" del archivo CSV y "NOM_REG" que viene del mapa original. Al igual que antes el signo más significa que la propiedad correspondiente será tratada como número.
