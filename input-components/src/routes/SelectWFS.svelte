<!--
@component SelectWFS
@description Composant qui permet de sélectionner une ou plusieurs entités depuis un flux WFS.

@prop {string} placeholder - Texte affiché dans le bouton.
@prop {string} wfs_endpoint - URL du service WFS utilisé pour récupérer les entités.
@prop {string} layername - Nom de la couche dans le service WFS contenant les entités.
@prop {string} search_field - Champ utilisé pour la recherche des entités.
@prop {function} label - Fonction qui renvoie l'étiquette à afficher pour chaque entité. Par défaut, utilise la valeur du champ `search_field`.
@prop {function} group - Fonction de regroupement des entités. Par défaut, aucun regroupement n'est effectué.
@prop {Object} geojson - Objet GeoJSON contenant les entités sélectionnées.


TODO : permettre de passer des id pour pré-remplir le select (recherche IN dans le WFS)
-->

<script>

    
     import { onMount } from 'svelte';
     import Select from 'svelte-select';
    
     export let placeholder;
     export let wfs_endpoint;
     export let layername ;
     export let search_field;
     export let label = function(e) { return e.properties[search_field] }
     export let group = function(e) { return null};
     
     export let geojson = {type:'FeatureCollection', 'features':[]};

     let filterText;
     let emptyText;
     let value = [];


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
        };
        let data2 = [];
        data.features.forEach((c) => { //ajouter directement à la table arquero ?
            data2.push({value:c, label:label(c)} )
            }
        );
        
        data2.sort(function (a, b) { //Sort by name length (short first)
            if (a.value.properties[search_field].length < b.value.properties[search_field].length){
                return -1;}
            else {
                return 1;}
        });

        return data2;
      }

    onMount(() => {
      //fetchOptions();
    });
</script>

<Select bind:filterText bind:value loadOptions={fetchOptions} placeholder={placeholder} groupBy="{(e) => group(e.value)}" multiple on:change={generate_geojson} on:clear={generate_geojson}>
        <div slot="empty">{emptyText}</div>
</Select>


