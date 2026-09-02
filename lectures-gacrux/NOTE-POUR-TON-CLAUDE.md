# La voix de FrançaisPro — ce qu'il y a à brancher

Note à lire par le Claude Code qui travaille sur l'application Français Pro.
Écrite le 10 août 2026 par le terminal de YourRender.

---

## Pourquoi cette note existe

Aujourd'hui, l'application demande la voix **à l'appareil de l'élève** : « donne-moi
Audrey, sinon Amélie, sinon Thomas, sinon n'importe quelle voix française ». Sur un
téléphone qui n'a aucune de ces voix, l'élève entend ce qui traîne — parfois un accent
québécois, parfois une voix robotique, parfois rien.

Alexia a écouté 42 voix le 10 août 2026 et en a choisi une : **Gacrux**. Les 176
lectures de l'application sont maintenant des **fichiers**, identiques pour tous les
élèves, sur tous les téléphones, même sans connexion.

## Ce que tu reçois

Un dossier `lectures-gacrux/` prêt à poser dans les fichiers publics de l'application :

```
lectures-gacrux/
  lectures.json     le catalogue : 176 entrées { n, texte }
  001.mp3 … 176.mp3 une lecture par fichier, MP3 mono
```

Le champ `texte` de `lectures.json` est **exactement** la chaîne passée aujourd'hui à
`speak()` — c'est-à-dire le deuxième argument de `listen(...)` et de `listenMcq(...)`
dans `francaispro.html`, au caractère près, accents et ponctuation compris. Le champ
`n` est le nom du fichier sans l'extension.

## Tu n'as aucune voix à générer

C'est le point important, et c'est pour ça que cette note existe.

Les lectures sont **des fichiers, pas une génération**. Tu n'appelles aucun modèle,
aucune API, aucun service externe, et surtout aucune clé ne doit apparaître dans
l'application. Tu joues 176 MP3 déjà là, exactement comme tu afficherais un logo.

## Le contrat

**La clé est le texte, jamais le numéro.** Au démarrage, construis une table
`texte → fichier` depuis `lectures.json`. Quand une leçon veut faire entendre quelque
chose, cherche le texte dans cette table.

**Une seule fonction change dans ton code.** Aujourd'hui `speak(text, rate)` parle par
`window.speechSynthesis`. Elle doit d'abord regarder si le texte a un fichier :

- **fichier trouvé** → le jouer avec un élément `Audio` ;
- **fichier absent** → garder exactement le comportement actuel (la voix du navigateur).

Les six endroits qui appellent `speak(...)` ne bougent pas. Le repli n'est pas un détail
de confort : c'est lui qui fait qu'un nouveau chapitre écrit demain fonctionne le jour
même, avant que ses lectures soient fabriquées.

**Le bouton « Plus lentement » n'a pas besoin d'un deuxième jeu de fichiers.** Le même
MP3 se rejoue au ralenti en posant `playbackRate = 0.55` sur l'élément `Audio` — c'est
la transposition exacte du `rate` que tu passes aujourd'hui à la voix du navigateur.

**Rien ne se précharge.** Une lecture se charge quand l'élève appuie sur le bouton. Ne
mets pas les 176 fichiers dans la page : le dossier reste à côté, comme `avatars/`.

## Ce qu'il ne faut pas faire

- **Ne pas embarquer les MP3 en base64 dans `francaispro.html`.** La page pèserait
  plusieurs mégaoctets et se rechargerait en entier à chaque visite.
- **Ne pas renommer les fichiers ni réordonner `lectures.json`.** Si un texte change
  d'un seul caractère dans l'application, sa lecture ne sera plus trouvée — elle
  retombera sur la voix du navigateur, et c'est justement ce qu'on veut éviter.
- **Ne pas supprimer le chemin de repli** même quand les 176 fichiers sont en place.

## Quand de nouveaux chapitres arrivent

Leurs textes n'auront pas encore de fichier : le repli les fera lire par la voix du
navigateur, l'application continue de marcher. Signale-le à Alexia — les nouvelles
lectures sont produites ailleurs, avec la même voix Gacrux, et déposées dans ce même
dossier avec la suite des numéros. Ton code n'a pas à changer.

## Vérification avant de dire que c'est fait

1. Une leçon d'écoute jouée en vrai dans un navigateur : on entend Gacrux, pas la voix
   du téléphone.
2. Le bouton « Plus lentement » ralentit bien le même enregistrement.
3. Un texte volontairement modifié d'un caractère : l'application ne plante pas, elle
   repasse à la voix du navigateur.
4. Le poids de `francaispro.html` n'a pas bougé.
