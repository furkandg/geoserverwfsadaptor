<template>
    <div id="map" class="map">
        <div class="row" style="position:absolute;z-index:1000000;top:20px;left:60px;">
            <button type="button" class="btn btn-warning rounded-0" data-toggle="modal" data-target="#exampleModal">
                <i class="fas fa-globe"></i> Add Service
            </button>
            <!-- Modal -->
            <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="exampleModalLabel">Service Add Area</h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">
                            <div class="form-group">
                                <label for="exampleInputEmail1">Service Address:</label>
                                <input type="text" class="form-control" aria-describedby="emailHelp" v-on:keyup="commitValue" v-model="someValue" placeholder="../geoserver/namespace">
                                <small id="emailHelp" class="form-text text-muted">Add service link Please...</small>
                            </div>

                            <div class="form-group" v-if="items.length>0">
                                <label for="exampleFormControlSelect1">Layers:</label>
                                <select class="form-control" multiple aria-label="multiple select example" v-model="selectedLayers">
                                    <option v-for="item in items">{{ item.Ad }}</option>
                                </select>
                                <small id="emailHelp" class="form-text text-muted">You can multiple select!!</small>
                            </div>
                        </div>
                        <div class="modal-footer" v-if="items.length>0">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                            <!--<button type="button" class="btn btn-primary" v-on:click="addservicelayer()">Add</button>-->
                            <button type="button" class="btn btn-primary" v-on:click="dene()">Add</button>
                        </div>
                    </div>
                </div>
            </div>

            <islem v-bind:map="map" v-bind:addedlayer="addedlayer" />
        </div>
        <div class="p-3" style="background-color:#F8EFBA;position:absolute;z-index:1000000;top:59px;left:45px;width:230px;" v-if="items.length>0">
            <h5>Layer List</h5>

            <div class="form-check" v-for="itemcheck in selectedLayers">
                <input class="form-check-input" type="radio" :name="itemcheck" :value="itemcheck" v-model="checkradio">
                <label class="form-check-label" :for="itemcheck">
                    {{itemcheck}}
                </label>
            </div>
            <!--<button type="button" class="btn btn-dark text-white btn-block mt-2" v-on:click="openedit">{{addedlayer ? 'Edit Close' : 'Edit Open'}}</button>-->
        

            <div class="btn-group mt-2" role="group" aria-label="Basic example">
                <button type="button" class="btn btn-danger text-white btn-block  rounded-0" v-on:click="closeedit" v-if="addedlayer">Edit</button>
                <button type="button" class="btn btn-success text-white btn-block rounded-0" v-on:click="openedit" v-else="!addedlayer">Edit</button>
                <snaptool class="ml-1" v-bind:map="map" v-bind:addedlayer="addedlayer" />
            </div>





        </div>
        
    </div>
 
</template>



<script>

    import islem from './components/islem.vue';
    import snaptool from './components/snaptool.vue';

    export default {
        name: 'App',

        components: {
            islem,
            snaptool
        },
        data() {

            return {
                checkradio: "",
                someValue: "",
                selectedLayers: "",
                selectedLayers2: "",

                servicelayer: {
                    url: "",
                    coordinatesystem: "EPSG:3857",
                    version: "1.1.0",
                    namespace: "",
                    layer: "",
                    type: "",
                    featureNs: ""
                },
                map: "",
                source: "",
                addedlayer: "",
                items: [],
            }
        },
        mounted() {
            this.map = new ol.Map({
                target: 'map',
                layers: [
                    new ol.layer.Tile({
                        source: new ol.source.OSM()
                    })
                ],
                view: new ol.View({
                    center: ol.proj.fromLonLat([29.41, 40.82]),
                    zoom: 9
                })
            });


            var style = new ol.style.Style({
                stroke: new ol.style.Stroke({
                    color: 'blue',
                    width: 1
                }),
                fill: new ol.style.Fill({
                    color: "red"
                })
            });


        },
        computed: {
            commitValue() {
                $.ajax({
                    url: this.someValue + `/wfs?service=wfs&version=1.1.0&request=DescribeFeatureType&typeNames=` + this.someValue.split("/")[this.someValue.split("/").length - 1] + `&outputFormat=application%2Fjson`,
                    type: 'GET',
                    outputFormat: 'application/json',
                    exceptions: 'application/json',
                    success: (result) => {
                        result.featureTypes.forEach((item, index) => {
                            debugger
                            if (item.properties.filter(x => x.type.split(":")[0] == "gml")[0]) {
                                this.items.push(
                                    {
                                        "Ad": item.typeName,
                                        "GeoType": item.properties.filter(x => x.type.split(":")[0] = "gml")[0].localType
                                    }
                                )
                            }
                        });
                        this.servicelayer.featureNs = result.targetNamespace
                    }
                })

            },


        },
        methods: {
            openedit() {
                this.addedlayer = this.map.getLayers().array_.filter(x => x.className_ == this.checkradio)[0]
            },
            closeedit() {
                this.addedlayer = null;
                this.checkradio = "";
            },

            dene() {
                var i;
                for (i = 0; i < this.selectedLayers.length; i++) {
              
                    var url2 = this.someValue + "/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=" + this.someValue.split("/")[this.someValue.split("/").length - 1] + "%3A" + this.selectedLayers[i] + '&outputFormat=application%2Fjson';
                    var source = new ol.source.Vector({
                        url: url2,
                        format: new ol.format.GeoJSON({
                        })
                    });
                    var addedlayer = new ol.layer.Vector({
                        className: this.selectedLayers[i],
                        featureNs: this.servicelayer.featureNs,
                        schema: this.someValue.split("/")[this.someValue.split("/").length - 1],
                        source: source,
                        geoType: this.items.filter(x => x.Ad == this.selectedLayers[i])[0].GeoType,
                        serviceUrl: this.someValue
                    });
                    this.map.addLayer(addedlayer);


                }
            },
            addservicelayer() {
              
                var url2 = this.servicelayer.url + '/' + this.servicelayer.namespace + '/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=' + this.servicelayer.namespace + '%3A' + this.servicelayer.layer + '&outputFormat=application%2Fjson';
                this.source = new ol.source.Vector({
                    url: url2,
                    format: new ol.format.GeoJSON({
                    })
                });

                this.addedlayer = new ol.layer.Vector({
                    className: "Feature Service",

                    source: this.source,
                   
                });

                this.addedlayer.typeLayer = this.servicelayer.type

                this.map.addLayer(this.addedlayer);

            }
        },

    };
</script>
