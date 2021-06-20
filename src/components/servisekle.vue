<template>

    <button type="button" class="btn btn-warning rounded-0" data-toggle="modal" data-target="#exampleModal">
        Add Service
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
                        <input type="text" class="form-control" aria-describedby="emailHelp" placeholder="../geoserver/wfs" v-model="servicelayer.url">
                        <!--<small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>-->
                    </div>
                    <div class="form-group">
                        <label for="exampleInputEmail1">Reference Coordinate System:</label>
                        <input type="email" class="form-control" aria-describedby="emailHelp" placeholder="EPSG:3857" v-model="servicelayer.coordinatesystem">
                        <!--<small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>-->
                    </div>
                    <div class="form-group">
                        <label for="exampleInputPassword1">Version:</label>
                        <input type="text" class="form-control" v-model="servicelayer.version">
                    </div>
                    <div class="form-group">
                        <label for="exampleInputPassword1">Layer:</label>
                        <input type="text" class="form-control" v-model="servicelayer.layer">
                    </div>
                    <div class="form-group">
                        <label for="exampleInputPassword1">Layer2:</label>
                        <input type="text" class="form-control" v-model="servicelayer.layer2">
                    </div>

                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    <button type="button" class="btn btn-primary" v-on:click="addservicelayer()">Add</button>
                </div>
            </div>
        </div>
    </div>


</template>

<script>
    /* http://localhost:5050/geoserver/editdeneme/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=editdeneme%3Acizim&maxFeatures=50&outputFormat=application%2Fjson*/
    /*    WFS 1.1.0 ve 2.0.0, GML3'ü varsayýlan GML olarak döndürürken, WFS 1.0.0'da varsayýlan GML2'dir.*/
    export default {
        props: ["map"],
        components: {
        },
        data() {
            return {
                servicelayer: {
                    url: "http://localhost:5050/geoserver",
                    coordinatesystem: "EPSG:3857",
                    version: "1.1.0",
                    layer: "editdeneme",
                    layer2:"cizim"
                }
               
            }
        },
        created: function () {

        },
        methods: {
            addservicelayer() {

                var style = new ol.style.Style({
                    stroke: new ol.style.Stroke({
                        color: 'blue',
                        width: 1
                    }),
                    fill: new ol.style.Fill({
                        color: "red"
                    })
                });
                var url2 = this.servicelayer.url + '/' + this.servicelayer.layer + '/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=' + this.servicelayer.layer + '%3A' + this.servicelayer.layer2+'&maxFeatures=50&outputFormat=application%2Fjson';
                this.source = new ol.source.Vector({
                    url: url2,
                    format: new ol.format.GeoJSON({
                    })
                });

                var layer1 = new ol.layer.Vector({
                    className:"Feature Service",
                    source: this.source,
                    style: style,
                });

                this.map.addLayer(layer1);

            }


        }


    };
</script>

<style>
</style>
