# maps
Maps of Chile for interactive visualizations

This repository contains the raw shapefiles for maps of Chile's communes, provinces and regions as well as instructions to export the maps to the [GeoJSON](http://geojson.org/) format for their visualization.

## Requirements

[Topojson](https://github.com/mbostock/topojson) is used for exporting the maps to the GeoJson format. Topojson itself requires [Node.js](https://nodejs.org)  and [npm](https://www.npmjs.com/)

## Map data

The maps were compiled using data from the [chilean library of congress](https://www.bcn.cl/siit/mapas_vectoriales/index_html) open data website

### regions
The files in the *regiones* directory have the map data for Chile's 15 regions. Each region has the following properties:

| Name        | Description           |  
| ------------- |:-------------|
|  NOM_REG  | Name of the region |
| COD_REGI  | Identifier number for the region |

### provinces
The files in the *provincias* directory have the map data for Chile's 54 provinces. Each province has the following properties:

| Name        | Description           |  
| ------------- |:-------------|
|  NOM_PROV  | Name of the province |
| COD_PROV  | Identifier number for the province |
| COD_REGI  | Identifier number for the region the province belongs to |
| REGION  | Name of the region the province belongs to |

### communes
The files in the *comunas* directory have the map data for Chile's 346 communes. Each commune has the following properties:

| Name        | Description           |  
| ------------- |:-------------|
|  NOM_COM  | Name of the commune |
| CODIGO  | Identifier number for the commune |
| REGION  | Name of the region the commune belongs to |
| PROV  | Name of the province the commune belongs to |

## Exporting
In order to use the maps for visualizations they have to be exported to the GeoJSON format, in order to export the maps we'll use TopoJSON, an extension of GeoJSON that uses a more compact representation to reduce file size.

First to check if topojson is installed correctly type `topojson --help` into a terminal, this should display a message with the version and available options.
If for example we wanted to export the provinces map we should firt navigate to the *provincias* folder and enter the following command:

```
topojson -o output.json -p --shapefile-encoding utf8 provincias.shp
```
To break down what this command is doing, the `-o output.json` specifies the output file. The `-p` flag ensures the map will contain all the provinces properties such as names and identifiers, without it the resulting map will only contain the province's shapes. The `--shapefile-encoding utf8` option specifies that the utf-8 encoding should be used when writing the properties. Finally `provincias.shp` is the file containing the map's raw data.

The result of this command sohuld be the newly created output.json file which contains the map data. We can test it by uploading the file to [geojson.io](http://geojson.io) this should display all the provinces as polygons overlayed on a map of the world, you can click on any of the provinces to see it's properties.

### Reducing map size
TopoJSON can be used to reduce the map's file size by reducing the quality of the map, that is, the amount of points it contains. We do this by using the `--simplify-proportion` option, using the same example as before to reduce the size of the provinces map we use the following command:
```
topojson -o output.json -p --shapefile-encoding utf8 \
 --simplify-proportion 0.5 provincias.shp
```
This results in an output.json file which has roughly half the points of the original map. We can control how many points are eliminated by passing a number between 0 and 1 as an argument. Once again we can upload the resulting file to [geojson.io](http://geojson.io) to see the difference in quality.

### Filtering or renaming properties
The `-p` option can be used to omit certain properties from the resulting map or to rename them. By default TopoJSON removes all properties, using `-p` will preserve all properties. Passing an argument will preserve the specified properties, for example `-p FOO` will preserve the property with name "FOO". Multiple properties can be specified by separating them with commas, as in `-p FOO,BAR`.

The `-p` option also allows us to rename properties. By passing an argument in the form `-p target=source` we will preserve the "source" property and rename it to "target". Renaming properties also works with multiple properties. For example, to downcase the property names "FOO" and "BAR", use `-p foo=FOO,bar=BAR`.

### Adding external properties

TopoJSON also allows us to import properties from a TSV or CVS file and add them to our map with the `-e` option.
We can use the [example_data.csv](./example_data.csv) file included in this repository to test this functionality by adding a new property to our regions map. Navigate to the *regiones* directory and enter the following command:

```
topojson -o output.json -e ../example_data.csv \
--id-property=COD_REGI,+ID \
-p unemployment=+VALUE,name=NOM_REG \
--shapefile-encoding utf8 \
--simplify-proportion 0.5 regiones.shp
```

The `-e ../example_data.csv` part specifies the CSV file we will be taking the data from. The [example_data.csv](./example_data.csv) file contains two columns ID and VALUE, the first corresponds to the identifier of each region and the second to the value we want to add to the map.

The `--id-property=COD_REGI,+ID` part of the command tells TopoJSON the properties "COD_REGI" from the original map and "ID" from the CSV file function as identifiers for the objects in the map, the program will use these identifiers to join the properties on the map with those in the CSV file. "ID" is preceded by the plus sign (+) in the command to specify the values int the "ID" column should be treated as numbers otherwise they would be treated as strings and wouldn't match the values in "COD_REGI".  
Finally the `-p new_prop=+VALUE,NOM_REG` tells the program the output map should have the properties "new_prop" which takes it's values from the "VALUE" column in the CSV file and "NOM_REG" which is taken from the original map. Once again we prepend "VALUE" with a plus sign to trear these values as numbers.
