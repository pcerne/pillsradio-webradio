# pillsradio — site webradio

Site statique dédié à la webradio **PillsRadio**, distinct du site vitrine `pillsradio.fr`.
Il remplace la page AzuraCast par défaut (`radio.pillsradio.fr/public/pillsradio`) par quelque chose de plus soigné : player en direct, programme, émissions, histoire (papy tango).

## Contenu

- **Hero** : plein cadre, le vrai fond de la station (`/static/uploads/pillsradio/background.<version>.jpg`) et un **player maison** centré, dans le thème nuit (pochette, émission, titre, artiste, bouton de lecture du flux `radio.mp3`, auditeurs en direct). Les infos viennent de l'API `api/nowplaying/pillsradio` (CORS ouvert, rafraîchi toutes les 20 s). Section **« ce qui passe »** : les derniers titres (`song_history`).
- **« c'est quoi ce titre ? »** : saisie d'un jour + une heure, recherche dans l'historique AzuraCast le morceau joué à ce moment. Nécessite `AZURACAST_API_KEY` (voir plus bas).
- **Détection de l'appareil** : sur Android, la carte « application android » est mise en avant ; sur iPhone, c'est la « web app iphone » (celle créée dans l'autre fil). Sur ordinateur, les deux sont proposées. Sans JavaScript, les deux restent visibles.
- **Programme**, **émissions** (carrousel), **histoire** : contenu statique.

## Lancer en local

C'est un site statique, aucun build. Ouvrir `index.html`, ou servir le dossier :

```bash
python3 -m http.server 8080
# puis http://localhost:8080
```

## À renseigner avant mise en ligne

Le squelette est prêt ; il reste à brancher le vrai contenu.

1. **Liens et clés** — en haut du `<script>` dans `index.html` :
   - `APP_ANDROID_URL` : lien Play Store ou `.apk`.
   - `IOS_WEBAPP_URL` : URL de la web app iPhone (autre fil).
   - `AZURACAST_API_KEY` : clé API AzuraCast pour la section « c'est quoi ce titre ? » (l'endpoint d'historique exige une authentification). Tant qu'elle est vide, la recherche affiche un message et reste désactivée.
   Pour les liens d'app, tant qu'ils sont vides, les boutons pointent vers `#app` et sont marqués « lien à renseigner ».

   Attention sécurité : une clé API placée dans le JS d'une page statique est lisible par tous. Pour une mise en ligne publique, passer plutôt par un petit proxy côté serveur qui garde la clé. Pour un usage interne, la clé en clair peut dépanner.
2. **Programme** (`#programme`) : horaires et noms d'émissions sont des exemples à remplacer par la vraie grille.
3. **Émissions** (`#emissions`) : remplacer les vignettes dégradées par les vraies pochettes des émissions.
4. **Histoire** (`#histoire`) : le texte sur papy tango est une amorce neutre, à remplacer par la vraie histoire de la station. Remplacer aussi la photo d'illustration (actuellement une photo de bibliothèque, créditée en pied de page) par une vraie photo.

## Fichiers

| Fichier | Rôle |
| --- | --- |
| `index.html` | la page et sa logique (player, détection appareil, now-playing) |
| `styles.css` | styles de la page (thème nuit, en-tête de provenance Hallmark) |
| `tokens.css` | jetons de couleur, type et espacement (thème Ultraviolet) |
| `base.css` | règles de base partagées (réinitialisation, accessibilité, traitement photo) |

Design : thème Hallmark **Ultraviolet** en colourway nuit.
