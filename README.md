# Ember-g-map [![Build Status](https://travis-ci.org/asennikov/ember-g-map.svg?branch=master)](https://travis-ci.org/asennikov/ember-g-map) [![Ember Observer Score](http://emberobserver.com/badges/ember-g-map.svg)](http://emberobserver.com/addons/ember-g-map) [![Coverage Status](https://coveralls.io/repos/asennikov/ember-g-map/badge.svg?branch=master&service=github)](https://coveralls.io/github/asennikov/ember-g-map?branch=master) [![Code Climate](https://codeclimate.com/github/asennikov/ember-g-map/badges/gpa.svg)](https://codeclimate.com/github/asennikov/ember-g-map)

An ember-cli add-on for easy integration with Google Maps.
Each object displayed on map is inserted via child component,
so you can easily declare which marker and when to display on map
using `{{#if}}` and `{{#each}}` on template level.

# Installation

* `ember install ember-g-map`

# Configuration

You must define the size of the canvas in which the map is displayed.
Simply add something similar to this to your styles:

```css
.g-map-canvas {
  width: 600px;
  height: 400px;
}
```

In `config/environment.js` you can specify additional Google Maps libraries
to be loaded along with this add-on (check the full list [here](https://developers.google.com/maps/documentation/javascript/libraries)),
optional API key for your application (additional info could be found [here](https://developers.google.com/maps/web/)) and optional explicit protocol setting.

```javascript
ENV['g-map'] = {
  libraries: ['places', 'geometry'],
  key: 'your-unique-google-map-api-key',
  protocol: 'https'
}
```

# Usage

## Simple map

```handlebars
{{g-map lat=37.7833 lng=-122.4167 zoom=12}}
```

## Map with custom options

Any [custom options](https://developers.google.com/maps/documentation/javascript/3.exp/reference#MapOptions)
(except for `center` and `zoom` to avoid conflicts) could be set for
Google Map object on creation and updated on change.

```handlebars
{{g-map lat=37.7833 lng=-122.4167 zoom=12 options=customOptions}}
```

## Map with Markers

Mandatory `context` attribute ties child-elements
with the main `g-map` component. You can also set optional attributes:
- simple title appearing on hover using `title` attribute,
- marker label using `label`,
- `onClick` action to track all `click` events on that marker.

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}
  {{g-map-marker context lat=37.7933 lng=-122.4167 onClick=(action "handleClick")}}
  {{g-map-marker context lat=37.7833 lng=-122.4267 onClick="handleClick" title=titleForSecondMarker}}
  {{g-map-marker context lat=37.7733 lng=-122.4067 label="3" title="Marker #3"}}
{{/g-map}}
```

## Map with Info Windows

These Info Windows will be open right after component is rendered
and will be closed forever after user closes them. You can specify
optional `onClose` action to tear down anything you need when Info Window
has been closed by user.

Available options (see details [in docs](https://developers.google.com/maps/documentation/javascript/3.exp/reference#InfoWindowOptions)):
- disableAutoPan,
- maxWidth,
- pixelOffset

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}
  {{#g-map-infowindow context lat=37.7733 lng=-122.4067}}
    <h1>Info Window with Block</h1>
    <p>Text with {{bound}} variables</p>
    <button {{action "do"}}>Do</button>
  {{/g-map-infowindow}}
  {{g-map-infowindow context lat=37.7733 lng=-122.4067
                     title="Blockless form" message="Plain text."}}
  {{g-map-infowindow context lat=37.7733 lng=-122.4067
                     title="With action set"
                     onClose="myOnCloseAction"}}
  {{g-map-infowindow context lat=37.7733 lng=-122.4067
                     title="With closure action set"
                     onClose=(action "myOnCloseAction")}}
{{/g-map}}
```

## Map fits to show all initial Markers

`markersFitMode` attribute overrides `lat`, `lng`, `zoom` settings.
`markersFitMode` value can be one of:
* 'init' - which will make the map fit the markers on creation.
* 'live' - which will keep the map keep fitting the markers as they are added,
removed or moved.

```handlebars
{{#g-map markersFitMode='live' as |context|}}
  {{g-map-marker context lat=37.7933 lng=-122.4167}}
  {{g-map-marker context lat=37.7833 lng=-122.4267}}
  {{g-map-marker context lat=37.7733 lng=-122.4067}}
{{/g-map}}
```

## Map with Markers and bound Info Windows

Markers can have bound Info Windows activated on click.
To properly bind Info Window with Marker you should call `g-map-marker`
in block form and set context of Info Window to the one provided by Marker.

You can optionally setup custom `openOn`/`closeOn` events for each Info Window,
available options are: `click`, `dblclick`, `rightclick`, `mouseover`, `mouseout`.
By default `openOn` is set to `click` and `closeOn` is set to `null`. When `openOn`
and `closeOn` are the same, Info Window visibility is being toggled by this event.

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}
  {{#g-map-marker context lat=37.7833 lng=-122.4267 as |markerContext|}}
    {{#g-map-infowindow markerContext openOn="mouseover" closeOn="mouseout"}}
      <h2>Bound Info Window</h2>
    {{/g-map-infowindow}}
  {{/g-map-marker}}
  {{#g-map-marker context lat=37.7833 lng=-122.4267 as |markerContext|}}
    {{g-map-infowindow markerContext openOn="click" closeOn="click" title="Blockless form 1"}}
  {{/g-map-marker}}
  {{#g-map-marker context lat=37.7833 lng=-122.4267 as |markerContext|}}
    {{g-map-infowindow markerContext openOn="dblclick" title="Blockless form 2"}}
  {{/g-map-marker}}
{{/g-map}}
```

## Grouped Markers with Info Windows

Additionally you can specify parameter `group` which ensures that only
one Info Window is open at each moment for Markers of each group.

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}

  {{#g-map-marker context group="cats" lat=37.7833 lng=-122.4167 as |markerContext|}}
    {{#g-map-infowindow markerContext}}
      <h2>Cat #1</h2>
    {{/g-map-infowindow}}
  {{/g-map-marker}}
  {{#g-map-marker context group="cats" lat=37.7433 lng=-122.4467 as |markerContext|}}
    {{#g-map-infowindow markerContext}}
      <h2>Cat #2</h2>
    {{/g-map-infowindow}}
  {{/g-map-marker}}

  {{#g-map-marker context group="dogs" lat=37.7533 lng=-122.4167 as |markerContext|}}
    {{#g-map-infowindow markerContext}}
      <h2>Dog #1</h2>
    {{/g-map-infowindow}}
  {{/g-map-marker}}
  {{#g-map-marker context group="dogs" lat=37.7733 lng=-122.4467 as |markerContext|}}
    {{#g-map-infowindow markerContext}}
      <h2>Dog #2</h2>
    {{/g-map-infowindow}}
  {{/g-map-marker}}
{{/g-map}}
```

## Marker bound to address query

Proxy `g-map-address-marker` component takes address string as parameter
and translates it to lat/lng under the hood.

Optional `onLocationChange` action hook will send you back coordinates
of the latest address search result and the raw array of
[google.maps.places.PlaceResult](https://developers.google.com/maps/documentation/javascript/reference#PlaceResult) objects provided by `places` library.

Other optional parameters are the same as for `g-map-marker`.
Requires `places` library to be specified in `environment.js`.

```javascript
ENV['g-map'] = {
  libraries: ['places']
}
```

```javascript
actions: {
  onLocationChangeHandler(lat, lng, results) {
    Ember.Logger.log(`lat: ${lat}, lng: ${lng}`);
    Ember.Logger.debug(results);
  }
}
```

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}
  {{g-map-address-marker context address="San Francisco, Russian Hill"}}
  {{#g-map-address-marker context address="Delft, The Netherlands"}}
    {{#g-map-infowindow markerContext}}
      Works in block form too.
    {{/g-map-infowindow}}
  {{/g-map-address-marker}}

  {{g-map-address-marker context address=searchedAddress
                         onLocationChange=(action "onLocationChangeHandler")}}
  {{g-map-address-marker context address=anotherSearchedAddress
                         onLocationChange="onLocationChangeHandler"}}
{{/g-map}}
```

## Map with route between 2 locations

Using Google Maps [Directions](https://developers.google.com/maps/documentation/javascript/directions) service.

You can optionally set travel mode with `travelMode` attr:
- `walking`
- `bicycling`
- `transit`
- `driving` (default)

```handlebars
{{#g-map lat=37.7833 lng=-122.4167 zoom=12 as |context|}}
  {{g-map-route context
                originLat=37.7933 originLng=-122.4167
                destinationLat=37.7733 destinationLng=-122.4167}}
{{/g-map}}
```

## Demo

http://asennikov.github.io/ember-g-map/

# Planned Features

- Auto-closing Info Windows
- Polylines & Polygons
- Google Maps events

## Running

* `ember server`
* Visit your app at http://localhost:4200.

## Running Tests

* `ember test`
* `ember test --server`

## Building

* `ember build`

For more information on using ember-cli, visit [http://www.ember-cli.com/](http://www.ember-cli.com/).

## Legal

[Licensed under the MIT license](http://www.opensource.org/licenses/mit-license.php)
