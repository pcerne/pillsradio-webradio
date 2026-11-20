# pillsradio — site webradio

Site statique dédié à la webradio **PillsRadio**.
Il remplace la page AzuraCast par défaut (`radio.pillsradio.fr/public/pillsradio`) par quelque chose de plus soigné : player en direct, programme, émissions, histoire (papy tango).

## Prérequis — hébergement (lire avant de déployer)

> Source de vérité : `~/perso/migration` (voir `docs/audit-2026-11-18.md` et `docs/url-scheme.md`).

L'infra pillsradio est **auto-hébergée**, pas sur un PaaS :

- Serveur en cours de migration **Scaleway Dedibox `51.15.5.136` → OVH `217.182.192.17`** (à finir avant fin nov. 2026).
- Reverse proxy **nginx** (Dedibox) puis **Caddy** (OVH, config-as-truth repo `pillsbox-stack`), TLS Let's Encrypt automatique. Les sites statiques sont servis depuis `/var/www/<site>`.
- **Déploiement = rsync over SSH** vers `pills@pillsradio.fr`, suivant la convention `~/perso/debian-vps-config/sites/pillsradio-accueil/deploy.sh` (`rsync -av --delete ./ pills@…:/var/www/<cible>/`). Pas de sudo.
- **Cible de ce site** : l'**apex `pillsradio.fr` = la webradio** (décision du 18/11, `docs/url-scheme.md`) ; `radio.pillsradio.fr` → 301 vers l'apex ; l'ancienne page d'accueil part sur `home.pillsradio.fr`.

Conséquence : **GitHub Pages n'est pas la cible** (c'est juste un aperçu temporaire). Le déploiement réel se fait sur le serveur (Caddy), comme les autres sites. Servi sous `pillsradio.fr`, le site devient **same-origin** avec AzuraCast, ce qui permet un **proxy serveur** pour cacher `AZURACAST_API_KEY` (fonction « c'est quoi ce titre »).

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

## Mise en ligne

Dépôt source (versionnement) : https://github.com/pcerne/pillsradio-webradio
Aperçu temporaire GitHub Pages : https://pcerne.github.io/pillsradio-webradio/ (à retirer une fois le déploiement serveur en place).

**Cible réelle : le serveur (Caddy sur OVH), apex `pillsradio.fr`.** Deux options :

1. Convention du projet (comme `pillsradio-accueil`) : un `deploy.sh` qui fait
   `rsync -av --delete ./ pills@pillsradio.fr:/var/www/webradio/`, le vhost Caddy servant ce dossier sur l'apex. Déploiement manuel ou via un hook.
2. CI vers le serveur : adapter `.github/workflows/deploy.yml` pour pousser par `rsync`/`scp` sur le serveur (clé de déploiement en secret GitHub) au lieu de Pages.

À faire au moment du branchement :
- poser le vhost Caddy de l'apex vers le dossier du site (dans `pillsbox-stack`), avec `radio.` → 301 vers l'apex ;
- ajouter un **proxy serveur** pour l'historique AzuraCast (garde `AZURACAST_API_KEY` côté serveur, le JS appelle une route locale type `/api/history`) ;
- le DNS de l'apex pointe déjà (ou pointera) sur l'IP OVH `217.182.192.17` dans le cadre de la migration — pas de config DNS GitHub.

## Fichiers

| Fichier | Rôle |
| --- | --- |
| `index.html` | la page et sa logique (player, détection appareil, now-playing) |
| `styles.css` | styles de la page (thème nuit, en-tête de provenance Hallmark) |
| `tokens.css` | jetons de couleur, type et espacement (thème Ultraviolet) |
| `base.css` | règles de base partagées (réinitialisation, accessibilité, traitement photo) |

Design : thème Hallmark **Ultraviolet** en colourway nuit.
