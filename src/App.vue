<template>
  <div id="app">
    <l-map style="height: 100vh;"
           ref="lMap"
           :zoom.sync="zoom"
           :center.sync="center"
           :min-zoom="minZoom"
           :max-zoom="maxZoom"
           :options.sync="mapOptions"
           :bounds="bounds"
           :maxBounds="maxBounds"
           :maxBoundsViscosity="maxBoundsViscosity"
           :crs="crs">
      <l-tile-layer :url="url" />
      <l-marker :draggable=true />
      <editable-polyline ref="editablePolyline"
                         :latlngs.sync="adminToolBox.polyline.latlngs"
                         :editablePolylineOptions="adminToolBox.polyline.data"
                         :visible="adminToolBox.polyline.editMode"
      />
    </l-map>
  </div>
</template>

<script>
import {LMap, LTileLayer, LMarker} from 'vue2-leaflet'
import 'leaflet/dist/leaflet.css'
import L, {CRS, latLng} from 'leaflet'
import EditablePolyline from '../EditablePolyline'

export default {
  name: 'App',
  components: {
    EditablePolyline,
    LMap,
    LMarker,
    LTileLayer,
  },
  data () {
    return {
      crs: null,
      bounds: null,
      center: null,
      maxBounds: null,
      zoom: 4,
      minZoom: 3,
      maxZoom: 11,
      maxBoundsViscosity: 1,
      mapOptions: {
        zoomControl: true,
        zoomSnap: 1
      },
      url: 'https://nodes.alaatv.com/upload/raheAbrishamMap/{z}/{x}/{y}.png?v=123',
      adminToolBox: {
        marker: {
          id: 0,
          action: {},
          data: {
            latlng: {
              lng: 0,
              lat: 0
            },
            headline: {
              text: null,
              fontSize: 14
            },
            icon: {
              options: {
                iconAnchor: [0, 0],
                iconUrl: null,
                iconSize: [70, 70]
              }
            }
          },
          enable: 1,
          tags: [],
          min_zoom: 3,
          max_zoom: 11,
          editMode: false,
          type: {
            name: 'marker'
          }
        },
        polyline: {
          action: {},
          enable: 1,
          min_zoom: 3,
          max_zoom: 11,
          data: {
            line: {
              className: '',
              bubblingMouseEvents: false,
              options: {
                flowing: {
                  dir: 'fixed',
                  style: { 'animation-duration': 20 }
                }
              },
              color: 'red',
              weight: 5,
              dashArray: '10 30',
              dashOffset: '0'
            },
            displayZoom: 1,
            iconSize: [16, 16],
            iconAnchor: [10, 10]
          },
          latlngs: [[-5840, 23056], [-4000, 2500], [-9000, 8000]],
          editMode: true,
          tags: [],
          type: {
            name: 'polyline'
          }
        }
      },
    }
  },
  created() {
    this.crs = this.getCRS([0.00000000, -15426.00000000, 26934.00000000, 0.00000000]);
    this.bounds = [[-20140, 920 ], [-920, 28650 ]]; // down - left - up - right
    this.center = latLng(-8560, 21008);
    this.maxBounds = this.bounds;
  },
  methods: {
    getCRS(mapExtent) {
      let mapMaxZoom = 10,
          mapMaxResolution = 0.12500000,
          mapMinResolution = Math.pow(2, mapMaxZoom) * mapMaxResolution,
          tileExtent = mapExtent,
          crs = CRS.Simple;
      crs.transformation = new L.Transformation(1, -tileExtent[0], -1, tileExtent[3]);
      crs.scale = function(zoom) {
        return Math.pow(2, zoom) / mapMinResolution;
      };
      crs.zoom = function(scale) {
        return Math.log(scale * mapMinResolution) / Math.LN2;
      };

      return crs;
    },
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
#app {

}
</style>
