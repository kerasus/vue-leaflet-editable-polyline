<template>
    <div id="app">
        <button v-on:click="addMarker" style="background-color: forestgreen">add marker</button>
        <button @click="enough" style="background-color: orangered">stop add marker</button>
        <l-map style="height: 100vh;"
               ref="lMap"
               v-model:zoom="zoom"
               v-model:center="center"
               :min-zoom="minZoom"
               :max-zoom="maxZoom"
               v-model:options="mapOptions"
               :maxBoundsViscosity="maxBoundsViscosity"
               :crs="crs"
               @click="mapClick($event)"
               @update:zoom="zoomUpdated">
            <l-tile-layer :url="url"/>
            <editable-polyline ref="editablePolyline"
                               v-model:latlngs="adminToolBox.polyline.latlngs"
                               :editablePolylineOptions="adminToolBox.polyline.data"
                               :visible="adminToolBox.polyline.editMode"
                               @update:latlngs="newLatlngs"
            />
        </l-map>
    </div>
</template>

<script>
import {LMap, LTileLayer} from '@vue-leaflet/vue-leaflet'
import 'leaflet/dist/leaflet.css'
import L, {CRS, latLng} from 'leaflet'
import EditablePolyline from '../EditablePolyline'

export default {
    name: 'App',
    components: {
        EditablePolyline,
        LMap,
        LTileLayer,
    },
    data() {
        return {
            crs: null,
            bounds: null,
            center: null,
            maxBounds: null,
            zoom: 18,
            minZoom: 3,
            maxZoom: 18,
            maxBoundsViscosity: 1,
            mapOptions: {
                zoomControl: true,
                zoomSnap: 1
            },
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
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
                                    style: {'animation-duration': 20}
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
                    latlngs: [],
                    // latlngs: [
                    //     {
                    //         "lat": -12897.3115234375,
                    //         "lng": 21054.05078125
                    //     },
                    //     {
                    //         "lat": -12897.216796875,
                    //         "lng": 21054.01416015625
                    //     },
                    //     {
                    //         "lat": -12897.11474609375,
                    //         "lng": 21054.03464453125
                    //     },
                    //     {
                    //         "lat": -12897.096509325,
                    //         "lng": 21054.0468515625
                    //     },
                    //     {
                    //         "lat": -12897.091796875,
                    //         "lng": 21054.06982421875
                    //     },
                    //     // {
                    //     //     "lat": -12897.35595703125,
                    //     //     "lng": 21054.2490234375
                    //     // }
                    // ],
                    editMode: true,
                    tags: [],
                    type: {
                        name: 'polyline'
                    }
                }
            },
            enableToAddMarker: false
        }
    },
    created() {
        // console.log(document.getElementsByClassName("leaflet-interactive"))
        this.crs = this.getCRS([0.00000000, -15426.00000000, 26934.00000000, 0.00000000]);
        this.bounds = [[-20140, 920], [-920, 28650]]; // down - left - up - right
        this.center = latLng(-12897.3, 21054.1);
    },
    methods: {
        newLatlngs(latlngs) {
            this.adminToolBox.polyline.latlngs = latlngs
            // console.log(document.getElementsByClassName("leaflet-interactive"))
        },
        addMarker() {
            this.enableToAddMarker = true
        },
        enough() {
            this.enableToAddMarker = false
        },
        zoomUpdated() {
            console.log(this.center)
        },
        mapClick(event) {
            if (this.enableToAddMarker && event.latlng) {
                let lat = event.latlng.lat,
                    lng = event.latlng.lng,
                    // newPolyline = L.latLng(lat, lng)
                    newMarker = [lat, lng]
                this.adminToolBox.polyline.latlngs.push(newMarker)
                const index = this.adminToolBox.polyline.latlngs.length - 1
                if (index > 0) {
                    this.$refs.editablePolyline.addMidPointToEditablePolyline(index)
                }
            }
        },
        getCRS(mapExtent) {
            let mapMaxZoom = 10,
                mapMaxResolution = 0.12500000,
                mapMinResolution = Math.pow(2, mapMaxZoom) * mapMaxResolution,
                tileExtent = mapExtent,
                crs = CRS.Simple;
            crs.transformation = new L.Transformation(1, -tileExtent[0], -1, tileExtent[3]);
            crs.scale = function (zoom) {
                return Math.pow(2, zoom) / mapMinResolution;
            };
            crs.zoom = function (scale) {
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


</style>
