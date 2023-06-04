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
    
     export let wfs_endpoint = 'https://www.geo2france.fr/geoserver/spld/ows';
     export let layername = 'spld:communes';
     export let code_commune = 'code';
     export let nom_commune = 'nom_min';
     export let group_by_dept = false; 
     export let value = [];
     export let featuresCollection = {type:'FeatureCollection', 'features':[]};
     
     

     let filterText;
     let emptyText;
     let dataTable = aq.table({label:[], value:[], geom:[]});
     let options = dataTable.objects();

    function generate_geojson(){
            let features = [];
            value.forEach(function(e){
                features.push({type:'Feature', 'geometry':e.geom, 'properties':{'nom':e.label}})
            })
            featuresCollection.features = features;
    }   

    function group(e){
        if(group_by_dept){
            return e.group;
        }
        else{
            return null;
        }
    }
     
     async function fetchOptions() {
        //console.log(value)

        // Effectuez une requête à votre API avec les premières lettres tapées par l'utilisateur
         //console.log(filterText)
        if (filterText ==''){
            emptyText='Taper les premières lettres pour commencer'
            return []
        }
        const response = await fetch(`${wfs_endpoint}?SERVICE=WFS&REQUEST=GetFeature&VERSION=2.0.0&TYPENAMES=${layername}&OUTPUTFORMAT=application%2Fjson&&srsName=EPSG:4326&propertyName=${nom_commune},${code_commune},geometry&CQL_FILTER=${nom_commune}%20ilike%20%27${filterText}%25%27`);
        const data = await response.json();
        if (data.features.length < 1){
            emptyText='Aucune correspondance'
            return []
        }
        let data2 = [];
        data.features.forEach((c) => { //ajouter directement à la table arquero ?
            data2.push({value:c.properties.code, label:`${c.properties.nom_min} (${c.properties.code})`, geom:c.geometry} )
            }
        )
        dataTable = aq.from(data2)
        options = dataTable.derive({ name_length: d => aq.op.length(d.label), group: d=> aq.op.substring(d.value,0,2) }).orderby('name_length').objects();
        
        return options;
      }

    onMount(() => {
      //fetchOptions();
    });
</script>

<Select bind:filterText bind:value loadOptions={fetchOptions} placeholder="Nom de la commune" groupBy="{(e) => group(e)}" multiple on:change={generate_geojson} on:clear={generate_geojson}>
        <div slot="empty">{emptyText}</div>
</Select>


