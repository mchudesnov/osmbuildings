# OSM Buildings

A JavaScript library for visualizing 3D OSM building data on interactive maps

## Important

The whole library consists of client JavaScript files, a server side PHP script and a MySQL or PostGis database.
Everything should be seen as alpha state, all components are likely to change extensively.

Bottleneck at the moment is data availability. I can't process and host a whole OSM planet file.
Actually I'm looking for a partner to provide a data service to you.


## Prerequisites

1. You will need MySQL as data storage. Version 5.0.16 or better has the required <a href="http://dev.mysql.com/doc/refman/5.0/en/spatial-extensions.html">Spatial Extensions</a> enabled.
Everything can be done in different Geo enabled databases too. For now we just go MySQL.
For those who have trouble importing the data into MySQL or running a different server, <a href="https://twitter.com/D_Guidi">Diego Guidi</a> did a great job creating a Shapefile.

* See Data conversion below *

2. You will need to create your database table using the dump file /server/data/mysql-CREATE_TABLE.sql .
Then import building data, i.e. from /server/data/mysql-berlin.zip . Either upload this directly in PhpMyAdmin or unpack and import as you like.

3. Make sure PHP is running and create a /server/config.php file for your setting s and database credentials.
Or just adapt and rename /server/config.sample.php for your needs.


## Integration

I assume, Leaflet is already integrated in your html page. If not, head over to its <a href="http://leaflet.cloudmade.com/reference.html">documentation</a>.
Important: version 0.4 of Leaflet.js is required.

Then in header section, add:

```html
<head>
    :
    :
  <script src="dist/buildings.js"></script>
</head>
```

after Leaflet initialization add:

```javascript
var map = new L.Map('map');
  :
  :
// You may stay with any maptiles you are already using. these are just my favourites.
// Remember to obtain an API key from MapBox.
// And keep the attribution part for proper copyright notice.
var mapboxTiles = new L.TileLayer(
	'http://{s}.tiles.mapbox.com/v3/mapbox.mapbox-streets/{z}/{x}/{y}.png',
	{
		attribution: 'Map tiles &copy; <a href="http://mapbox.com">MapBox</a>',
		maxZoom: 17
	}
);

// there is just data for Berlin, Germany at the moment, lets start there
map.setView(new L.LatLng(52.52111, 13.40988), 17).addLayer(mapboxTiles);


// This finally creates the buildings layer.
// As soon as you attach it to the map it starts loading data from your PHP/MySQL combo.
// You will need to have this on the same server, otherwise there are cross origin issues.
var buildings = new OSMBuildings(
	'server/?w={w}&n={n}&e={e}&s={s}&z={z}',
	{
		strokeRoofs: false,
		wallColor: 'rgb(190,170,150)',
		roofColor: 'rgb(230,220,210)',
		strokeColor: 'rgb(145,140,135)'
	}
);
buildings.addTo(map);

// Like to save code? This one liner basically does the same thing as above.
new OSMBuildings('server/?w={w}&n={n}&e={e}&s={s}&z={z}').addTo(map);
```

done.

## Data Conversion

As PostGIS seems to be much more popular for handling Geometry and MySQL being much mor popular for commercial Webhosting, there is now a data conversion script in using Node.js.
Requirements are: a working Node.js installation (I tested successfully 0.6 on Windows) and the node-postgres module. Install the module with <code>npm install pg</code>.
Then have a look into /server/data/convert.js and change database, table, output settings.
Run the conversion with <code>node convert.js</code>.

What it does:

- reads height and footprint polygons from PostGIS
- turns height into a number if needed
- swaps lat/lon of polygons if needed
- creates a mysql dump file

For any further information visit <a href="http://osmbuildings.org">http://osmbuildings.org</a>, follow <a href="https://twitter.com/osmbuildings">@osmbuildings</a> on Twitter or report issues here on Github.