# Entraînement LangGraph — Routage de tickets d’assistance

## Objectif

Ce travail est un exercice d’entraînement à LangGraph. Il présente deux usages :

1. Un graphe conditionnel qui oriente un ticket vers un traitement standard ou prioritaire.
2. Un agent Gemini qui peut appeler des outils de calcul.

## Exemple : routage d’un ticket

Le premier graphe transmet un état contenant l’identifiant du ticket, sa gravité, son temps d’attente, la décision et le résultat du traitement.

La gravité est interprétée sur une échelle de 1 à 5. La fonction d’analyse applique la règle suivante :

~~~text
urgent si severity >= 4 OU waiting_hours >= 48
standard sinon
~~~

Une seule des deux conditions suffit pour rendre le ticket urgent. La branche standard est choisie lorsque la gravité est inférieure à 4 et l’attente inférieure à 48 heures.

### Parcours du graphe

~~~mermaid
flowchart TD
    START --> analyze_ticket
    analyze_ticket -->|standard| standard_support_node
    analyze_ticket -->|urgent| urgent_support_node
    standard_support_node --> END
    urgent_support_node --> END
~~~

Le nœud d’analyse calcule la décision. Le routeur lit cette décision dans l’état et dirige le ticket vers le nœud de traitement adapté. Chaque branche ajoute un résultat à l’état, puis termine le graphe.

### Exemple exécuté et analyse

Le notebook lance le ticket suivant :

~~~python
{
    "ticket_id": "TCK-1042",
    "severity": 5,
    "waiting_hours": 8,
}
~~~

La sortie indique que le ticket **TCK-1042** est transmis en priorité. Son temps d’attente est inférieur à 48 heures, mais sa gravité vaut 5 et satisfait donc le premier critère d’urgence.

À titre de comparaison :

- Une gravité de 2 avec 12 heures d’attente mène au traitement standard.
- Une gravité de 2 avec 48 heures d’attente mène au traitement urgent.

Cette décision est déterministe et repose sur les règles du programme ; le LLM n’intervient pas dans ce premier graphe.

## Agent avec outils et Gemini

La seconde partie définit les outils **somme**, **produit** et **qotient**. Si Gemini demande l’exécution d’un outil, le graphe transmet la demande au nœud **tools**, puis revient à **assistant**. Lorsque la réponse ne demande plus d’outil, le graphe se termine.

La clé Gemini est chargée depuis un fichier local **.env**, exclu par le **.gitignore**. Ne publiez jamais ce fichier.

## Préparer et exécuter

Depuis le dossier **Entrainement**, installez les dépendances avec :

~~~powershell
uv sync
~~~

Créez ensuite localement un fichier **.env** contenant votre clé Gemini, sélectionnez le noyau **.venv** dans VS Code, puis exécutez les cellules du notebook dans l’ordre.

Le notebook sauvegardé conserve les compteurs d’exécution et la sortie du test du ticket **TCK-1042**.
