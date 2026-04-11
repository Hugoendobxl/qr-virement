# ALFRED.md — qr-virement

> Documentation vivante à jour au 2026-04-11. Ce fichier est lu par Alfred au démarrage de toute session Claude Code sur ce module. Il doit être mis à jour à chaque modification significative du code.

## 1. Rôle du module

PWA (Progressive Web App) très simple pour générer un **QR code de virement SEPA** au format EPC à utiliser au desk du cabinet. Les patients qui veulent payer par virement scannent le QR avec leur application bancaire et tous les champs (IBAN, montant, communication) sont pré-remplis. Fonctionne hors ligne (service worker + manifest dynamiques).

## 2. URLs et identifiants

- **Frontend (PWA)** : https://qr-virement.vercel.app
- **Backend** : aucun (PWA statique sans API)
- **Dépôt GitHub** : Hugoendobxl/qr-virement
- **Projet Vercel** : `qr-virement`
- **Projet Railway** : aucun

## 3. Stack technique

- **Runtime** : navigateur
- **Framework** : **aucun** — HTML + CSS + JS vanilla, fichier statique unique (`qr-virement-app.html`, ~672 lignes, ~24 KB)
- **Lib externe** : `qrcodejs` 1.0.0 (chargé depuis CDNJS)
- **Build** : aucun
- **PWA** : manifest + service worker générés dynamiquement en JavaScript et installés à la volée (Blob URL)
- **Hébergement** : Vercel (déploiement direct du fichier statique)
- **Authentification** : aucune (le module ne stocke ni ne transmet d'info personnelle)

## 4. Variables d'environnement

Aucune.

## 5. Structure du projet

```
qr-virement/
├── index.html              # Redirection HTML vers qr-virement-app.html (12 lignes)
└── qr-virement-app.html    # Tout le module (672 lignes, 24 KB)
```

⚠️ Pas de `package.json`, pas de `vercel.json`, pas de `.gitignore`.

L'`index.html` racine fait juste une redirection (`<meta http-equiv="refresh">` + `window.location.href`) vers `qr-virement-app.html`. C'est probablement un héritage historique — on pourrait simplement renommer `qr-virement-app.html` en `index.html` et supprimer le redirect.

## 6. Points d'entrée API

N/A (aucun backend).

## 7. Schéma de base de données

N/A.

## 8. Écrans / Pages

Une seule page :

- **Formulaire de saisie** :
  - Bénéficiaire (préfilé `Endodontie Louise SC`)
  - IBAN (préfilé `BE14068930419983`)
  - Montant (en EUR)
  - Communication (libre)
- **Affichage du QR code** (généré localement avec `qrcodejs`, format **EPC** SEPA)
- **Boutons** : télécharger l'image, partager (Web Share API)
- **Badge "Hors ligne"** affiché si pas de connexion

## 9. Règles métier critiques

- **IBAN par défaut** : `BE14068930419983` (compte du cabinet — ligne 417)
- **Bénéficiaire par défaut** : `Endodontie Louise SC` (ligne 416)
- **Format QR** : standard **EPC SEPA** (Quick Response Code Service Tag for SEPA Credit Transfer) — voir https://www.europeanpaymentscouncil.eu/document-library/guidance-documents/quick-response-code-guidelines-enable-data-capture-initiation
- **PWA installable** : manifest généré à la volée (`name: 'QR Virement — Endodontie Louise'`)
- **Service worker** : enregistré dynamiquement pour permettre l'utilisation hors ligne
- **Détection en ligne / hors ligne** : `window.addEventListener('online'/'offline')` met à jour un badge visuel

## 10. Intégrations externes

| Service | Rôle | Variables d'env |
|---|---|---|
| `cdnjs.cloudflare.com` | Chargement de la lib `qrcodejs` 1.0.0 | aucune |

Aucune autre intégration. Le module est **100% côté client** — aucune donnée ne transite vers un serveur.

## 11. Dépendances avec d'autres modules de l'écosystème

**Est appelé par** :
- `endodontie-launcher-v2` (tuile "QR Code SEPA" — ouverture dans nouvel onglet)

**Appelle** : aucun autre module (et aucun backend).

Module **complètement autonome**.

## 12. Déploiement

- **Déclenchement** : push sur `main` → auto-deploy Vercel
- **Pas de build** : Vercel sert le HTML statique
- **Branche de production** : `main`

## 13. Fichiers sensibles à ne pas modifier sans validation

- `qr-virement-app.html` — c'est tout le module
- L'IBAN et le bénéficiaire (lignes 416-417) — leur modification engagerait la responsabilité du cabinet sur les paiements

## 14. Commandes utiles

```bash
# Pas de commande de build
# Pour tester en local
open qr-virement-app.html
# ou
python3 -m http.server 8000
```

## 15. Historique des décisions importantes

- **PWA pour usage hors ligne** : décision intelligente pour ne jamais bloquer les paiements même si le wifi du cabinet est en panne.
- **Pas de framework, pas de backend** : volonté d'avoir un outil ultra-simple et réparable. La logique est minimale.
- **Lib `qrcodejs` chargée depuis CDNJS** : pas d'installation locale, pas de bundling. Si CDNJS tombe, le module peut être inutilisable la première fois (avant que le service worker n'ait caché la lib). À considérer.

## 16. TODO et points d'attention

- **Aucun commentaire `TODO/FIXME`** détecté.
- **`index.html` redondant** : pourrait être supprimé en renommant directement `qr-virement-app.html` → `index.html`. À faire en une seule passe.
- **Lib externe (CDNJS)** : risque mineur de cassure si CDNJS change ou tombe au premier chargement. Alternative : embarquer la lib en local.
- **Pas de `.gitignore`** : pas de risque puisque le repo n'a que des fichiers statiques.

---

**Dernière mise à jour** : 2026-04-11
**Mis à jour par** : Claude Code (session naissance du workspace, Alfred)
