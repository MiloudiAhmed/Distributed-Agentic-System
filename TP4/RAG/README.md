# TP4 — Implémentation d’un système RAG

Ce dossier présente un système de génération augmentée par récupération (*Retrieval-Augmented Generation*, RAG). Le notebook charge un document PDF, le découpe en passages, indexe ces passages avec des embeddings, retrouve ceux qui correspondent à une question et demande à un modèle Gemini de répondre à partir du contexte récupéré.

## Contenu du dossier

- `rag.ipynb` : notebook du TP, de la préparation du document à l’évaluation de la réponse ;
- `pyproject.toml` et `uv.lock` : dépendances Python et versions verrouillées ;
- `rag_juge.gif` : illustration liée à l’évaluation ;
- `pdfs/` : dossier local à créer pour le PDF utilisé. Les documents PDF ne sont pas publiés dans le dépôt.

## Prérequis

- Python 3.11 ou une version compatible avec `pyproject.toml` ;
- [uv](https://docs.astral.sh/uv/) ;
- VS Code avec les extensions Python et Jupyter ;
- une clé API Gemini valide et une connexion Internet pour les appels au modèle et aux embeddings.

## Préparer l’environnement

Depuis la racine du dépôt, dans PowerShell :

```powershell
cd .\TP4\RAG
uv sync
```

Pour activer l’environnement dans le terminal PowerShell de VS Code :

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
```

L’environnement peut aussi être choisi directement comme kernel du notebook : sélectionnez l’interpréteur `.venv` du dossier `TP4\RAG`.

## Configurer Gemini

Créez un fichier `.env` dans `TP4\RAG` et ajoutez-y votre clé, sans la publier :

```dotenv
GEMINI_API_KEY=votre_cle_api_gemini
```

Remplacez la valeur d’exemple par votre clé réelle. Exécutez ensuite la cellule qui charge `.env` avant celles qui initialisent le modèle et les embeddings. Le fichier `.env` est exclu par `.gitignore` : ne copiez jamais la clé dans le notebook, le README ou un commit.

Le notebook configure Gemini pour la génération et les embeddings. Les appels nécessitent une clé valide, une connexion réseau et un quota disponible.

## Choisir le document PDF

Créez le dossier `pdfs` s’il n’existe pas, puis placez-y localement le document à analyser. Dans le notebook, adaptez la variable `pdf_file` au nom de votre fichier, par exemple :

```python
pdf_file = "./pdfs/mon_document.pdf"
```

Exécutez le notebook depuis le dossier `TP4\RAG` afin que ce chemin relatif soit résolu correctement. Utilisez de préférence un PDF dont le texte peut être sélectionné ; les PDF constitués uniquement d’images nécessitent une étape OCR qui n’est pas incluse dans ce TP.

Les fichiers PDF personnels et les bases Chroma générées (`store/` et `store_gemini/`) sont ignorés par Git. Ils doivent rester locaux.

## Exécuter le notebook

Ouvrez `rag.ipynb` dans VS Code, sélectionnez le kernel `.venv`, puis exécutez les cellules dans l’ordre, de haut en bas. Le notebook suit les étapes suivantes :

1. charger le PDF et le découper en *chunks* ;
2. créer les embeddings et indexer les passages dans Chroma ;
3. rechercher les passages pertinents pour une question avec un retriever ;
4. construire le prompt avec la question et le contexte récupéré ;
5. générer une réponse avec Gemini ;
6. regrouper la recherche et la génération dans la fonction `RAG` ;
7. évaluer la fidélité de la réponse au contexte (*groundness / faithfulness*).

Pour tester le système sur un autre document, adaptez le chemin PDF et les questions d’exemple dans le notebook. Si vous modifiez le modèle d’embeddings ou la structure des documents, recréez l’index Chroma correspondant avant de relancer la recherche.

## Confidentialité et publication

Le document source, la clé API et la base vectorielle ne doivent pas être ajoutés au dépôt. Attention : les sorties enregistrées dans un notebook peuvent elles-mêmes contenir des extraits du PDF, des réponses ou des coordonnées. Avant de publier ou de mettre à jour `rag.ipynb`, vérifiez chaque sortie et retirez les informations que vous ne souhaitez pas rendre publiques.

## Dépannage

- **Clé Gemini manquante** : vérifiez que `.env` se trouve dans `TP4\RAG`, que le nom de variable est exactement `GEMINI_API_KEY` et que la cellule de chargement a été exécutée avant l’initialisation des clients.
- **PDF introuvable** : vérifiez le dossier de travail du notebook et la valeur de `pdf_file` ; les PDF ne sont pas fournis par Git.
- **Kernel ou module introuvable** : sélectionnez le kernel `.venv` de `TP4\RAG`, puis lancez `uv sync` depuis ce dossier.
- **Appel Gemini refusé ou indisponible** : vérifiez la connexion, la validité de la clé et le quota du compte Gemini.
- **Réponse peu pertinente** : vérifiez les chunks retournés, reformulez la question ou ajustez la taille et le chevauchement des chunks avant de comparer les réponses.
