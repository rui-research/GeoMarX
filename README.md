# GeoMarX

GeoMarX is designed to empower geospatial marking and annotation of complex targets in urban environments. GeoMarX addresses the need for precise marking and annotation by providing a user-friendly platform equipped with integrated mapping functionalities and annotation tools as well as leveraging cutting-edge machine learning technologies, which enables users to accurately mark specific points/areas of interest and annotate them with rich contextual information to further facilitate efficient data management and decision-making processes in urban planning, transportation management, environmental monitoring,and other urban-related domains.

## Requirements

For the user side, it is cross-platform software, and there is no need to do additional software installation or configuration.

For the server side, we need to install some softwares and python packages as dependencies. Here we take Windows as an example:

1. Apache 2.4.61 (Win64)
2. GeoServer 2.19.1
3. PostgreSQL & PostGIS:
    * PostgreSQL 16.1
    * PostGIS="3.4.1

The required python packages are as follows:

```python
Django==3.2.16
djangorestframework==3.13.1
geopandas==0.14.3
numpy==2.1.3
opencv_python==4.9.0.80
pandas==1.3.5
Pillow==11.0.0
psycopg2==2.9.9
psycopg2_binary==2.9.6
rasterio==1.3.9
Requests==2.32.3
segment_anything==1.0
segment_anything_py==1.0
segment_geospatial==0.10.2
Shapely==1.8.5.post1
SQLAlchemy==2.0.29
torch==2.2.1
```

## Installation

### Download Front-end files

First, download the `dist` we provided and unzip to a path. This file houses the front-end pages of GeoMarX, and you need to use apache to forward the pages to access our software. 

You will find a `confi.js` file in the `dist` and open the `confi.js`. Because we need to receive the data sent by the backend through the IP address, we need to change the LOCAL_IP in the `config.js` to the IP address of the current device.

### Apache configuration

We use Apache to forward front-end pages to enable multi-user collaboration. As a result, Apache is an integral part of GeoMarX. 

First of all, we download the Apache installation package, you need to download different versions according to your computer. It is worth noting that you need to change the storage path of Apache in the `~\Apache24\conf` before installing the Apache service.

And then, put the following codes into `~\Apache24\conf`.

```python
Listen 81

<VirtualHost *:81>
    DocumentRoot "Path/to/dist"
    ProxyPass /samserver/ http://Server_IP_Address:8001/
    ProxyPassReverse /samserver/ Server_IP_Address:8001/
    <Directory "Path/for/dist">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

Alias /media/ "Path/to/Street/View/imagery"
<Directory "Path/to/Street/View/imagery">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

But before running Apache, please make sure to replace the path to the `dist`, the path to the street view imagery and the server's IP address.

### GeoServer configuration

In GeoMarX, GeoServer is used to convert the road network, POI, and administrative boundaries into layers and provide them to the front-end, and these multiple data are important references in the labeling process. If you just want to use GeoMarX's annotation features without having to add multiple data as references, then you can skip the GeoServer installation。

Of course, you'll need to install the GeoServer yourself. First, create a workspace in the geoserver and publish all the layers in this workspace. And then modify `LAYER_NUM`, `OPACITY`, `LAYERS` and `STYLES` in `~/dist/config.js` as follow:

```python

LAYER_NUM: '2',
OPACITY: '0.5',
LAYERS: 'guangzhou_osm_road_networks,guangzhou_uv_pois',
STYLES: 'line_red,point_green',

```

Let's explain what these variables mean: 

* `LAYER_NUM`: the number of layers you need to publish
* `OPACITY`: the transparency of the layer
* `LAYERS`: the names of layers
* `STYLES`: the style of the layer

However, it is important to note that the content in `LAYERS` and `STYLES` are separated only by `,`, and there should be no extra spaces.

## Manual

### Base Map and Auxiliary Data

1. Upload data to the server's GeoServer and publish as layers.
2. Modify parameters in `~/dist/config.js`.
3. The software automatically loads base maps and auxiliary data.

### Filter Target Prompts

1. Use the table in the upper right corner of the software interface to display information about urban village prompts.
2. Use drop-down lists to filter by province, city, county administrative units, and annotation status.
3. The software automatically saves filter results and page numbers for users to return to the last working page.

### Prompt Location Jump

1. Click on an item in the data table.
2. The software will automatically zoom to the location of POI and display it with a blue prompt point.


### Annotation Result Location Jump

1. Click on an item in the result display table
2. The base map automatically jumps and zooms to the location of the annotation result.
3. Click on a annotation on the base map, and the data management table automatically jumps to the page of the corresponding record.


### Display Street View Images

1. Click the 'Enable' button in the upper left corner to activate the street view display function.
2. Click on a blue prompt point on the base map to load street view images from four different angles at that location.


### Create Mapping Area

1. Click the 'Create Mapping Area' button.
2. Draw a rectangle that includes the area to be annotated.
3. Wait for the software to generate the required tile map
4. Click the prompt to confirm the completion of the mapping area creation.


### Manual Annotation

1. Click the polygon tool button.
2. Draw the boundaries of the urban village point by point on the base map.
3. Fill in the annotation result's remarks: name, remarks and urban village or not.


### Machine Learning Enhanced Assisted Annotation
Use point prompts or rectangle prompts for assisted annotation and fill in the annotation result information.


#### Point Prompt

1. Click the point tool button.
2. Point the marker at the location of the target to be labeled.
3. Wait for the results to be generated.
4. Fill in the annotation result's remarks: name, remarks and urban village or not.

#### Rectangle Prompt

1. Click the rectangle tool button.
2. Draw a rectangular area that encompasses the target to be labeled.
3. Wait for the results to be generated.
4. Fill in the annotation result's remarks: name, remarks and urban village or not.

### Data Storage and Management

#### Data Storage

_The software automatically stores annotation results into the database._

#### Edit Annotation Boundaries

1. Click the select button in front of the record in the result management table.
2. Click on the 'Edit' button located on the left.
3. Drag, add, or delete vertices.
4. Click 'Save' button to save the modified results.

#### Delete Annotations

1. Find out the record to delete in the result management table.
2. Click the 'delete' button at the end of the record.
3. Click 'yes' to confirm the deletion.


## Demo video

You can watch the demo video [here](https://www.youtube.com/watch?v=bkIffmBS43Y)
