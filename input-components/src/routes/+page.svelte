<script>
    /********
    Le select permet de récupérer la liste des communes sélectionnées, et le geojson de ces communes (feature collection)
    
    ********/
    import MySelect from "./SelectWFS.svelte";
    import Map from "./map.svelte";
    
    let selectedCommunes;
    let featuresCollection;
    
</script>

<h1>Welcome to SvelteKith</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>


<MySelect bind:value={selectedCommunes} bind:featuresCollection group_by_dept={false} />
<p>{#if selectedCommunes} 
    {#each selectedCommunes as commune} <li>{commune.value}</li> {commune.geom }

    {/each}
<Map geojson={featuresCollection}/>
{:else} Aucune sélection{/if}</p>


