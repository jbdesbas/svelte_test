<script>
    import { onMount, onDestroy } from 'svelte';
    import { Map } from 'maplibre-gl';
    import 'maplibre-gl/dist/maplibre-gl.css';
    
    export const wfs_endpoint = 'https://www.geo2france.fr/geoserver/spld/ows';
    export const layername = 'spld:communes';
    export const code_commune = 'code';
    export const nom_commune = 'nom_min';
    export const groupByDept = true; //wip
    export let geojson;


   let map = null;
   let mapContainer;
   
     $: {
        if (map != null) {
            if (map.loaded()){
                map.getSource('maine').setData(geojson)
                console.log(geojson)
            }else{
                map.on('load', () => map.getSource('maine').setData(geojson))
            }
        }
    }
   
   onMount( () => {
      
        const initialState = { lng: 2.9, lat: 49.6844, zoom: 8 };

        map = new Map({
          container: mapContainer,
          style: 'https://api.maptiler.com/maps/streets/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL',
          center: [initialState.lng, initialState.lat],
          zoom: initialState.zoom
        });
        
        map.on('load', function(){
            map.addSource('maine', {type:'geojson', data:geojson});
            map.addLayer({
                'id': 'maine',
                'type': 'fill',
                'source': 'maine',
                'layout': {},
                'paint': {
                'fill-color': '#088',
                'fill-opacity': 0.8
                }
            });
        });
           
    });
    
  onDestroy(() => {
    console.log('destroy')
    if (map != null) { map.remove(); }
  });
</script>


<div class="map-wrap">
  <a href="https://www.maptiler.com" class="watermark"><img
    src="https://api.maptiler.com/resources/logo.svg" alt="MapTiler logo"/></a>
  <div class="map" id="map" bind:this={mapContainer}></div>
</div>

<style>

  .map-wrap {
    position: relative;
    width: 100%;
    height: calc(100vh - 77px); /* calculate height of the screen minus the heading */
  }

  .map {
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .watermark {
    position: absolute;
    left: 10px;
    bottom: 10px;
    z-index: 999;
  }
</style>
