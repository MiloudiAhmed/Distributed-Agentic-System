# TP6 — Routage de tickets d’assistance avec LangGraph

## Objectif

Ce notebook présente deux usages de LangGraph :

1. Un graphe conditionnel qui oriente un ticket vers un traitement standard ou prioritaire.
2. Un agent qui peut appeler des outils de calcul, avec Gemini comme modèle conversationnel.

## Exemple : routage d’un ticket

L’état transmis dans le premier graphe contient :

| Champ | Signification |
| --- | --- |
| **ticket_id** | Identifiant du ticket |
| **severity** | Gravité, interprétée ici sur une échelle de 1 à 5 |
| **waiting_hours** | Nombre d’heures d’attente |
| **decision** | Branche choisie : **standard** ou **urgent** |
| **result** | Résultat du traitement |

La fonction **analyze_ticket** applique cette règle :

~~~text
urgent si severity >= 4 OU waiting_hours >= 48
standard sinon
~~~

Les deux conditions d’urgence sont indépendantes : une gravité élevée suffit, tout comme une longue attente. Le traitement standard est choisi seulement si la gravité est inférieure à 4 et si l’attente est inférieure à 48 heures.

### Parcours du graphe

~~~mermaid
flowchart TD
    START --> analyze_ticket
    analyze_ticket -->|standard| standard_support_node
    analyze_ticket -->|urgent| urgent_support_node
    standard_support_node --> END
    urgent_support_node --> END
~~~

Le routeur lit la décision calculée dans l’état, puis les arêtes conditionnelles choisissent le nœud correspondant. Chaque branche termine le graphe après avoir ajouté le texte du résultat à l’état.

### Résultat de l’exemple exécuté

Le notebook utilise le ticket suivant :

~~~python
{
    "ticket_id": "TCK-1042",
    "severity": 5,
    "waiting_hours": 8,
}
~~~

Le résultat est **urgent**. Même si le ticket n’attend que 8 heures, sa gravité vaut 5 et satisfait donc le premier critère. Le nœud **urgent_support_node** indique que le ticket est transmis en priorité.

Cette première démonstration est déterministe : la décision provient des règles écrites dans le code, sans intervention du LLM. Pour comparer, un ticket de gravité 2 qui attend depuis 12 heures suivrait la branche standard ; un ticket de gravité 2 qui attend depuis 48 heures serait urgent.

## Agent avec outils et Gemini

La seconde partie définit trois outils de calcul : **somme**, **produit** et **qotient**. L’agent Gemini peut demander leur exécution. Si la réponse du modèle contient un appel d’outil, le graphe l’envoie au nœud **tools**, puis revient au nœud **assistant**. S’il n’y a plus d’appel d’outil, le graphe se termine.

La clé **GEMINI_API_KEY** est conservée dans le fichier local **.env**. Ce fichier est exclu de Git par le **.gitignore** de TP6 et ne doit pas être publié.

## État du notebook

Le notebook sauvegardé contient les compteurs d’exécution des cellules et la sortie du test de routage, notamment le résultat prioritaire du ticket **TCK-1042**.
