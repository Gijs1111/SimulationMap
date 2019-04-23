<template>
  <div class="fullscreen">
      <mapbox-map></mapbox-map>
  </div>
</template>

<script>
    import MapboxMap from '@/components/MapboxMap.vue'
    console.log(Map)
  export default {
        components: {MapboxMap}
  }

</script>

<style>
  body, html {
  margin: 0;
    padding: 0;
    width: 100%;
    height:100%;
  }
  .fullscreen {
    width: 100%;
    height:100%;
  }
</style>
