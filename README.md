GeoMarX is designed to empower geospatial marking and annotation of complex targets in urban environments. GeoMarX addresses the need for precise marking and annotation by providing a user-friendly platform equipped with integrated mapping functionalities and annotation tools as well as leveraging cutting-edge machine learning technologies, which enables users to accurately mark specific points/areas of interest and annotate them with rich contextual information to further facilitate efficient data management and decision-making processes in urban planning, transportation management, environmental monitoring,and other urban-related domains.

# Requirements

GeoMarX adpots the B/S structure.

For the user side, a browser is all that needed and there is no need for additional software installation or configuration.

For the server side, we need to install some softwares and Python packages as dependencies, which are cross-platform. Here we take Windows as an example:

1. Apache 2.4.61 (Win64)
2. GeoServer 2.19.1
3. PostgreSQL & PostGIS:
    * PostgreSQL 16.1
    * PostGIS="3.4.1

The required Python packages are as follows:

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

# Installation and configuration

## Download front-end files

First, download the `dist` we provided and unzip to a path. This file houses the front-end pages of GeoMarX, and you need to use Apache to forward the pages to access our software. 

You will find a `confi.js` file in the `dist` and open the `confi.js`. Because we need to receive the data sent by the backend through the IP address, we need to change the LOCAL_IP in the `config.js` to the IP address of the current device.

## Apache configuration

We use Apache to forward front-end pages to enable multi-user collaboration. As a result, Apache is an integral part of GeoMarX. 

First of all, we download the Apache installation package. It is worth noting that you need to change the storage path of Apache in the `~\Apache24\conf` before installing the Apache service.

And then, put the following codes into `~\Apache24\conf`.

```HTML
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

Before running Apache, please make sure to replace the path to `dist`, the path to the street view imagery and the server's IP address.

## GeoServer configuration

In GeoMarX, GeoServer is used to convert auxiliary data, such as road networks, points-of-interest, and administrative boundaries, into layers and provide them to the front-end. These multi-source data are important references in the labeling process. If you just want to use GeoMarX's annotation features without having to add multiple data as references, then you can skip the GeoServer installation。

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

The variable needs to be modified according to the actural demand, and the value here is only used as a reference. However, it is important to note that the content in `LAYERS` and `STYLES` are separated only by `,`, and there should be no extra spaces.

# Functionality and features

## Enhancing focus through geospatial localization

It is imperative to quickly localize and maintain concentration on the area of interest for geospatial marking and annotation, particularly in complex environments like urban areas, where numerous intricate components are clustered together.

Our tool addresses this challenge through a dual-view approach: reference points-of-interest (POIs) or areas-of-interest (AOIs) data and geographic zoning systems.

The former will be displayed in the POI/AOI Search Panel. For the labeling of urban villages, we collected publicly available urban village data and associated them with spatial locations to form POI data. You can use the drop-down lists to filter data from the province, city, and district levels. Click on the row quickly locating the location of the urban village.

For Geographic Zoning Systems, it can be either Geographic Grid Systems or Administrative Boundaries. In our case, we use Administrative Boundaries as the second system, select the province and city option in the drop-down list, and the basemap will automatically zoom to the corresponding region.

## Informed decision-making with integrated data

In the past, the identification of complex geographical targets depended on on-site surveys or solely on remote sensing imagery.However, making accurate decisions based solely on overhead views from satellite imagery is challenging.

To support informed decision-making, our tool serves as a platform that integrates multi-source heterogeneous geospatial data in various formats. This includes remote sensing imagery, street view images, POIs, road networks, administrative boundaries, and geographic grid systems. 

For labeling urban villages, we also integrate this data, with POI to help quickly identify urban villages, and road networks to facilitate precise boundary demarcation. Street view images are also an important help to discover urban villages and determine the location of boundaries.

## Enhancing geospatial marking with AI support

We utilize the cutting-edge Segment Anything Model (SAM) to improve the efficiency of geospatial marking when working with remote sensing images. You need to draw a plotted area to include the labeling range first. And then mark the cue point with the point tool, and GeoMarX will quickly generate the segmentation results based on SAM. For a large target area, we can use the rectangle tool to draw a box as a hint input, which often results in a better segmentation than the point.

## Efficient management of annotation data

It's critical to store and manage geospatial marking and annotation data. Our tool is designed to facilitate this by enabling the saving and management of data within PostgreSQL and PostGIS. Each marking and annotation action automatically saves a record of the geometry and associated attributes. This data is readily accessible through a data management panel, which displays both the geometrical shapes and their associated attributes, and can be quickly navigated to the corresponding locations via single clicking action. In turn, we can also quickly navigate to corresponding records via clicking on the geospatial objects in the map panel. These features allow users to efficiently edit and manage records, streamlining the annotation process.

## Collaborative and simultaneous geospatial annotation

Geospatial marking and annotation require a different approach to workload division compared to natural image annotation, primarily due to the involvement of large geographic regions represented by extensive remote sensing imagery. 

Our tools facilitate this large-scale effort through two key features. First, we provide support for auxiliary geographic coordinate systems, such as uniform grid systems or administrative boundaries. This allows for a structured division of labor, with marked units highlighted to inform annotators of progress and help them identify starting points quickly. Second, our tool's distributed B/S architecture enables multiple users to work concurrently. With the back-end hosted on a server and accessible through a browser interface, users can collaborate in real-time via network connections, supporting distributed teamwork effectively.

# Demo video
For details about how to use GeoMarX, please refer to the demo video [here](https://www.youtube.com/watch?v=LipSBH0jFGw).
