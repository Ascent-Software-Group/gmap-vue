## About GmapVue

GmapVue is a fork from [vue2-google-maps](https://github.com/xkjyeah/vue-google-maps).

This project has all features added to `vue2-google-maps` plugin up to [v0.10.8](https://github.com/xkjyeah/vue-google-maps/releases/tag/v0.10.8), but in the case of `gmap-vue` it has **the last features** added to `vue2-google-maps` repository by the community developers in many PRs, that they can't landed in a new version of that project, for different reasons.

Because of that we fork the project and plain to continue working and adding new features to this great plugin.

## Braking changes

### v3.0.0

- `autobindAllEvents` config option was renamed to `autoBindAllEvents`
- `vueGoogleMapsInit` name was renamed to `GoogleMapsCallback`
- `gmapApi` function was renamed to `getGoogleMapsAPI`
- `MapElementMixin` now is exported from `components` object instead of `helpers` object
- `customCallback` config option was added to reuse existing Google Maps API that already loaded, eg from an HTML file

### v2.0.0

- All components were rewriting in SFC (`.vue`)
- `MarkerCluster` was renamed to `Cluster`
- `@google/markerclustererplus` was replace for `@googlemaps/markerclusterer`
- The plugin exports two main objects:
  - `components`: it has all components and mountable mixin)
  - `helpers`: it has promise lazy factory function, gmapApi function and map element mixin
- The plugin now exports by default the install function, this means that you can do the following

  ```js
  import GmapVue from 'gmap-vue';
  ```

  instead of

  ```js
  import * as GmapVue from 'gmap-vue';
  ```

## Installation

### npm

```shell
npm install gmap-vue --save
```

### yarn

```shell
yarn add gmap-vue
```

### Manually

Just download `dist/gmap-vue.js` file and include it from your HTML.

```html
<script src="./gmap-vue.js"></script>
```

### jsdelivr

You can use a free CDN like [jsdelivr](https://www.jsdelivr.com) to include this plugin in your html file

```html
<script src="https://cdn.jsdelivr.net/npm/gmap-vue@1.2.2/dist/gmap-vue.min.js"></script>
```

### unpkg

You can use a free CDN like [unpkg](https://unpkg.com) to include this plugin in your html file

```html
<script src="https://unpkg.com/gmap-vue@1.2.2/dist/gmap-vue.js"></script>
```

::: warning
Be aware that if you use this method, you cannot use TitleCase for your components and your attributes.
That is, instead of writing `<GmapMap>`, you need to write `<gmap-map>`.
:::

[Live example](/guide/).

## Basic usage

### Get an API key from Google

[Generating an Google Maps API key](https://developers.google.com/maps/documentation/javascript/get-api-key).

### Quickstart (Webpack, Nuxt):

If you are using Webpack and Vue file components, just add the following to your code!

In your `main.js` or inside a Nuxt plugin:

```js
import Vue from 'vue'
import GmapVue from 'gmap-vue'

Vue.use(GmapVue, {
  load: {
    // [REQUIRED] This is the unique required value by Google Maps API
    key: 'YOUR_API_TOKEN',
    // [OPTIONAL] This is required if you use the Autocomplete plugin
    // OR: libraries: 'places,drawing'
    // OR: libraries: 'places,drawing,visualization'
    // (as you require)
    libraries: 'places',

    // [OPTIONAL] If you want to set the version, you can do so:
    v: '3.26',

    // If you already have an script tag that loads Google Maps API and you want to use it set you callback
    // here and our callback `GoogleMapsCallback` will execute your custom callback at the end; it must attached
    // to the `window` object, is the only requirement.
    customCallback: 'MyCustomCallback',
  },

  // [OPTIONAL] If you intend to programmatically custom event listener code
  // (e.g. `this.$refs.gmap.$on('zoom_changed', someFunc)`)
  // instead of going through Vue templates (e.g. `<GmapMap @zoom_changed="someFunc">`)
  // you might need to turn this on.
  autoBindAllEvents: false,

  // [OPTIONAL] If you want to manually install components, e.g.
  // import {GmapMarker} from 'gmap-vue/src/components/marker'
  // Vue.component('GmapMarker', GmapMarker)
  // then set installComponents to 'false'.
  // If you want to automatically install all the components this property must be set to 'true':
  installComponents: true,
})
```

::: tip

If you already have an script tag that loads Google Maps API and you want to use it set you callback in the `customCallback` option and our `GoogleMapsCallback` callback will execute your custom callback at the end.

 **It must attached to the `window` object**, is the only requirement.

```js
window.MyCustomCallback = () => { console.info('MyCustomCallback was executed') }
```

:::

In you components or `.vue` files add the following

```vue
<GmapMap
  :center="{lat:10, lng:10}"
  :zoom="7"
  map-type-id="terrain"
  style="width: 500px; height: 300px"
>
  <GmapMarker
    :key="index"
    v-for="(m, index) in markers"
    :position="m.position"
    :clickable="true"
    :draggable="true"
    @click="center=m.position"
  />
</GmapMap>
```

### The three main utilities

The plugin exports two main useful function and one important object, **they are references to the same memory places**.

In your components you have access to the following instance properties:

```js
  this.$gmapDefaultResizeBus; // this is the default resize function used if you not provide you own resize function
  this.$gmapApiPromiseLazy; // this is the promise that you need to wait to be sure that the plugin is ready
  this.$gmapOptions; // this is the final set of options passed to the Google Maps API
```

In the main Vue instance you have access to the following static properties:

```js
  Vue.$gmapDefaultResizeBus; // this is the default resize function used if you not provide you own resize function
  Vue.$gmapApiPromiseLazy; // this is the promise that you need to wait to be sure that the plugin is ready
  Vue.$gmapOptions; // this is the final set of options passed to the Google Maps API
```

### Getting a map reference

If you need to gain access to the `Map` instance (e.g. to call `panToBounds`, `panTo`):

```vue
<template>
  <GmapMap ref="mapRef" ...>
  </GmapMap>
</template>
<script>
  export default {
    mounted () {
      // At this point, the child GmapMap has been mounted, but
      // its map has not been initialized.
      // Therefore we need to write mapRef.$mapPromise.then(() => ...)

      this.$refs.mapRef.$mapPromise.then((map) => {
        map.panTo({lat: 1.38, lng: 103.80})
      })
    }
  }
</script>
```

### Accessing Google Maps API

If you need to access the Google maps API directly you can use the exported `gmapApi()` function like this

```vue
<template>
  <GmapMap ref="mapRef" ...>
    <GmapMarker ref="myMarker" :position="google && new google.maps.LatLng(1.38, 103.8)" />
  </GmapMap>
</template>
<script>
import { gmapApi } from 'gmap-vue';

export default {
  name: 'your-component-name',
  computed: {
    // The below example is the same as writing
    // google() {
    //   return gmapApi();
    // },
    google: gmapApi,
  },
};
</script>
```

You will get an object with a `maps` property and inside of it you can find all the Google maps API.

### Map Options

Control the options of the map with the options property. For more information about google [MapOptions](https://developers.google.com/maps/documentation/javascript/reference/map#MapOptions) visit the link.

 ```vue
 <GmapMap
  :options="{
    zoomControl: true,
    mapTypeControl: false,
    scaleControl: false,
    streetViewControl: false,
    rotateControl: false,
    fullscreenControl: true,
    disableDefaultUi: false
  }"
>
</GmapMap>
```

### Region and language localization

To add region and language localization. For more information about google [Localization](https://developers.google.com/maps/documentation/javascript/localization) visit the link.

```js
Vue.use(GmapVue, {
  load: {
    region: 'VI',
    language: 'vi',
  },
})
```

### Lazy loading

If you need to wait for google maps API to be ready, you can use the `this.$gmapApiPromiseLazy()` function to wait for it. This function is automatically added to your components when you use GmapVue plugin, also, you can find a reference of this function in your Vue instance if you have a reference of following common practice to assign it to `vm` variable.

A simple example

```js
export default {
  name: 'your-component-name',
  data() {
    return {
      center: { lat: 4.5, lng: 99 },
      markers: [],
    }
  },
  async mounted() {
    // if you have a reference of Vue in a `vm` variable
    // the following option should work too
    // await vm.$gmapApiPromiseLazy();

    await this.$gmapApiPromiseLazy();
    this.markers = [
      {
        location: new google.maps.LatLng({ lat: 3, lng: 101 }),
        weight: 100
      },
      {
        location: new google.maps.LatLng({ lat: 5, lng: 99 }),
        weight: 50
      },
      {
        location: new google.maps.LatLng({ lat: 6, lng: 97 }),
        weight: 80
      }
    ];
  }
};
```

### GmapMap slots

GmapMap component has two slots with a different behavior.
The default slot is wrapped in a class that sets `display: none;` so by default any component you add to your map will be invisible.

This is ok for most of the supplied components that interact directly with the Google map object, but it's not good if you want to bring up things like toolboxes, etc.

There is a second slot named **"visible"** that must be used if you want to display content within the responsive wrapper for the map, hence that's why you'll see this in the [drawing manager with slot example](/examples/drawing-manager-with-slot.html). It's actually not required in the [first example](/examples/drawing-manager.html) because the default toolbox is part of the Google map object.

> Thanks to [@davydnorris](https://github.com/davydnorris) to document this part of GmapVue.

### Exported utilities from the GmapVue plugin

In previous version the plugin exports all without any organization, from version 2 the plugin exports a default object with the install function required for Vue.js to install this plugin and two objects, `helpers` and `components`.

This two objects exports the same functions as before but in a better way, in the `main.js` file. The code below is a copy from that file.

```js

/**
 * Export all components and mixins
 */
export const components = {
  HeatmapLayer,
  KmlLayer,
  Marker,
  Polyline,
  Polygon,
  Circle,
  Cluster,
  Rectangle,
  DrawingManager,
  InfoWindow,
  MapLayer,
  PlaceInput,
  Autocomplete,
  MountableMixin,
  StreetViewPanorama,
};

/**
 * Export all helpers
 */

export const helpers = {
  gmapApi,
  loadGmapApi,
  MapElementMixin,
  MapElementFactory,
};

```

### Nuxt.js config

For Nuxt.js projects, please import GmapVue in the following way:

```js
import GmapVue from '~/node_modules/gmap-vue'
```

Add the following to your `nuxt.config.js`'s `build.extend()`:

```js
transpile: [/^gmap-vue($|\/)/]
```

::: details

Please take a look at this [example on stackblitz](https://stackblitz.com/edit/github-52grpx)

:::

### Officially supported components:

The list of officially support components are:

- Rectangle, Circle
- Polygon, Polyline
- KML Layer
- Marker
- InfoWindow
- Autocomplete
- Cluster* (via `marker-clusterer-plus`)
- Heat map
- Drawing map: rectangle, circle, polygon, line

Check our [documentation guide](/guide/) to see examples of every component.

For `Cluster`, you **must** import the class specifically, e.g.

```js
import { components } from "gmap-vue";
const { Cluster } = components;

Vue.component('GmapCluster', Cluster);
```

Inconvenient, but this means all other users don't have to bundle the marker clusterer package
in their source code.

### Adding your own components

It should be relatively easy to add your own components (e.g. Heatmap, GroundOverlay). please refer to the
[source code for `mapElementFactory`](https://github.com/diegoazh/gmap-vue/blob/master/packages/gmap-vue/src/utils/factories/map-element.js).

Example for [DirectionsRenderer](https://developers.google.com/maps/documentation/javascript/reference/3/#DirectionsRenderer):

```js
// DirectionsRenderer.js
import { helpers } from 'gmap-vue'
const { MapElementFactory } = helpers;

export default mapElementFactory({
  name: 'directionsRenderer',
  ctr: () => google.maps.DirectionsRenderer,
  //// The following is optional, but necessary if the constructor takes multiple arguments
  //// e.g. for GroundOverlay
  // ctrArgs: (options, otherProps) => [options],
  events: ['directions_changed'],

  // Mapped Props will automatically set up
  //   this.$watch('propertyName', (v) => instance.setPropertyName(v))
  //
  // If you specify `twoWay`, then it also sets up:
  //   google.maps.event.addListener(instance, 'propertyName_changed', () => {
  //     this.$emit('propertyName_changed', instance.getPropertyName())
  //   })
  //
  // If you specify `noBind`, then neither will be set up. You should manually
  // create your watchers in `afterCreate()`.
  mappedProps: {
    routeIndex: { type: Number },
    options: { type: Object },
    panel: { },
    directions: { type: Object },
    //// If you have a property that comes with a `_changed` event,
    //// you can specify `twoWay` to automatically bind the event, e.g. Map's `zoom`:
    // zoom: {type: Number, twoWay: true}
  },
  // Any other properties you want to bind. Note: Must be in Object notation
  props: {},
  // Actions you want to perform before creating the object instance using the
  // provided constructor (for example, you can modify the `options` object).
  // If you return a promise, execution will suspend until the promise resolves
  beforeCreate (options) {},
  // Actions to perform after creating the object instance.
  afterCreate (directionsRendererInstance) {},
})
```

Thereafter, it's easy to use the newly-minted component!

```vue
<template>
  <GmapMap :zoom="..." :center="...">
    <DirectionsRenderer />
  </GmapMap>
</template>
<script>
import DirectionsRenderer from './DirectionsRenderer.js'

export default {
  components: {DirectionsRenderer}
}
</script>
```

## Testing

More automated tests should be written to help new contributors.

Improvements to the tests are welcome :)
