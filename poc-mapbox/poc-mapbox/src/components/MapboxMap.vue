<template>

    <div class="fullscreen" id='simulation'>
        <button v-on:click="moveTo" id='replay'>Move to</button>
        <input v-model="longitude" placeholder="lng">
        <input v-model="latitude" placeholder="lat">
    </div>

</template>

<script>
    import mapboxgl from 'mapbox-gl/dist/mapbox-gl'
    import turf from 'turf'

    export default {
        data() {
            return {
                map: null,
                origin: [7.460684, 51.513106],
                destination: null,
                arc: [],
                steps: 1,
                counter: 0,
                longitude: null,
                latitude: null
            }
        },
        computed: {
            lineDistance() {
                return turf.lineDistance(this.route.features[0], 'kilometers')
            },
            route() {
                return {
                    "type": "FeatureCollection",
                    "features": [
                        {
                            "type": "Feature",
                            "geometry": {
                                "type": "LineString",
                                "coordinates": [
                                    this.origin,
                                    this.destination
                                ]
                            }
                        }
                    ]
                }
            },
            point() {
                return {
                    "type": "FeatureCollection",
                    "features": [{
                        "type": "Feature",
                        "properties": {},
                        "geometry": {
                            "type": "Point",
                            "coordinates": this.origin
                        }
                    }]
                }
            }
        },
        mounted() {
            var self = this

            mapboxgl.accessToken = 'pk.eyJ1IjoibWF1cml0c21hYXMiLCJhIjoiY2p1OXNmemtyMG1xeDQ0b2J5bHFhb3l5YyJ9.JuTW9fhzu23gGdnJ4Uze3g';
            self.map = new mapboxgl.Map({
                container: 'simulation', // container id
                style: 'mapbox://styles/mapbox/streets-v11', // stylesheet location
                center: [10, 51], // starting position [lng, lat]
                zoom: 5 // starting zoom
            });

            self.map.on('load', function ()
            {

                self.map.addSource('point', {
                    "type": "geojson",
                    "data": self.point
                });


                self.map.addLayer({
                    "id": "point",
                    "source": "point",
                    "type": "symbol",
                    "layout": {
                        "icon-image": "car-15",
                        "icon-rotation-alignment": "map",
                        "icon-allow-overlap": true,
                        "icon-ignore-placement": true
                    }
                });

            })
        },
        methods : {
            nextMovement(payload) {
                var self = this
                var nextMovementX = payload[0];
                var nextMovementY = payload[1];
                this.origin = [payload[0], payload[1]];
                this.destination = [nextMovementX, nextMovementY];

                self.animate()
            },
            animate () {
                var self = this

                // Update the source with self new data.
                self.map.getSource('point').setData(self.point);
                requestAnimationFrame(self.animate);
            },
            moveTo: function () {
                var coord = [ this.longitude, this.latitude ]
                this.nextMovement( coord )
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }

    #simulation {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
        height: 100%;
    }
</style>
