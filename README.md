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

**Requête**:

```SQL
PREFIX log: <http://www.w3.org/2000/10/swap/log#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>                                                                                                                                                                                                               
SELECT *
WHERE {
    ?x rdfs:subClassOf ?y.
}
``` 

La requête précédente fournit lesrésultats suivants:

```xml
<?xml version="1.0"?>
<sparql xmlns='http://www.w3.org/2005/sparql-results#'>
    <head>
        <variable name='x' />
        <variable name='y' />
    </head>
    <results>
        <result>
            <binding name='x'>
                <uri>http://www.w3.org/2002/07/owl#Thing</uri>
            </binding>
            <binding name='y'>
                <uri>http://www.w3.org/2002/07/owl#Thing</uri>
            </binding>
        </result>
        <result>
            <binding name='x'>
                <uri>http://www.w3.org/2002/07/owl#Nothing</uri>
            </binding>
            <binding name='y'>
                <uri>http://www.w3.org/2002/07/owl#Thing</uri>
            </binding>
        </result>
        <result>
            <binding name='x'>
                <uri>http://www.w3.org/2002/07/owl#Nothing</uri>
            </binding>
            <binding name='y'>
                <uri>http://www.w3.org/2002/07/owl#Nothing</uri>
            </binding>
        </result>
    </results>
</sparql>
```

## 2. Vérification de caractéristiques de propriétés et requêtes **SPARQL**

### Exercice 3 

> Identifier des propriétés de l'ontologie qui sont *sémantiquement* fonctionnelles, où une propriété entre deux instances est *fonctionnelle* lorsque qu'elle associe au plus une valeur à chaque objet;

Voici une courte sélection des propriétés fonctionnelles identifiables dans le fichier `ontology.xml`:

`hasBeenCreatedBy`

```xml
<owl:ObjectProperty rdf:about="https://www.datatourisme.gouv.fr/ontology/core#hasBeenCreatedBy">
    <owl:inverseOf rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#hasCreated" />
    <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#FunctionalProperty" />
    <rdfs:domain rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest" />
    <rdfs:range rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#Agent" />
    <rdfs:comment xml:lang="fr">L&apos;agent qui a créé ce POI dans le système d&apos;information.</rdfs:comment>
    <rdfs:comment xml:lang="en">The agent who has created the POI in the system.</rdfs:comment>
    <rdfs:label xml:lang="fr">a été créé par</rdfs:label>
    <rdfs:label xml:lang="en">has been created by</rdfs:label>
    <hasPriority rdf:datatype="http://www.w3.org/2001/XMLSchema#float">8.0</hasPriority>
</owl:ObjectProperty>
```

`hasEligiblePolicy`

```xml
<owl:ObjectProperty rdf:about="https://www.datatourisme.gouv.fr/ontology/core#hasEligiblePolicy">
    <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#FunctionalProperty" />
    <rdfs:domain rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#PriceSpecification" />
    <rdfs:range rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#PricingPolicy" />
    <rdfs:comment xml:lang="fr">La politique de prix qui s&apos;applique. Ex: Plein tarif.</rdfs:comment>
    <rdfs:comment xml:lang="en">The pricing policy the pricing applies on. Ex: Base rate.</rdfs:comment>
    <rdfs:label xml:lang="fr">a comme politique de tarification</rdfs:label>
    <rdfs:label xml:lang="en">has eligible policy</rdfs:label>
    <hasPriority rdf:datatype="http://www.w3.org/2001/XMLSchema#float">7.0</hasPriority>
</owl:ObjectProperty>
```

`worstRating`

```xml
<owl:DatatypeProperty rdf:about="https://www.datatourisme.gouv.fr/ontology/core#worstRating">
    <rdf:type rdf:resource="http://www.w3.org/2002/07/owl#FunctionalProperty" />
    <rdfs:domain rdf:resource="https://www.datatourisme.gouv.fr/ontology/core#ScaleReviewSystem" />
    <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#int" />
    <rdfs:comment xml:lang="fr">La pire valeur que ce système de classement propose.</rdfs:comment>
    <rdfs:comment xml:lang="en">The worst rating a scale review system provides.</rdfs:comment>
    <rdfs:label xml:lang="fr">Pire note</rdfs:label>
    <rdfs:label xml:lang="en">Worst rating</rdfs:label>
    <hasPriority rdf:datatype="http://www.w3.org/2001/XMLSchema#float">10.0</hasPriority>
</owl:DatatypeProperty>
```

> Extraire chacune de ces propriétés à l'aide d'une requête **SPARQL**;

```SQL
PREFIX <owl:http://www.w3.org/2002/07/owl#>
SELECT *
WHERE {
    ?fun a owl:FunctionalProperty
}
```

> Mettre en place un traitement `VerifFunctionelle` qui, pour un nom de propriété, récupère les triplets de cette propriété à l’aide d’une requête **SPARQL**, et vérifie qu’il s’agit bien d’une propriété fonctionnelle;

### Exercice 4

> La propriété `isLocatedAt` est transitive. Identifier si d’autres propriétés sont *sémantiquement* transitives;

> Mettre en place un traitement `VerifTransitive` qui, pour une propriété donnée, récupère les triplets de cette propriété à l’aide d’une requête **SPARQL**, et vérifie qu’il s’agit bien d’une propriété transitive.

## 3. Visualisation de données géolocalisées et requêtes **GraphQL**

### Exercice 5

> Que renvoie la requête **GraphQL** suivante:

```
{
    poi(
    filters:[{ 
        isLocatedAt: { 
            schema_address : { 
                schema_addressLocality : { 
                    _eq: "La Rochelle"
                }
            } 
        } 
    }])
    { 
        total results{rdf_type _uri rdfs_label{ value } }
    }
}
```

On obtient le résultat suivant, qui correspond aux **POI** localisé à la Rochelle:

```json
{
    "data": {
        "poi": {
            "total": 99,
            "results": [
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/LocalBusiness",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#SightseeingBoat",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsAndLeisurePlace"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/02242e2d-489f-3480-996c-48ef2eb193d6",
                    "rdfs_label": [
                        {
                            "value": "LA ROCHELLE CROISIERES"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/CivicStructure",
                        "http://schema.org/Event",
                        "https://www.datatourisme.gouv.fr/ontology/core#EntertainmentAndEvent",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlayArea",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsAndLeisurePlace",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsEvent"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/026949c8-aac9-32bf-9773-3d099675cb15",
                    "rdfs_label": [
                        {
                            "value": "CHASSE AU TR��SOR TERRA AVENTURA"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/LocalBusiness",
                        "http://schema.org/Museum",
                        "https://www.datatourisme.gouv.fr/ontology/core#CulturalSite",
                        "https://www.datatourisme.gouv.fr/ontology/core#Museum",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/04f7fba2-23d5-3049-92be-09226732139f",
                    "rdfs_label": [
                        {
                            "value": "MUSEE DU NOUVEAU MONDE"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsAndLeisurePlace"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/092111cc-2ff7-3759-b489-583f6321326b",
                    "rdfs_label": [
                        {
                            "value": "LE MARY LILI"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/LocalBusiness",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#SightseeingBoat",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsAndLeisurePlace"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0bd69952-5e85-3ed0-b4a1-d4150d4663d3",
                    "rdfs_label": [
                        {
                            "value": "ALDABRA YACHT CHARTER"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/Park",
                        "https://www.datatourisme.gouv.fr/ontology/core#CulturalSite",
                        "https://www.datatourisme.gouv.fr/ontology/core#ParkAndGarden",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0bdb85f6-8b93-3ee8-8966-2c7f406f5b87",
                    "rdfs_label": [
                        {
                            "value": "LE JARDIN DES PLANTES"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/LocalBusiness",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#SightseeingBoat",
                        "https://www.datatourisme.gouv.fr/ontology/core#SportsAndLeisurePlace"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0c07f4d8-9ef2-3d92-8dc8-a77f6bd98f4a",
                    "rdfs_label": [
                        {
                            "value": "AGENCE PAMPLEMOUSSE"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "https://www.datatourisme.gouv.fr/ontology/core#CulturalSite",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#RemarkableBuilding",
                        "https://www.datatourisme.gouv.fr/ontology/core#RemembranceSite",
                        "https://www.datatourisme.gouv.fr/ontology/core#Tower"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0c0d338a-ed34-30b4-ae4d-849ffc0e60b1",
                    "rdfs_label": [
                        {
                            "value": "CLOCHER ST BARTHELEMY"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "https://www.datatourisme.gouv.fr/ontology/core#ActivityProvider",
                        "https://www.datatourisme.gouv.fr/ontology/core#LeisureSportActivityProvider",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0dbf19c6-543a-3281-a651-af5d9a657b46",
                    "rdfs_label": [
                        {
                            "value": "ALTITUDES PARACHUTISME"
                        }
                    ]
                },
                {
                    "rdf_type": [
                        "urn:resource",
                        "http://schema.org/Park",
                        "https://www.datatourisme.gouv.fr/ontology/core#CulturalSite",
                        "https://www.datatourisme.gouv.fr/ontology/core#ParkAndGarden",
                        "https://www.datatourisme.gouv.fr/ontology/core#PlaceOfInterest",
                        "https://www.datatourisme.gouv.fr/ontology/core#PointOfInterest"
                    ],
                    "_uri": "https://data.datatourisme.gouv.fr/20/0de7d300-f87a-3c36-b26c-75e64cd46501",
                    "rdfs_label": [
                        {
                            "value": "PARC DE LA PORTE ROYALE"
                        }
                    ]
                }
            ]
        }
    }
}
```

### Exercice 6

> Ecrire une requête **GraphQL** qui renvoie:

> 1. Les noms des plages de Biarritz et de La Rochelle;

**Requête**:

```
{
  poi(filters: [{_or: [{isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "La Rochelle"}}}}, {isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "Biarritz"}}}}]}, {rdf_type: {_eq: "https://www.datatourisme.gouv.fr/ontology/core#Beach"}}]) {
    total
    results {
      rdfs_label {
        value
      }
    }
  }
}
```

**Résultats**:

```json
{
  "data": {
    "poi": {
      "total": 1,
      "results": [
        {
          "rdfs_label": [
            {
              "value": "Rocher de la Vierge"
            }
          ]
        }
      ]
    }
  }
}
```

> 2. Les noms des parcs de La Rochelle;

**Requête**:

```
{
  poi(filters: [{_or: [{isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "La Rochelle"}}}}, {rdf_type: {_eq: "http://schema.org/Park"}}]) {
    total
    results {
      isOwnedBy {
          schema_givenName
      }
    }
  }
}
```

**Résultats**:

```json
{
  "data": {
    "poi": {
      "total": 18,
      "results": [
        {
          "rdfs_label": [
            {
              "value": "LE JARDIN DES PLANTES"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC DE LA PORTE ROYALE"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC DE LA GARE"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC D'ORBIGNY"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC DES PERES"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "ALLEES DU MAIL"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "JARDIN DU MUSEE DU NOUVEAU MONDE"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PLAN D'EAU DE PORT NEUF"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC CHARRUYER"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "PARC FRANCK DELMAS"
            }
          ]
        }
      ]
    }
  }
}
```

> 3. Les noms des propriétaires des restaurants de La Rochelle;

**Requête**:

```
{
  poi(filters: [{isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "La Rochelle"}}}}, {rdf_type: {_eq: "https://www.datatourisme.gouv.fr/ontology/core#Restaurant"}}]) {
    total
    results {
      rdfs_label {
        value
      }
      isOwnedBy {
        schema_givenName
        schema_familyName
      }
    }
  }
}
```

**Résultats**:

```json
{
  "data": {
    "poi": {
      "total": 0,
      "results": []
    }
  }
}
```

> 4. Les noms des hôtels de Bordeaux acceptant des animaux;

**Requête**:

```
{
  poi(filters: [{isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "Bordeaux"}}}}, {rdf_type: {_eq: "https://www.datatourisme.gouv.fr/ontology/core#Hotel"}}, {isLocatedAt: {petsAllowed: {_eq: true}}}]) {
    total
    results {
      rdfs_label {
        value
      }
    }
  }
}
```

**Résultats**:

```json
{
  "data": {
    "poi": {
      "total": 11,
      "results": [
        {
          "rdfs_label": [
            {
              "value": "Eklo Bordeaux"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "H��tel Le Burdigala - Inwood-Hotels Bordeaux"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "Ibis Kitchen Lounge"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "Restaurant Campanile Bordeaux Le Lac"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "Mama Shelter Restaurant"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "H��tel Ibis Styles Bordeaux Meriadeck"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "Restaurant de l'H��tel Golden Tulip Bordeaux Euratlantique"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "H��tel Campanile Bordeaux Nord Le Lac"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "H��tel Mercure Bordeaux Centre"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "H��tel Novotel Bordeaux Centre"
            }
          ]
        }
      ]
    }
  }
}
```

> 5. Les plages de Nouvelle Aquitaine accessibles aux personnes à mobilité réduite;

**Requête**:

```
{
  poi(filters: [{rdf_type: {_eq: "https://www.datatourisme.gouv.fr/ontology/core#Beach"}}, {reducedMobilityAccess: {_eq: true}}]) {
    total
    results {
      rdfs_label {
        value
      }
    }
  }
}
}
```

**Résultats**:

```json
{
  "data": {
    "poi": {
      "total": 2,
      "results": [
        {
          "rdfs_label": [
            {
              "value": "R��servoirs de Piraillan"
            }
          ]
        },
        {
          "rdfs_label": [
            {
              "value": "Rocher de la Vierge"
            }
          ]
        }
      ]
    }
  }
}
```


### Exercice 7

> Ecrire une requête **GraphQL** qui renvoie les coordonnées géographiques de chaque **POI** de La Rochelle;

```
{
  poi(filters: [{isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "La Rochelle"}}}}]) {
    total
    results {
      rdf_type
      _uri
      rdfs_label {
        value
      }
      isLocatedAt {
        schema_geo {
          schema_latitude
          schema_longitude
        }
      }
    }
  }
```

> Mettre en place un traitement `geolocalisationPOI` qui affiche sur une carte tous les **POI** de La Rochelle;

> Modifier ce traitement afin qu'il affiche:
 - Les **POI** d'une ville donnée;
 - Les **POI** d'une ville donnée et d'un type défini (restaurant, cinéma, plage, etc...);
 - Les **POI** autour d’un point donné par sa géolocalisation;