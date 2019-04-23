<template>
    <div class="fullscreen" id='simulation'></div>
</template>

<script>
    import mapboxgl from 'mapbox-gl/dist/mapbox-gl'
    import turf from 'turf'

    export default {
        data() {
            return {
                map: null,
                origin: [7.460684, 51.513106],
                destination: [13.421612, 52.560955],
                arc: [],
                steps: 500,
                counter: 0
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

            self.nextMovement(self.destination)

            for (var i = 0; i < self.lineDistance; i += self.lineDistance / self.steps) {
                var segment = turf.along(self.route.features[0], i, 'kilometers');
                self.arc.push(segment.geometry.coordinates);

            }

            self.route.features[0].geometry.coordinates = self.arc;
            self.map.on('load', function () {
// Add a source and layer displaying a point which will be animated in a circle.
                self.map.addSource('route', {
                    "type": "geojson",
                    "data": self.route
                });

                self.map.addSource('point', {
                    "type": "geojson",
                    "data": self.point
                });

                self.map.addLayer({
                    "id": "route",
                    "source": "route",
                    "type": "line",
                    "paint": {
                        "line-width": 2,
                        "line-color": "#007cbf"
                    }
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

                function animate() {
                    // Update point geometry to a new position based on counter denoting
                    // the index to access the arc.
                    self.point.features[0].geometry.coordinates = self.route.features[0].geometry.coordinates[self.counter];

                    // Calculate the bearing to ensure the icon is rotated to match the route arc
                    // The bearing is calculate between the current point and the next point, except
                    // at the end of the arc use the previous point and the current point
                    self.point.features[0].properties.bearing = turf.bearing(
                        turf.point(self.route.features[0].geometry.coordinates[self.counter >= self.steps ? self.counter - 1 : self.counter]),
                        turf.point(self.route.features[0].geometry.coordinates[self.counter >= self.steps ? self.counter : self.counter + 1])
                    );

                    // Update the source with self new data.
                    self.map.getSource('point').setData(self.point);

                    // Request the next frame of animation so long the end has not been reached.
                    if (self.counter < self.steps) {
                        requestAnimationFrame(animate);
                    }

                    self.counter = self.counter + 1;
                }

                // Start the animation.
                animate(self.counter);
            })
        },
        methods : {
            nextMovement(payload) {
                var nextMovementX = payload[0] + 0.001;
                var nextMovementY = payload[1] + 0.001;
                this.origin = [payload[0], payload[1]];
                this.destination = [nextMovementX, nextMovementY];
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

    #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
    }
</style>
