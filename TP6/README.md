# TP6 — Système RAG agentique sur un rapport de futsal

## Objectif

Ce TP reprend le notebook du cours consacré au RAG (*Retrieval-Augmented Generation*). L’agent recherche d’abord des passages dans un document, puis s’en sert pour formuler une réponse. Le corpus choisi est le rapport de recherche sur la détection, le suivi et l’analyse d’actions au futsal.

Le notebook conserve le déroulement du professeur. Les appels OpenAI sont remplacés par Ollama local, et les données du rapport sont remplacées par le document de futsal.

## Fonctionnement

1. `PyPDFDirectoryLoader` charge les documents PDF du dossier `resources`.
2. `RecursiveCharacterTextSplitter` découpe le texte en segments de 1 000 tokens avec un chevauchement de 200 tokens.
3. `nomic-embed-text` calcule localement les embeddings ; Chroma indexe les segments en mémoire.
4. Le retriever renvoie les 10 passages les plus proches de la question.
5. L’outil `retriever_tool` donne ces passages à l’agent LangGraph.
6. `qwen3:0.6b`, exécuté sur Ollama, rédige une réponse fondée sur les extraits trouvés.

Le graphe suit le cycle agent → outil de recherche → agent. Le notebook n’utilise ni clé OpenAI ni clé Gemini.

## Préparer l’environnement sous Windows

Depuis le dossier `TP6`, ouvrez PowerShell et installez les dépendances :

```powershell
uv sync
```

L’application Ollama doit être démarrée. Téléchargez les deux modèles locaux si nécessaire :

```powershell
ollama pull qwen3:0.6b
ollama pull nomic-embed-text
ollama list
```

Dans VS Code, ouvrez `Tp6_RAG_Agentique.ipynb`, sélectionnez le noyau Python de `.venv`, puis exécutez les cellules dans l’ordre.

## Document source et confidentialité

Placez localement le fichier suivant dans `resources/` :

```text
Rapport_Futsal_AHMED_MILOUDI.pdf
```

Ce PDF contient des informations personnelles et **n’est pas inclus dans le dépôt**. Le `.gitignore` de TP6 l’exclut explicitement. Les chargements, embeddings et recherches du notebook utilisent l’instance Ollama locale à `http://127.0.0.1:11434`.

## Vérifier les résultats

La cellule de recherche manuelle affiche les passages récupérés avec le nom du fichier et la page PDF. Vérifiez ces extraits si une réponse indique que l’information n’est pas disponible, puis posez une question précise sur un sujet décrit dans le rapport.

Le notebook suit le comportement du notebook du cours : il ne conserve pas d’historique entre des appels séparés de l’agent.
