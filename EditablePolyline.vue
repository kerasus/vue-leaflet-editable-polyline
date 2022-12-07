<template>
    <div>
        <l-polyline
            :lat-lngs="latlngs"
            :color="editablePolylineOptions.line.color"
            :className="editablePolylineOptions.line.className"
            :bubblingMouseEvents="editablePolylineOptions.line.bubblingMouseEvents"
            :dashArray="editablePolylineOptions.line.dashArray"
            :dashOffset="editablePolylineOptions.line.dashOffset"
            :weight="editablePolylineOptions.line.weight"
            :key="count"
        >
        </l-polyline>
        <template v-if="zoom >= editablePolylineOptions.displayZoom">
            <l-marker
                v-for="(item, index) in latlngs"
                :key="index"
                :lat-lng="item"
                :draggable="true"
                @dblclick="removeEditablePolylineLatLang($event, item, index)"
                @drag="updateEditablePolylineLatLangs($event, index)"
                @dragend="checkForAddPointToEditablePolylineLatLangs(index)">
                <l-icon
                    :icon-url="getEditablePolylineNodeImg(index)"
                    :icon-size="editablePolylineOptions.iconSize"
                    :icon-anchor="editablePolylineOptions.iconAnchor"
                >
                    <img :src="getEditablePolylineNodeImg(index)" alt=""/>
                </l-icon>
            </l-marker>
        </template>
    </div>
</template>

<script>
import {LMarker, LPolyline, LIcon} from '@vue-leaflet/vue-leaflet'

export default {
    name: "EditablePolyline",
    emits: ['update:latlngs'],
    props: {
        latlngs: {
            type: Array,
            default: () => []
        },
        zoom: {
            type: Number,
            default: 1
        },
        center: {
            type: Object,
            default: () => {
                return {lat: 0, lng: 0}
            }
        },
        editablePolylineOptions: {
            type: Object,
            default: () => {
                return {
                    line: {
                        color: 'red',
                        className: '',
                        bubblingMouseEvents: false,
                        weight: 5,
                        dashArray: '4 2 8',
                        dashOffset: '0'
                    },
                    displayZoom: 6,
                    iconSize: [16, 16],
                    iconAnchor: [10, 10]
                }
            }
        }
    },
    components: {
        LMarker,
        LIcon,
        LPolyline,
    },
    data() {
        return {
            editablePolylinelatlngs: [],
            count: 0
        }
    },
    watch: {
        latlngs: {
            handler(to) {
                console.log(to)
                this.count++
                this.editablePolylinelatlngs = to
            },
            deep: true
        }
    },
    methods: {
        updateLatlng() {
            this.$emit('update:latlngs', this.editablePolylinelatlngs)
        },
        getEditablePolylineNodeImg(index) {
            if (index % 2 === 0) {
                return 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAsAAAALCAYAAACprHcmAAAAAXNSR0IArs4c6QAAAAZiS0dEAP4ACgAK4wU5iwAAAAlwSFlzAAALEwAACxMBAJqcGAAAAAd0SU1FB90EFQswOAsixbAAAAAZdEVYdENvbW1lbnQAQ3JlYXRlZCB3aXRoIEdJTVBXgQ4XAAABDUlEQVQY042QMUoEURBEq3sQ9jeyGm86IpOIycI/gAaCF9gTGHoDzyDew8hMBhSM5ggjbLCZijCDyZ/E32Ww4wauwb6kg6pquloAIIeAYhjgZqcgz+leisgHVJ81pReMiJsBwB7JBwEuGCOkLIGuA5sG0vdLqsYipQ7j5tc8m7mvViRJJzczLxbuITCb7cPNrvJ0Su97/sV9HfP5nD6Z3MFDeMwx/qr8J0Gva2bVd6X7sVTV2ECwhQikqoCiOFBR/WTbrgVy20yCbQvk/I1sdrPTzSHcw80Ocwjc4RsnAgBuVpJcCgDGSClLQdeRTSPS94Dqmab0tGmUQxAA1wJcgjyCyBtFapC3xTB8AcAPM3L4CmzlQRcAAAAASUVORK5CYII=';
            } else {
                return 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAsAAAALCAYAAACprHcmAAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH3gEEBgwnW9DvzQAAABl0RVh0Q29tbWVudABDcmVhdGVkIHdpdGggR0lNUFeBDhcAAAEPSURBVBjTbZBBSkJxGMR/89cQnxAKFdQiCTeK0M5c1y08iifwPO46QIgHUJQWlctQ1E1P1N6bFiooNsuZH/N9jDhSGkWX2FdO04KkNSHMQhzPD7n2kGw/Ca7dbKJKBeZz3O+jxeLHIbxl4ngrgCSff6ZUKqjXQ+Uy3rcYcKuFul0svSqNontns4+aTFCxePwVtpGEGw0YDD4D9i212g60T2BJO6/TwZvNXXCaFlStHlLOJKFqFTKZi6AQNh6PD3fPYRuPx5AkabD0zWiEl8uzZts7r91Gudw0CL7YbnG9jieTHXT0QtpqwXCIpffDzpHtFwH/7Awh9EMcz07uJvn8g+AGu4C0tjTF/sisVr8Af3fdgpLllLoZAAAAAElFTkSuQmCC'
            }
        },
        resetToCenterOfMap() {
            let newCenter = JSON.parse(JSON.stringify(this.center));
            this.editablePolylinelatlngs = [
                {lat: newCenter.lat - 100, lng: newCenter.lng - 100},
                this.calcMiddlePoint({lat: newCenter.lat - 100, lng: newCenter.lng - 100}, {
                    lat: newCenter.lat + 100,
                    lng: newCenter.lng + 100
                }),
                {lat: newCenter.lat + 100, lng: newCenter.lng + 100}
            ];
            this.updateLatlng();
        },
        calcMiddlePoint(latlang1, latlang2) {
            if (latlang1.lat) {
                return {
                    lat: (latlang1.lat + latlang2.lat) / 2,
                    lng: (latlang1.lng + latlang2.lng) / 2
                }
            } else {
                return {
                    lat: (latlang1[0] + latlang2[0]) / 2,
                    lng: (latlang1[1] + latlang2[1]) / 2
                }
            }
        },
        removeEditablePolylineLatLang(event, item, index) {
            if (
                index === this.editablePolylinelatlngs.length - 1 ||
                this.editablePolylinelatlngs.length <= 3
            ) {
                return;
            }

            this.editablePolylinelatlngs.splice(index + 1, 1)
            this.editablePolylinelatlngs.splice(index, 1)

            this.updateLatlng();
        },
        updateEditablePolylineLatLangs(event, item, index) {
            const lat = event.latlng.lat,
                lng = event.latlng.lng
            this.editablePolylinelatlngs[index] = [lat, lng]
            this.updateLatlng();
        },
        addMidPointToEditablePolyline(index) {
            let latlang1 = this.editablePolylinelatlngs[index - 1],
                latlangIndex = this.editablePolylinelatlngs[index],
                mid1 = this.calcMiddlePoint(latlang1, latlangIndex)
            this.editablePolylinelatlngs.splice(index, 0, mid1)
            this.editablePolylinelatlngs[index] = mid1
            this.updateLatlng();
        },
        checkForAddPointToEditablePolylineLatLangs(index) {
            if (index % 2 === 0) {
                return
            }
            let latlang1 = this.editablePolylinelatlngs[index - 1],
                latlangIndex = this.editablePolylinelatlngs[index],
                latlang2 = this.editablePolylinelatlngs[index + 1],
                mid1 = this.calcMiddlePoint(latlang1, latlangIndex),
                mid2 = this.calcMiddlePoint(latlangIndex, latlang2);
            this.editablePolylinelatlngs.splice(index, 0, mid1)
            // this.editablePolylinelatlngs[index] = mid1
            this.editablePolylinelatlngs.splice(index + 2, 0, mid2)
            // this.editablePolylinelatlngs[index + 2] = mid2
            this.updateLatlng();
        }
    },
    created() {
        this.editablePolylinelatlngs = this.latlngs;
        // this.updateLatlng();
    }
}
</script>

<style scoped>

</style>
