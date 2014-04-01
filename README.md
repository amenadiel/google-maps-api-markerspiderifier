Overlapping Marker Spiderfier,  Google Maps API v3 Library
==========================================================

h1. This is a fork from @jawj  [Overlapping Marker Spiderfier for Google Maps API v3](https://github.com/jawj/OverlappingMarkerSpiderfier)

** This fork preserves all functionalities. It's mostly original code converted the code from Coffee (for those of you that, like me, like to experiment with code and don't like to mess with coffee) with verbosity and namespace added to each object so it became clear what each part was doing.
** The code wasd onverted to reflect jshint best practices
** OverlappingMarkerSpiderifier is now loaded as part of the google.maps namespace, so instead of

```js
var Spider= new OverlappingMarkerSpiderifier();
```

it is called with

```js
var Spider= new google.maps.OverlappingMarkerSpiderifier();
```

This way, if you're loading Google Maps with AMD such as [RequireJS](http://requirejs.org/) you don't need to white shims to get a return object. Just make sure you load it after Google Maps.

As per default funcionalities, the following is a brief C&P from @jawj  [Overlapping Marker Spiderfier for Google Maps API v3](https://github.com/jawj/OverlappingMarkerSpiderfier)


Ever noticed how, in [Google Earth](http://earth.google.com), marker pins that overlap each other spring apart gracefully when you click them, so you can pick the one you meant? This code makes Google Maps API **version 3** map markers behave in that way (minus the animation). This extension can accomodate numberless markers in an expanding spiral so they don't overlap.

The compiled code has no dependencies beyond Google Maps. And it’s under 3K when compiled out of
[CoffeeScript](http://jashkenas.github.com/coffee-script/), minified with Google’s [Closure Compiler](http://code.google.com/closure/compiler/) and gzipped.

I wrote it as part of the data download feature for [Mappiness](http://www.mappiness.org.uk/maps/).

**There’s now also [a port for the Leaflet maps API](https://github.com/jawj/OverlappingMarkerSpiderfier-Leaflet)


Demo
----

See the [demo map](http://jawj.github.com/OverlappingMarkerSpiderfier/demo.html) (the data is random: reload the map to reposition the markers).

Download
--------

Download [the compiled, minified JS source](http://jawj.github.com/OverlappingMarkerSpiderfier/bin/oms.min.js).

How to use
----------

See the [demo map source](https://github.com/jawj/OverlappingMarkerSpiderfier/blob/gh-pages/demo.html),
or follow along here for a slightly simpler usage with commentary.

Create your map like normal:

```sh
     var map = new google.maps.Map(document.getElementById('map_canvas'), {
      mapTypeId: google.maps.MapTypeId.SATELLITE,
      center: new google.maps.LatLng(50, 0), 
      zoom: 6
    });

    var oms = new google.maps.OverlappingMarkerSpiderfier(map);
```


Instead of adding click listeners to your markers directly via `google.maps.event.addListener`, add a global listener on the `OverlappingMarkerSpiderfier` instance instead. The listener will be passed the clicked marker as its first argument, and the Google Maps `event` object as its second.


```js
    var iw = new google.maps.InfoWindow();
    oms.addListener('click', function(marker, event) {
      iw.setContent(marker.desc);
      iw.open(map, marker);
    });
```


You can also add listeners on the `spiderfy` and `unspiderfy` events, which will be passed an array of the markers affected. In this example, we observe only the `spiderfy` event, using it to close any open `InfoWindow`:

```sh
    oms.addListener('spiderfy', function(markers) {
      iw.close();
    });
```


Finally, tell the `OverlappingMarkerSpiderfier` instance about each marker as you add it, using the `addMarker` method:

```sh
    for (var i = 0; i < window.mapData.length; i ++) {
      var datum = window.mapData[i];
      var loc = new google.maps.LatLng(datum.lat, datum.lon);
      var marker = new google.maps.Marker({
        position: loc,
        title: datum.h,
        map: map
      });
      marker.desc = datum.d;
      oms.addMarker(marker);  // <-- here
    }
```


Docs
----

### Construction && Options


```sh
    new google.maps.OverlappingMarkerSpiderfier(map, options)
```

Creates an instance associated with `map` (a `google.maps.Map`).

The `options` argument is an optional `Object` specifying any options
you want changed from their defaults. The available options are:

**markersWontMove** and **markersWontHide** (defaults: `false`)

By default, change events for each added marker’s `position` and `visibility` are observed (so that, if a spiderfied marker is moved or hidden, all spiderfied markers are unspiderfied, and the new position is respected where applicable).

However, if you know that you won’t be moving and/or hiding any of the markers you add to this instance, you can save memory (a closure per marker in each case) by setting the options named `markersWontMove` and/or `markersWontHide` to `true` (or anything [truthy](http://isolani.co.uk/blog/javascript/TruthyFalsyAndTypeCasting)).

For example,

`var oms = new google.mapsOverlappingMarkerSpiderfier(map, {markersWontMove: true, markersWontHide: true});`.

**keepSpiderfied** (default: `false`)

By default, the OverlappingMarkerSpiderfier works like Google Earth, in
that when you click a spiderfied marker, the markers unspiderfy before
any other action takes place.

Since this can make it tricky for the user to work through a set of
markers one by one, you can override this behaviour by setting the
`keepSpiderfied` option to `true`.

**nearbyDistance** (default: `20`).

This is the pixel radius within which a marker is considered to be
overlapping a clicked marker.

**circleSpiralSwitchover** (default: `9`)

This is the lowest number of markers that will be fanned out into a
spiral instead of a circle. Set this to `0` to always get spirals, or
`Infinity` for all circles.

**legWeight** (default: `1.5`)

This determines the thickness of the lines joining spiderfied markers to
their original locations.

### Instance methods: managing markers

Note: methods that have no obvious return value return the
OverlappingMarkerSpiderfier instance they were called on, in case you
want to chain method calls.

**addMarker\*
\
Adds `marker` to be tracked.
\
**removeMarker(marker)\*

Removes `marker` from those being tracked. This *does not* remove the
marker from the map (to remove a marker from the map you must call
`setMap(null)` on it, as per usual).

**clearMarkers\*
\
Removes every `marker` from being tracked. Much quicker than calling
`removeMarker` in a loop, since that has to search the markers array
every time.
\
This *does not* remove the markers from the map @ on each of them, as
per usual).
\
**getMarkers()\*

Returns an array of all the markers that are currently being tracked.
This is a copy of the one used internally, so you can do what you like
with it.

### Instance methods: managing listeners

**addListener\*
\
Adds a listener to react to one of three events.
\
`event` may be `'click'`, `'spiderfy'` or `'unspiderfy'`.
\
For `'click'` events, `listenerFunc` receives one argument: the clicked
marker object. You’ll probably want to use this listener to do something
like show a `google.maps.InfoWindow`.
\
For `'spiderfy'` or `'unspiderfy'` events, `listenerFunc` receives two
arguments: first, an array of the markers that were spiderfied or
unspiderfied; second, an array of the markers that were not. One use for
these listeners is to make some distinction between spiderfied and
non-spiderfied markers when some markers are spiderfied — e.g.
highlighting those that are spiderfied, or dimming out those that
aren’t.
\
**removeListener(event, listenerFunc)\*

Removes the specified listener on the specified event.

**clearListeners\*
\
Removes all listeners on the specified event.
\
**unspiderfy()\*

Returns any spiderfied markers to their original positions, and triggers
any listeners you may have set for this event. Unless no markers are
spiderfied, in which case it does nothing.

### Instance methods: advanced use only!

**markersNearMarker\*
\
Returns an array of markers within `nearbyDistance` pixels of `marker` —
i.e. those that will be spiderfied when `marker` is clicked. If you pass
`true` as the second argument, the search will stop when a single marker
has been found. This is more efficient if all you want to know is
whether there are any nearby markers.
\
*Don’t* call this method in a loop over all your markers, since this can
take a *very* long time.
\
The return value of this method may change any time the zoom level
changes, and when any marker is added, moved, hidden or removed. Hence
you’ll very likely want call it every time the map’s `zoom_changed`
event fires *and* any time you add, move, hide or remove a marker.
\
Note also that this method relies on the map’s `Projection` object being
available, and thus cannot be called until the map’s first `idle` event
fires.
\
**markersNearAnyOtherMarker()\*

Returns an array of all markers that are near one or more other markers
— i.e. those will be spiderfied when clicked.

This method is several orders of magnitude faster than looping over all
markers calling `markersNearMarker` (primarily because it only does the
expensive business of converting lat/lons to pixel coordinates once per
marker).

The return value of this method may change any time the zoom level
changes, and when any marker is added, moved, hidden or removed. Hence
you’ll very likely want call it (and take appropriate action) every time
the map’s `zoom_changed` event fires *and* any time you add, move, hide
or remove a marker.

Note also that this method relies on the map’s `Projection` object being
available, and thus cannot be called until the map’s first `idle` event
fires.

### Properties

You can set the following properties on an OverlappingMarkerSpiderfier
instance:

**legColors.usual[mapType]** and **legColors.highlighted[mapType]**

These determine the usual and highlighted colours of the lines, where
`mapType` is one of the `google.maps.MapTypeId` constants ([or a custom
map type
ID](https://github.com/jawj/OverlappingMarkerSpiderfier/issues/4)).

The defaults are as follows:

    var mti = google.maps.MapTypeId;
    legColors.usual[mti.HYBRID] = legColors.usual[mti.SATELLITE] = '#fff';
    legColors.usual[mti.TERRAIN] = legColors.usual[mti.ROADMAP] = '#444';
    legColors.highlighted[mti.HYBRID] = legColors.highlighted[mti.SATELLITE] = 
      legColors.highlighted[mti.TERRAIN] = legColors.highlighted[mti.ROADMAP] = '#f00';

You can also get and set any of the options noted in the constructor
function documentation above as properties on an
OverlappingMarkerSpiderfier instance. However, for some of these options
(e.g. `markersWontMove`) modifications won’t be applied retroactively.

Licence
-------

This software is released under the [MIT
licence](http://www.opensource.org/licenses/mit-license.php).

Finally, if you want to say thanks, I am on
[Gittip](https://www.gittip.com/jawj).
