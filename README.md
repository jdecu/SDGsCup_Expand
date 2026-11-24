# SDGs Cup – Expand Consulting

Site de suivi de la SDGs Cup : les membres déclarent les défis relevés, les départements gagnent des points et le classement se met à jour en direct.

- **Site en ligne** : https://sdgs-cup-expand.web.app
- **Code** : un seul fichier, `index.html` (+ `logo.png` pour le logo)
- **Données en direct** : Firebase Realtime Database (projet `sdgs-cup-expand`)
- **Hébergement** : Firebase Hosting, déployé automatiquement à chaque `git push` sur la branche principale

---

## 1. Modifier le site (marche à suivre générale)

1. Ouvre `index.html` dans un éditeur (VS Code, Bloc-notes…).
2. Modifie le bloc `CONFIG` en haut de la balise `<script>` (tout se règle là, voir ci-dessous).
3. Envoie sur GitHub :
   ```
   git add .
   git commit -m "Mise à jour des défis"
   git push
   ```
4. Dans l'onglet **Actions** du repo GitHub, attends la coche verte (environ 1 minute).
5. Ouvre le site avec un rechargement forcé (**Ctrl + Maj + R**, ou ferme et rouvre l'onglet sur téléphone).

> **Tester sans toucher aux vraies données** : mets temporairement `firebase: null` dans `CONFIG` et ouvre `index.html` dans ton navigateur. Le site passe en « Mode démo » (données stockées uniquement sur ton appareil). Remets ta config Firebase avant de publier !

---

## 2. Les réglages de `CONFIG`

| Réglage | Rôle | Exemple |
|---|---|---|
| `start` | Date de lancement. La semaine S1 s'ouvre ce jour-là, S2 7 jours après, etc. | `"2027-02-01"` |
| `onlyCurrentWeek` | `false` : quand S2 s'ouvre, les défis de S1 restent postables (rattrapage). `true` : seuls les défis de la semaine en cours sont postables. | `false` |
| `adminPin` | PIN du mode organisateur. **À changer avant le lancement.** | `"1234"` |
| `weeks` | Liste des semaines. | `["S1", "S2", …, "S11"]` |
| `weekOfOdd` | Dans quelle semaine tombent les défis d'un ODD. | `{1:"S1", 7:"S2", …}` |
| `teams` | Les départements. | `["CxO", "HR", …]` |
| `teamPins` | PIN de chaque département (pour vérifier l'identité des membres). | `CxO: "5678"` |
| `challenges` | La liste des défis. | voir ci-dessous |
| `firebase` | Config de la base de données (ne pas toucher). | |

Hors de `CONFIG`, juste au-dessus, deux constantes utiles :

```js
const PTS = {easy:5, medium:10, hard:15};   // points par niveau
const TEAM_COLORS = ["#008A5E", …];          // couleurs des départements (par ordre de teams)
```

---

## 3. Gérer les défis

### Format d'une ligne de défi

```js
c("s1e1", 3, "easy", "Texte du défi affiché aux membres"),
//  │      │   │       └ description
//  │      │   └ niveau : "easy" | "medium" | "hard"
//  │      └ numéro de l'ODD (1 à 17)
//  └ identifiant unique du défi
```

Options facultatives, en 5ᵉ position : `{rep:true, unit:"2 €", pts:1, w:"S2"}`.

### Ajouter un défi

1. Dans `challenges`, copie une ligne existante et colle-la au bon endroit.
2. Donne-lui un **identifiant unique** (ex. `"s2e4"`), jamais utilisé auparavant.
3. Choisis le niveau, l'ODD et écris le texte.

```js
c("s2e4", 7, "easy", "Éteins tes appareils en veille pendant 3 jours"),
```

Le défi arrive dans la semaine liée à son ODD (voir « Changer la semaine d'un défi »). La convention actuelle est **3 faciles + 3 moyens + 3 difficiles par semaine**, tous réalisables en une semaine maximum.

### Modifier le texte, le niveau ou l'ODD

Change simplement la ligne. Garde le même identifiant pour que les posts déjà faits restent liés au défi.

### Supprimer un défi

Supprime la ligne. Attention : les posts déjà faits pour ce défi s'afficheront comme « Défi supprimé » et vaudront **0 point**. Pour les nettoyer, supprime-les en mode organisateur.

### Changer les points

**Pour tous les défis d'un niveau** : modifie `PTS`.

```js
const PTS = {easy:5, medium:10, hard:15};
```

**Pour un seul défi** : ajoute l'option `pts`.

```js
c("s3h1", 13, "hard", "Semaine végé : …", {pts:20}),
```

> **Les points sont recalculés à chaque affichage.** Changer les points d'un défi modifie aussi les points des posts déjà faits et donc le classement. À éviter une fois la compétition lancée.

### Défi répétable

Un membre ne peut normalement valider un défi qu'une seule fois. Avec `rep:true`, il peut le re-poster autant de fois qu'il le fait.

```js
c("s2e2", 7, "easy", "Trajet doux : …", {rep:true}),
```

### Points par tranche (ex. « +1 pt par 2 € »)

Avec `unit`, le formulaire demande un nombre de tranches et les points sont multipliés.

```js
c("s2m2", 8, "medium", "Achats éthiques : +1 pt par 2 € …", {rep:true, unit:"2 €", pts:1}),
```

---

## 4. Gérer les semaines

### Comment une semaine s'ouvre

Automatiquement, à partir de `start` : S1 le jour de départ, S2 7 jours plus tard, S3 14 jours plus tard… Avant `start`, aucune semaine n'est visible et l'accueil affiche « Les défis démarrent le … ». Les membres ne voient que les semaines déjà ouvertes (pas de S3 avec cadenas en S1).

### Changer la semaine d'un défi

**Pour tous les défis d'un ODD** : modifie `weekOfOdd`.

```js
weekOfOdd: {1:"S1", 2:"S1", 3:"S1", 4:"S1", 7:"S2", 8:"S2", 10:"S2", 11:"S2", 12:"S3", 13:"S3", 14:"S3", 15:"S3", 16:"S4", 17:"S4"},
```

Un ODD qui n'est pas dans cette liste n'est dans aucune semaine : ses défis ne s'affichent pas.

**Pour un seul défi** : ajoute `w`, qui prend le dessus sur `weekOfOdd`.

```js
c("s3e2", 12, "easy", "Explique le développement durable à un pote", {w:"S1"}),
```

### Ajouter ou retirer des semaines

Modifie `weeks` (ex. passer à 6 semaines) :

```js
weeks: ["S1", "S2", "S3", "S4", "S5", "S6"],
```

Une semaine sans défi affiche « Pas encore de défi ».

### Changer la date de départ

Modifie `start`. Toutes les semaines se décalent automatiquement.

### Rattrapage des semaines passées

- `onlyCurrentWeek: false` (par défaut) : quand S2 s'ouvre, les défis de S1 restent postables.
- `onlyCurrentWeek: true` : seuls les défis de la semaine en cours sont postables.

### Règle de publication

Un défi ne peut être posté que si sa semaine est ouverte. Ça vaut pour tout le monde, organisateurs compris, et c'est revérifié au moment de publier. Ce contrôle se fait dans le navigateur.

---

## 4 bis. Gérer les départements

- **Ajouter ou renommer** : modifie `teams` **et** ajoute/modifie la ligne correspondante dans `teamPins`. Renommer un département ne renomme pas les anciens posts.
- **Changer un PIN** : modifie `teamPins`. Les membres déjà vérifiés restent connectés sur leur appareil. Pour forcer une nouvelle vérification, il faut qu'ils vident les données du site.
- **Couleur** : `TEAM_COLORS` suit l'ordre de `teams`.

---

## 5. Mode organisateur

1. Va dans **Mon profil** → « Mode organisateur », tape `adminPin` et clique sur « Activer ». Un badge **ORGA** apparaît en haut à droite.
2. **Semaine en cours** (dans Mon profil) :
   - *Automatique* : selon la date de départ (par défaut).
   - *Pas encore commencé* : cache tous les défis.
   - *S1, S2…* : force la semaine en cours.
3. **Masquer / afficher une semaine** : onglet Défis → carte « Semaine visible/masquée ». Le bouton « Auto » rétablit le comportement automatique.
4. **Supprimer un post ou un commentaire** : bouton rouge dans le feed. Ça retire aussi les points, likes et commentaires du post.
5. Le bouton « Quitter » désactive le mode organisateur.

> Le PIN organisateur et les PIN des départements sont écrits en clair dans le code public : ils servent à éviter les erreurs, pas à sécuriser contre quelqu'un de malveillant.

---

## 6. Firebase

### Règles de la base (Realtime Database → onglet « Règles »)

```json
{
  "rules": {
    "posts":    { ".read": true, ".write": true },
    "settings": { ".read": true, ".write": true },
    "likes":    { ".read": true, ".write": true },
    "comments": { ".read": true, ".write": true }
  }
}
```

À publier avec le bouton **Publier**. À la fin de la compétition, passe `".write"` à `false` pour figer le classement.

### Données stockées

| Dossier | Contenu |
|---|---|
| `posts` | Les défis déclarés (prénom, département, défi, description, lien, date) |
| `likes` | Les ♥ par post |
| `comments` | Les commentaires par post |
| `settings` | La semaine en cours et les semaines masquées/affichées |

### Remettre à zéro avant le lancement

Dans la console Firebase → Realtime Database → Données, supprime les dossiers `posts`, `likes` et `comments` (croix à côté du nom). Garde `settings` sauf si tu veux aussi remettre les réglages de semaines à zéro.

---

## 7. En cas de problème

| Symptôme | Piste |
|---|---|
| Le site affiche « Mode démo » | `firebase` est à `null`, ou le déploiement n'est pas terminé (onglet Actions). |
| Les posts ne s'enregistrent pas | Les règles ne sont pas publiées, ou elles ne contiennent pas tous les dossiers. |
| La modification ne s'affiche pas | Rechargement forcé (Ctrl+Maj+R) : le navigateur garde l'ancienne version en cache. |
| Un défi n'apparaît pas | Son ODD n'est pas dans `weekOfOdd` (et il n'a pas de `w`), ou sa semaine n'est pas encore ouverte. |
| « Défi supprimé » sur un post | L'identifiant du défi a été supprimé ou modifié dans `challenges`. |
| Une erreur après modification | Une virgule ou un guillemet manque dans `CONFIG`. Ouvre la console du navigateur (F12 → Console) pour voir la ligne. |
