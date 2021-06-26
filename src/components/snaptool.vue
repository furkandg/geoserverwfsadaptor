<template>
    <div>
        <button class="btn btn-success rounded-0 btn-block" v-on:click="snapOpen" v-if="!snapprocess">Snap</button>
        <button class="btn btn-danger rounded-0 btn-block" v-on:click="snapClose" v-if="snapprocess">Snap</button>


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
                snapprocess:false
              
            }
        },
        created: function () {

        },
        methods: {
            snapOpen() {
                if (this.addedlayer) {
                    this.snapprocess = !this.snapprocess;
                    this.snap = new ol.interaction.Snap({
                        source: this.addedlayer.getSource(),
                        pixelTolerance: 10
                    });

                    this.map.addInteraction(this.snap)
                }
                else {
                    alert("Edit is not open")
                }

            },
            snapClose() {
                this.snapprocess = !this.snapprocess;
                this.map.removeInteractionByType(ol.interaction.Snap)
            }
        },
        watch: {
       
            addedlayer(o, n) {
                if (n) {
                    this.snapprocess = !this.snapprocess;
                }
                


            }

        }


    };
</script>

<style>
</style>
