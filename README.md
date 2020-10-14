# WEB3.0 - Ontologie Datatourisme

### ANTELME Mathis

## 1. Prise en main des données

### Exercice 1

> Identifier les classes principales. Qu'est ce qu'un **POI** et quelles en sont les principales propriétés ?

Un **POI** ou *Point Of Interest*, est défini comme étant tout élément touristique qui mérite d'être décrit et valorisé. C'est un élément touristique qui est géré par un agent et qui peut être consommé via des produits et services. Il s'agit de la classe minimale a instancier afin de gérer un produit dans le système d'information.

Il se décompose en quatre sous-types différents:

1. **Produit**: (:`Product`:) Un objet touristique qui peut se consommer (ex: une chambre d'hôtel, une pratique d'activité, une visite guidée, ...);
2. **Itinéraire touristique**: (:`Tour`:) Un itinéraire touristique est un POI qui propose un itinéraire composé d’étapes formant un parcours;
3. **Fête et Manifestation**: (:`EntertainmentAndEvent`:) Manifestations, festivals, exposition, ou tout autre évènement ayant un début et une fin;
4. **Lieu d'intérêt**: (:`PlaceOfInterest`:) Un lieu ayant un intérêt touristique (ex: un site naturel, un site culturel, un village, un restaurant, ...);

De manière générale, un **POI** regroupe les propriétés suivantes:

| Information               	| Relation Sémantique  	| Description                                                                                                                                                                                                                                                                          	|
|---------------------------	|----------------------	|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| La localisation           	| [:isLocatedAt]       	| Où est localisé le POI et quels horaires y sont appliqués                                                                                                                                                                                                                            	|
| Les contacts              	| [:hasContact]        	| Qui contacter pour quel besoin                                                                                                                                                                                                                                                       	|
| Le propriétaire           	| [:isOwnedBy]         	| Un **POI** peut appartenir à un Agent via cette relation                                                                                                                                                                                                                             	|
| La consommation           	| [:offers]            	| Tarif et période pour consommer le produit. On notera que la consommation n’est possible qu’à travers une instance de :Offer. Voir la partie Tarifs pour plus de détails. Selon leurs types, les POI ne peuvent donc pas tous référencer directement des tarifs (POI non marchands). 	|
| L’audience                	| [:hasAudience]       	| L’audience à qui s’adresse le POI (public cible).                                                                                                                                                                                                                                    	|
| Multimédia                	| [:hasRepresentation] 	| Les documents qui sont des représentations du POI.                                                                                                                                                                                                                                   	|
| Les équipements           	| [:hasFeature]        	| Quels équipements sont disponibles et selon quelles cardinalités.                                                                                                                                                                                                                    	|
| Les classements et labels 	| [:hasReview]         	| Quels classements et labels évaluent le produit et avec quel score                                                                                                                                                                                                                   	|
| Les thèmes                	| [:hasTheme]          	| Quels thèmes sont associés au POI    

> Décrire la façon dont la géolocalisation des données touristiques est gérée;

La localisation des données touristiques sont décrites de plusieurs façons:

- Très génériques via la classe **Path**;
- A travers une ville via la classe **City**;
- A travers une instance de **schema:LocalBusiness**;
- A travers les coordonnées géographiques précises d’un lieu via les classes **schema:GeoCoordinates** et **schema:GeoShape**;
- A travers une instance de **schema:PostalAddress**;
- Ou en combinant ces concepts qui dérivent tous de **schema:Place**;

> Décrire les régles utilisées pour générer de nouveaux triplets (p.35);

Ci-après, les règles d’inférence qui peuvent être intégrées au moteur d’inférence qui sera utilisé pour la plateforme d’agrégation DATAtourisme. Ces règles permettent de générer automatiquement des triplets par raisonnement sémantique, sur la base de l’ontologie. Aucune d’entre elle n’est requise. Il s’agit surtout d’optimisations (compatibilité `schema.org` + facilite certaines potentielles requêtes **SPARQL**).

1. **Contact**: Permet de remplir automatiquement le champ [schema:addressLocality] d’une adresse postale, sur base de la ville sémantisée dans DATAtourisme, pour une compatibilité `schema.org`;

```rdf
IF ?addr :PostalAdress
 ?addr :hasAddressCity ?city
 ?city rdfs:label ?label
THEN
 ?addr schema:addressLocality ?label
END
```

2. **Multimedia**: Permet d’associer automatiquement un **POI** à une image pour une compatibilité `schema.org`;

```rdf
IF ?p a :PointOfInterest
 ?p :hasRepresentation ?eo
 ?eo a :Image
 ?eo ebucore:hasRealisation ?real
 ?real ebucore:locator ?url
THEN
 ?p schema:image ?image
 ?image a schema:ImageObject
 ?image schema:contentUrl ?url
END
```

3. **Localisation**: Permet d’associer automatiquement une Offer à la localisation du **POI** associé pour une compatibilité `schema.org`;

```rdf
IF ?p a :PointOfInterest
 ?p :isLocatedAt ?place
 ?p schema:offers ?offer
THEN
 ?offer schema:areaServed ?place
END
```

4. **Equipement**: Permet d’associer automatiquement un **POI** à un équipement dès lors que ce **POI** possède une capacité non vide liée à cet équipement.

```rdf
IF ?p a :PointOfInterest
 ?p :hasFeature ?a
 ?a :features ?f
THEN
 ?p :isEquippedWith ?f
END
```

5. **Période**: Permet d’ajouter automatiquement une date de début et une date de fin à un **POI** évènement dont la période est connue, pour une compatibilité `schema.org`;

```rdf
IF ?p a :EntertainmentAndEvent
 ?p :offers ?o
 ?o :takesPlaceAt ?p
 ?p :startDate ?start
 ?p :endDate ?end
THEN
 ?p schema:startDate ?start
 ?p schema:endDate ?end
END
```

6. **Classement**: Permet d’ajouter automatiquement la relation `schema.org` liant un **POI** et un classement de type ScaleReview, pour une compatibilité `schema.org`;

```rdf
IF ?p :hasReview ?r
 ?r a :ScaleReview
THEN
 ?p schema:review ?r
END
```

---

### Exercice 2

> Extraire la hiérarchie de classes des données (disponibles sous moodle) à l’aide d’une requête **SPARQL** et visualiser le résultat de la requête (vous pouvez utiliser l’outil `Webvowl`, http://www.visualdataweb.de/webvowl/);

Afin d'obtenir la hiérarchie de classe des données on va utiliser la requête **SPARQL** suivante:

> Requête:

```SQL
PREFIX log: <http://www.w3.org/2000/10/swap/log#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>                                                                                                                                                                                                               
SELECT *
WHERE {
    ?x rdfs:subClassOf ?y.
}
```

[Résultats](res/ontology.xml.json)    

## 2. Vérification de caractéristiques de propriétés et requêtes **SPARQL**

### Exercice 3 

> Identifier des propriétés de l'ontologie qui sont *sémantiquement* fonctionelles, où une propriété

>