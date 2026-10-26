## Réponses niveau_0
## 0.2 Lire le schéma relationnel
### 1- Schéma relationnel
- categorie(**id**,libelle)
- plat(**id**,nom,prix,#id_categorie)
- commande(**id**,date_commande,numero_table)
- commande_ligne(**id**;#id_commande,#id_plat,quantite,prix_unitaire)


### 2 - 
 `commande_ligne` contient une colonne `prix_unitaire`, alors que le prix est déjà dans `plat` car celà facilite le calcul quand il y a beaucoup de données et celà aussi permet de garder les prix au moment de la commande en cas de changement de prix.

exemple : ce mois ci le prix du café est de 1000 ar mais le mois prochain ce sera 800 ar.

### 3- 
Les lignes de commande seront aussi supprimées si on supprime la commande car dans la commande_ligne il y a l'instruction ON DELETE CASCADE sur la colonne id_commande .

### 4 - Les 3 données absurdes :
- le prix peut être négatif 
- 
- 

## 0.3 Interroger l’API à la main
### 5-
- Pour la requête **b**,  le code 201 est obtenu car on a créé un élément donc POST , 200 c'est pour GET.

### 6-
- Pour la requête **d**, c'est PostgreSQL qui bloque la suppression. La ligne : 
```php 
return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.', 409); 
```

### 7- 
```bash 
curl -i -X POST http://localhost:8080/api/commandes \
     -H "Content-Type: application/json" \
     -d '{"numero_table":6,"lignes": [ { "id_plat": 4, "quantite": 2 },{ "id_plat": 10, "quantite": 1 } ]}'
```

## 0.4 Suivre une requête de bout en bout
### 8- 
- Où est le code métier de la création d’une commande ? dans services/CommandeService.php , fonction : creer()
- Pourquoi n’est-il pas dans le contrôleur Commandes.php ? c'est pour faciliter la maintenance du code et simplifier les modifications.

| # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | enregistrer() |
| 2 | La requête HTTP est construite | api.js | `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 | vite.config.js | '/api': 'http://localhost:8080'|
| 4 | CodeIgniter choisit le contrôleur | config/Routes.php | $routes->group('api', ['namespace' => 'App\Controllers\Api'], static function ($routes) { $routes->post('plats', 'Plats::create');}) |
| 5 | Le contrôleur lit le JSON | controller/ApiController.php | donneesJson()|
| 6 | Les règles de validation sont vérifiées | controller/Plats.php | if ($id === false) { return $this->erreurValidation($this->plats->errors());} |
| 7 | La ligne est insérée | controller/Plats.php |  return $this->respondCreated($this->plats->find($id)); |
| 8 | Le formulaire prévient l’écran parent | components/PlatForm.vue | `emit(…)` |
| 9 | L’écran recharge la liste | `views/CarteView.vue` | charger() |

## 0.5 Repérer les notions de Vue 
| Notion | À quoi ça sert | Votre exemple |
|---|---|---|
| `ref()` | une variable dont le changement met l’écran à jour | form (PlatForm.vue)|
| `computed()` | une valeur calculée à partir d’autres, recalculée automatiquement | const vueCourante = computed(() => onglets.find((o) => o.id === ongletActif.value).composant) (App.vue) |
| `v-for` | répéter un élément pour chaque élément d’une liste | v-for="onglet in onglets" (App.vue) |
| `v-if` / `v-else` | afficher un élément seulement si une condition est vraie |v-if="lignes.length === 0" (Panier.vue) |
| `v-model` | lier un champ de formulaire à une variable, dans les deux sens |v-model.number="numeroTable" (CommandeView.vue) |
| `defineProps` | recevoir des données du composant parent |defineProps({commande: { type: Object, required: true },}) (CommandeDetail.vue)|
 | `defineEmits` / `emit` | envoyer un événement au composant parent | const emit = defineEmits(['plus', 'moins']) (Panier.vue) |
| `onMounted` | exécuter du code quand le composant apparaît | onMounted(charger) (CarteView.vue) |

### 9 - 
Panier.vue ne modifie jamais la quantité lui-même car un composant enfant ne dois pas modifier directement les props. C'est le composant parent qui met à jour la quantité.

### 10-
Il fait la mise à jour du formulaire à chaque fois qu'on clique sur modifier un autre plat.

