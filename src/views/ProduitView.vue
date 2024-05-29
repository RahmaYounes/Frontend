<template>
    <main>
        <div>
            <table>
                <caption> Les produits - page {{data.infoPage.number + 1}} / {{ data.infoPage.totalPages}}</caption>
                <caption>Liste des produits</caption>
                <tr>
                    <th>Nom</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Commandés</th>
                </tr>
                <!-- Si le tableau des produits est vide -->
                <tr v-if="!data.listeProduits">
                    <td colspan="4">Veuillez patienter, chargement des produits...</td>
                </tr>
                <!-- Si le tableau des catégories n'est pas vide -->
                <tr v-for="produit in data.listeProduits" :key="produit.code">
                    <td>{{ produit.nom }}</td>
                    <td>{{ produit.prixUnitaire }}</td>
                    <td>{{ produit.unitesEnStock }}</td>
                    <td>{{ produit.unitesCommandees }}</td>
                </tr>
                <tr>
                    <td>
                        <button @click="chargeProduit(data.links.first.href)">
                            Premiere page
                        </button>
                    </td>
                    <td>
                        <button @click="chargeProduit(data.links.prev.href)">
                            Page précédente
                        </button>
                    </td>
                    <td>
                        <button @click="chargeProduit(data.links.next.href)">
                            Page suivante
                        </button>
                    </td>
                    <td>
                        <button @click="chargeProduit(data.links.last.href)">
                            Dernière page
                        </button>
                    </td>
                </tr>
            </table>
        </div>
    </main>
</template>

<script setup>

import { reactive, onMounted } from "vue";
import { BACKEND, doAjaxRequest } from "../api";

// Pour réinitialiser le formuaire
const produitVide = {
    nom: "",
    prixUnitaire: "",
    unitesEnStock: "",
    unitesCommandees: ""
};

let data = reactive({
    // Les données saisies dans le formulaire
    formulaireProduit: { ...produitVide },
    // La liste des catégories affichée sous forme de table
    listeProduits: [],
    links : {},
    infoPage : {}
});

function chargeProduit(href) {
    // Appel à l'API pour avoir la liste des catégories
    // Trié par code, descendant
    // Verbe HTTP GET par défaut
    doAjaxRequest(href)
        .then((json) => {
            data.listeProduits = json._embedded.produits;
            data.links = json._links;
            data.infoPage = json.page;
        })
        .catch((error) => alert(error.message));
}

function ajouteProduit() {
    // Ajouter une catégorie avec les données du formulaire
    const options = {
        method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
        body: JSON.stringify(data.formulaireProduit),
        headers: {
            "Content-Type": "application/json",
        },
    };
    doAjaxRequest(BACKEND + "/api/produits", options)
        .then(() => {
            // Réinitialiser le formulaire
            data.formulaireProduit = { ...produitVide };
            // Recharger la liste des catégories
            chargeProduit();
        })
        .catch((error) => alert(error.message));
}
/**
 * Supprime une entité
 * @param entityRef l'URI de l'entité à supprimer
 */
/*
*function deleteEntity(entityRef) {
*    doAjaxRequest(entityRef, { method: "DELETE" })
*        .then(chargeCategories)
*        .catch((error) => alert(error.message));
}
*/

// A l'affichage du composant, on affiche la liste
onMounted(() => chargeProduit(BACKEND + "/api/produits?page=0&size=5"));

</script>


<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}


th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}
</style>