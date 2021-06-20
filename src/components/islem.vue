<template>
    <div>
        <button class="btn btn-info rounded-0" v-on:click="ekle()">Add</button>
        <button class="btn btn-success rounded-0" v-on:click="duzenle()">Modify</button>
        <button class="btn btn-danger rounded-0" v-on:click="sil()">Delete</button>
    

    </div>
</template>

<script>

    /*    WFS 1.1.0 ve 2.0.0 = GML3, 1.0.0= GML2'dir.*/

    ol.Map.prototype.removeInteractionByType = function (ins) {
        var interactions = this.interactions.array_.filter(a => a instanceof ins);
        for (var i = 0; i < interactions.length; i++) {
            this.removeInteraction(interactions[i]);
        }
    }
    export default {

        props: ["map", "addedlayer"],
        components: {

        },
        data() {
            return {

                state: "select",
                servisState: {
                    url: "",
                    featureNs:""
                }
            }
        },
        created: function () {

        },
        methods: {
            bilgial() {
                console.log(this.addedlayer.getSource())
            },
            ekle() {
                var _self = this
                console.log(_self.addedlayer.schema)
                var draw = new ol.interaction.Draw({
                    source: this.addedlayer.getSource(),
                    type: this.addedlayer.values_.geoType,

                })
                this.map.addInteraction(draw);
                 draw.on('drawend', function (e) {

                    _self.transactWFS("insert", e.feature)
                    _self.map.removeInteractionByType(ol.interaction.Draw);
                });

            },
            sil() {

                var _self = this
                var select = new ol.interaction.Select();
                this.map.addInteraction(select);
                this.secilmis = select.getFeatures();
                select.on('select', function (evt) {

                    _self.addedlayer.getSource().removeFeature(evt.selected[0]);
                    _self.transactWFS("delete", evt.selected[0])
                    _self.map.removeInteractionByType(ol.interaction.Select);
                });
            },

            duzenle() {
                var _self = this;

                var select = new ol.interaction.Select();
                this.map.addInteraction(select);
                var edit = new ol.interaction.Modify({
                    features: select.getFeatures()
                });
                _self.map.addInteraction(edit)

                var dirty = {};
                select.getFeatures().on('add', function (e) {

                    e.element.on('change', function (e) {
                        dirty[e.target.getId()] = true;
                    });
                });
                select.getFeatures().on('remove', function (e) {

                    var f = e.element;
                    if (dirty[f.getId()]) {
                        delete dirty[f.getId()];
                        var featureProperties = f.getProperties();
                        delete featureProperties.boundedBy;
                        var clone = new ol.Feature(featureProperties);
                        clone.setId(f.getId());

                        _self.transactWFS("update", clone);
                        _self.map.removeInteractionByType(ol.interaction.Select);
                        _self.map.removeInteractionByType(ol.interaction.Draw);

                    }
                });
            },

            transactWFS(mode, f) {
                var _self = this
                var formatGML = new ol.format.GML({
                    featureNS: this.addedlayer.values_.featureNs,
                    featureType: this.addedlayer.className_
                   

                });
                var formatWFS = new ol.format.WFS();
                var xs = new XMLSerializer();
                var node;
                switch (mode) {
                    case 'insert':
                        node = formatWFS.writeTransaction([f], null, null, formatGML);
                        var payload = xs.serializeToString(node)
                        payload = payload.replace("</geometry>", "</geom>")
                        payload = payload.replace("<geometry>", "<geom>")
                        break
                    case 'update':
                        node = formatWFS.writeTransaction(null, [f], null, formatGML);
                        var payload = xs.serializeToString(node)
                        payload = payload.replace("geometry", "geom")
                        break

                    case 'delete':
                        node = formatWFS.writeTransaction(null, null, [f], formatGML);
                        var payload = xs.serializeToString(node)
                }
                $.ajax(this.addedlayer.values_.serviceUrl+'/ows', {
                    service: 'WFS',
                    type: 'POST',
                    dataType: 'xml',
                    processData: false,
                    contentType: 'text/xml',
                    data: payload,
                    success() {
                       
                    }
                }).done((response) => {
                  
                    this.addedlayer.getSource().refresh()
                });
            },
          
        },
        watch: {
       
          

        }


    };
</script>

<style>
</style>
