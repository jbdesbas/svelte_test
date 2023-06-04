<script>
    /********
    Le select permet de récupérer la liste des communes sélectionnées, et le geojson de ces communes (feature collection)
    
    ********/
    import MySelect from "./SelectWFS.svelte";
    import Map from "./map.svelte";
    
    let selectedCommunes;
    let featuresCollection;
    
    function label(element){
        return `${element.properties.nom_min} (${element.properties.code}) `
    }
    
    
</script>

<h1>Welcome to SvelteKith</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>


<MySelect 
    bind:geojson={featuresCollection}
    group_by_dept={false} group={(e) => e.properties.code.substring(0,2) }
    wfs_endpoint='https://www.geo2france.fr/geoserver/spld/ows' layername='spld:communes' label={(e) => label(e)} search_field='nom' 
    placeholder="Nom de la commune"/>
    
    
<p>{#if featuresCollection} 
    {#each featuresCollection.features as commune} <li>{commune.properties.nom_min}</li>

    {/each}

{:else} Aucune sélection{/if}</p>

<Map geojson={featuresCollection}/>
