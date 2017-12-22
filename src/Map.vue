<template>
    <div id="mapid"></div>
</template>

<script>
import L from 'leaflet';
import { antPath } from 'leaflet-ant-path';
import axios from 'axios';

export default {
    data() {
        return {
            mymap: null,
            biletomatyLayer: null,
            punktyLayer: null,
            biletomaty: [],
            punkty: []
        }
    },
    methods: {
        init() {
            this.mymap = L.map('mapid').setView([52.4, 16.9], 15);
            this.mymap.zoomControl.setPosition('topright');
            L.tileLayer('http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: 'Map data © <a href="http://openstreetmap.org">OpenStreetMap</a> contributors',
                maxZoom: 20,
            }).addTo(this.mymap);
            let customControl = L.Control.extend({
                options: {
                    position: 'topright'
                },
                onAdd: function (map) {
                    var container = L.DomUtil.create('a', 'leaflet-bar leaflet-control-zoom-out leaflet-control-custom');
                    container.style.backgroundColor = 'white';
                    container.style.width = '34px';
                    container.style.height = '34px';
                    container.onclick = function(){
                        console.log('buttonClicked');
                    }
                    return container;
                }
            })

            //add zoom control with your options
            this.mymap.addControl(new customControl());
        },
        getData() {
            "http://www.poznan.pl/mim/plan/map_service.html?mtype=environment&co=omnc01%20"
            axios.get("https://cors-anywhere.herokuapp.com/http://www.poznan.pl/mim/plan/map_service.html?mtype=pub_transport&co=class_objects&class_id=4000")
            .then(result => {
                this.biletomaty = result.data;
                this.biletomatyLayer.addData(this.biletomaty);
            })
            .catch(error => {
                console.log(error);
            })
            axios.get("https://cors-anywhere.herokuapp.com/http://www.poznan.pl/mim/plan/map_service.html?mtype=pub_transport&co=class_objects&class_id=4803")
            .then(result => {
                this.punkty = result.data;
                this.punktyLayer.addData(this.punkty);
            })
            .catch(error => {
                console.log(error);
            })
        },
        addPopup(feature, layer) {
            if (feature.properties) {
                layer.bindPopup(feature.properties.nazwa + '\n' + feature.properties.opis);
                //layer.bindPopup();
            }
        },
        showMarkers() {
            var myStyle = {
                "color": "#ff7800",
                "weight": 5,
                "opacity": 0.65
            };
            this.biletomatyLayer = L.geoJSON(null, {
                style: myStyle,
                onEachFeature: this.addPopup
                }).addTo(this.mymap);
            this.punktyLayer = L.geoJSON(null, {
                style: {
                    "color": "#ff0000"  
                },
                onEachFeature: this.addPopup
            }).addTo(this.mymap);
            /*
            geoJSON(states, {
                style: function(feature) {
                    switch (feature.properties.party) {
                        case 'Republican': return {color: "#ff0000"};
                        case 'Democrat':   return {color: "#0000ff"};
                    }
                }
            }).addTo(map);
            */
        },
        getCurrentPosition() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(position => {
                    console.log(position.coords.latitude, position.coords.longitude);
                    this.mymap.setView([position.coords.latitude, position.coords.longitude], 19);
                });
            } else {
                console.log("Api not available");
            }
        },
        onAdd(map) {
            var container = L.DomUtil.create('div', 'leaflet-bar leaflet-control leaflet-control-custom');
            container.style.backgroundColor = 'white';
            container.style.width = '30px';
            container.style.height = '30px';
            container.onclick = function(){
                console.log('buttonClicked');
            }
            return container;
        },
        createPath() {

        }
    },
    mounted() {
        /*
        console.log("test");
        /*
        const path = antPath(route,
            {"delay":400,"dashArray":[10,20],"weight":5,"color":"#0000FF","pulseColor":"#FFFFFF","paused":false}
        );
        const myMap = map('map').setView([0, 0], 13);
        tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        //myMap.addLayer(path);
        myMap.fitBounds(path.getBounds())
        */
        this.init();
        this.getCurrentPosition();
        this.showMarkers();
        this.getData();
        //this.showMarkers();
    },
    beforeCreate() {
        console.log('Nothing gets called before me!')
    }
}
</script>

<style>
    #mapid {
        height: 100%;
        width: 100%;
    }
</style>