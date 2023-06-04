<!--
@component
Ce composant permet de sélectionner une ou plusieurs communes dans un select depuis un flux wfs.
Le résultat peut être récupéré directement en geojson.

TODO : desactiver la récupération des géométries et du geojson
-->

<script>
  import { onMount } from 'svelte';
    import Select from 'svelte-select';
    import * as aq from 'arquero';
    
     export let placeholder;
     export let wfs_endpoint;
     export let layername ;
     export let search_field;
     export let group_by_dept = false; 
     export let label = function(e) { return e.properties[search_name] }
     export let group = function(e) { return null}; // Function
     
     export let value = [];
     export let geojson = {type:'FeatureCollection', 'features':[]};

     let filterText;
     let emptyText;
     let dataTable = aq.table({label:[], value:[]});
     let options = dataTable.objects();

    function generate_geojson(){
            let features = [];
            value.forEach(function(e){
                features.push(e.value)
            })
            geojson.features = features;
    }   

     
     async function fetchOptions() {
        //console.log(value)

        // Effectuez une requête à votre API avec les premières lettres tapées par l'utilisateur
         //console.log(filterText)
        if (filterText ==''){
            emptyText='Taper les premières lettres pour commencer'
            return []
        }
        const response = await fetch(`${wfs_endpoint}?SERVICE=WFS&REQUEST=GetFeature&VERSION=2.0.0&TYPENAMES=${layername}&OUTPUTFORMAT=application%2Fjson&&srsName=EPSG:4326&CQL_FILTER=${search_field}%20ilike%20%27${filterText}%25%27`);
        const data = await response.json();
        if (data.features.length < 1){
            emptyText='Aucune correspondance'
            return []
        }
        let data2 = [];
        data.features.forEach((c) => { //ajouter directement à la table arquero ?
            data2.push({value:c, label:label(c)} )
            }
        )
        dataTable = aq.from(data2)
        options = dataTable.derive({ name_length: d => aq.op.length(d.label) }).orderby('name_length').objects();
        
        return options;
      }

    onMount(() => {
      //fetchOptions();
    });
</script>

<Select bind:filterText bind:value loadOptions={fetchOptions} placeholder={placeholder} groupBy="{(e) => group(e.value)}" multiple on:change={generate_geojson} on:clear={generate_geojson}>
        <div slot="empty">{emptyText}</div>
</Select>


