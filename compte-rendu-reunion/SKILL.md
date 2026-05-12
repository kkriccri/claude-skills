---
name: compte-rendu-reunion
description: >
  GÃ©nÃ¨re un compte rendu de rÃ©union professionnel au format DOCX Ã  partir d'un fichier audio (.m4a, .mp3, .wav, .ogg, .webm).
  Utilise ce skill dÃ¨s que l'utilisateur mentionne : compte rendu, CR de rÃ©union, minutes de rÃ©union, transcrire une rÃ©union,
  rÃ©sumÃ© de rÃ©union, procÃ¨s-verbal, PV de rÃ©union, ou fournit un fichier audio d'une rÃ©union/call/Teams/Zoom/Meet.
  MÃªme si l'utilisateur ne dit pas explicitement "compte rendu", si un fichier audio de rÃ©union est uploadÃ© avec du contexte
  (participants, projet, date), ce skill doit se dÃ©clencher.
---

# Compte Rendu de RÃ©union depuis un Audio

Ce skill transforme un enregistrement audio de rÃ©union en un compte rendu structurÃ© au format DOCX.

## Workflow

### Ãtape 1 : Collecter les informations de contexte

L'utilisateur doit fournir avec le fichier audio :
- La liste des participants (PrÃ©nom NOM + sociÃ©tÃ©/entitÃ©)
- La rÃ©fÃ©rence projet ou l'objet de la rÃ©union
- La date et l'heure de la rÃ©union
- Le type de rÃ©union (Teams, prÃ©sentiel, Zoom, etc.)

Si des informations manquent, les demander avant de commencer la transcription. Ne jamais inventer de participants, de dates ou de rÃ©fÃ©rences.

### Ãtape 2 : Transcrire l'audio

La transcription se fait avec Whisper (OpenAI). L'audio de rÃ©union peut Ãªtre long (30-60+ min), donc il faut dÃ©couper en chunks pour Ã©viter les timeouts.

#### ProcÃ©dure de transcription

1. VÃ©rifier la durÃ©e du fichier audio avec ffprobe
2. DÃ©couper en segments de 5 minutes avec ffmpeg :
   ```bash
   mkdir -p /sessions/WORKDIR/chunks
   ffmpeg -i "AUDIO_FILE" -f segment -segment_time 300 -c copy /sessions/WORKDIR/chunks/chunk_%03d.m4a
   ```
3. Installer whisper si nÃ©cessaire :
   ```bash
   pip install openai-whisper --break-system-packages
   ```
4. Transcrire chaque chunk sÃ©parÃ©ment avec le modÃ¨le `base` (meilleur compromis vitesse/qualitÃ©) :
   ```python
   export PATH="/sessions/WORKDIR/.local/bin:$PATH"
   python3 -c "
   import whisper
   model = whisper.load_model('base')
   result = model.transcribe('CHUNK_PATH', language='fr', fp16=False)
   with open('OUTPUT_PATH', 'w') as f:
       f.write(result['text'])
   print('DONE:', len(result['text']), 'chars')
   "
   ```
   Traiter chaque chunk dans un appel Bash sÃ©parÃ© pour Ã©viter les timeouts.
5. Combiner tous les fichiers de transcription

Le modÃ¨le `base` est le bon dÃ©faut. Si la qualitÃ© est insuffisante (termes techniques trÃ¨s mal reconnus), on peut rÃ©essayer un chunk avec `small`, mais c'est 3x plus lent.

### Ãtape 3 : Analyser et structurer le contenu

Lire la transcription complÃ¨te et en extraire :
- Les sujets abordÃ©s (regroupÃ©s par thÃ¨me)
- Les dÃ©cisions prises
- Les points d'information importants
- Les actions identifiÃ©es (qui fait quoi, pour quand)
- Les points en suspens ou questions ouvertes
- La date/objet de la prochaine rÃ©union si mentionnÃ©

**RÃ¨gles impÃ©ratives :**
- Ne rien inventer. Si un point n'est pas clair dans la transcription, le formuler avec prudence ("il semble que..." ou "sous rÃ©serve de confirmation...")
- Toujours Ã©crire PrÃ©nom NOM (pas juste le nom de famille)
- Ne jamais utiliser de tirets longs (â). Utiliser des points-virgules (;) pour sÃ©parer les Ã©lÃ©ments dans une mÃªme phrase
- La transcription automatique fait des erreurs : croiser le contexte pour reconstituer les noms propres, les termes techniques et les acronymes correctement

### Ãtape 4 : GÃ©nÃ©rer le DOCX

Utiliser le script `scripts/generate_cr.js` comme base. Le document doit contenir :

#### Structure obligatoire du document

1. **En-tÃªte** : nom de l'entreprise du rÃ©dacteur + rÃ©fÃ©rence projet (en haut de chaque page)
2. **Titre** : "COMPTE RENDU DE RÃUNION" + sous-titre projet
3. **Sommaire** (table des matiÃ¨res)
4. **Tableau d'informations** : Date, Type, Participants, RÃ©fÃ©rence projet, RÃ©dacteur
5. **Sections numÃ©rotÃ©es** : une par thÃ¨me abordÃ©, avec les sous-sections si nÃ©cessaire
6. **Tableau d'actions** : NÂ° ; Action ; Responsable ; ÃchÃ©ance â toujours en derniÃ¨re section numÃ©rotÃ©e
7. **Prochaine rÃ©union** (si applicable)
8. **Pied de page** : numÃ©ro de page centrÃ©

#### Mise en forme

- Police : Arial
- Couleur d'en-tÃªte des tableaux : bleu foncÃ© (#1F3864) avec texte blanc
- Titres H1 : bleu foncÃ© (#1F3864), gras
- Titres H2 : bleu moyen (#2E75B6), gras
- Listes Ã  puces avec bullet "â¢"
- Pas de tirets longs (â) nulle part dans le document ; utiliser ";" ou " - " si besoin de sÃ©paration

### Ãtape 5 : Livrer et avertir

- Sauvegarder le DOCX dans `/sessions/WORKDIR/mnt/outputs/`
- Nommage : `CR_[Objet]_[Date-YYYY-MM-DD].docx`
- Fournir le lien `computer:///sessions/WORKDIR/mnt/outputs/FILENAME.docx`
- Ajouter un avertissement que la transcription automatique peut contenir des approximations sur les noms propres et termes techniques
- RÃ©sumer les points clÃ©s en quelques lignes dans le message (pas plus de 10 lignes)

## RÃ©fÃ©rence

Consulter `references/docx_template.md` pour le template Node.js dÃ©taillÃ© de gÃ©nÃ©ration du DOCX.
