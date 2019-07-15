
# Prerequisites

* Install docker and docker-compose

Retrieve the images or build them

    docker-compose up

Also enable to run the database first



./configure.sh osm-bright/osm-bright.imposm.mml > osm-bright/project.mml

Get shapefiles

    cd osm-bright/shp/
    wget https://osmdata.openstreetmap.de/download/simplified-water-polygons-split-3857.zip
    wget https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip
    unp *.zip
    mkdir -p ne_10m_admin_0_boundary_lines_land && cd ne_10m_admin_0_boundary_lines_land
    wget http://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_0_boundary_lines_land.zip
    unp *.zip
    cd ../../..

Get OSM dataset to import

    wget https://download.geofabrik.de/europe/france/aquitaine-latest.osm.pbf
    docker-compose run imposm3 import \
        -mapping mapping.json \
        -read aquitaine-latest.osm.pbf \
        -overwritecache -write \
        -connection 'postgis://osm:osm@localhost:5432/osm'

Then, publish in production (move tables from import schema to public schema)

    docker-compose run imposm3 import \
    -mapping mapping.json \
    -connection 'postgis://osm:osm@localhost:5432/osm' \
    -deployproduction


Serve project using Kosmtik

    docker-compose run kosmtik kosmtik serve osm-bright/project.mml --host 0.0.0.0

Generate tiles

    docker-compose run kosmtik kosmtik export osm-bright/project.mml --format mbtiles --output output.mbtiles --minZoom 10 --maxZoom 16


Rebuild an image

    docker-compose build --no-cache kosmtik

Check running services

    docker ps
