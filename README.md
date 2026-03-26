# 🏗 TRANOKO — Guide de déploiement complet

## 📦 Fichiers du projet

```
tranoko/
├── index.html           ← App BTP client (GitHub Pages)
├── tranoko_admin.html   ← Console admin (GitHub Pages)
├── .nojekyll            ← Désactive Jekyll
├── README.md            ← Ce fichier
└── gas/
    └── Code.gs          ← Google Apps Script (cerveau du système)
```

---

## 🧠 Architecture — Comment ça marche

```
┌──────────────────┐   POST verif_essai    ┌─────────────────────┐
│  index.html      │ ───────────────────▶  │  Google Apps Script │
│  (app client)    │ ◀─── {ok,start_ts} ── │  ↕                  │
│  GitHub Pages    │                       │  Google Sheet (BDD) │
└──────────────────┘                       │  ↕                  │
                                           │  ← gen_licence ─────│
┌──────────────────┐   POST gen_licence    │                     │
│ tranoko_admin    │ ───────────────────▶  │  ← login_admin ─────│
│ (console admin)  │ ◀── {ok,code}  ────── └─────────────────────┘
│  GitHub Pages    │
└──────────────────┘
```

**Flux client :**
1. Client ouvre `index.html` → s'inscrit (prénom + tel)
2. GAS enregistre dans Sheet "Essais", renvoie `start_ts` ancré serveur
3. Bandeau d'essai affiché (1j standard / 2j avec code promo)
4. À expiration → app bloquée en lecture seule
5. Client appelle l'admin → admin génère un code → client le saisit → GAS valide → app débloquée

**Flux admin :**
- Admin ouvre `tranoko_admin.html` → se connecte avec son PIN
- Voit Dashboard (installations, essais, expirés, licenciés, revenus)
- Génère des licences depuis l'onglet Licences
- Gère ses commerciaux (liens affiliés, commissions)

---

## 🚀 ÉTAPE 1 — Google Sheet

1. Aller sur [sheets.google.com](https://sheets.google.com) → Créer une feuille
2. Nommer les **5 onglets** exactement ainsi (clic droit sur l'onglet → Renommer) :
   - `Essais`
   - `Licences`
   - `Commerciaux`
   - `Promos`
   - `Log`
3. Copier l'**ID du Sheet** depuis l'URL :
   ```
   https://docs.google.com/spreadsheets/d/ >>>VOTRE_ID_ICI<<< /edit
   ```

---

## 🚀 ÉTAPE 2 — Google Apps Script

1. Aller sur [script.google.com](https://script.google.com) → **Nouveau projet**
2. Remplacer tout le contenu par le fichier `gas/Code.gs`
3. Modifier les 2 variables en haut du fichier :
   ```js
   var SHEET_ID  = 'VOTRE_ID_SHEET_ICI';   // ← l'ID copié à l'étape 1
   var ADMIN_PIN = '1234';                  // ← votre code PIN (changez-le !)
   ```
4. **Initialiser les en-têtes** : sélectionner la fonction `initSheet` → cliquer **Exécuter**
   (Autoriser les permissions si demandé)
5. **Déployer** :
   - Cliquer **Déployer** → **Nouvelle exécution web**
   - Exécuter en tant que : **Moi**
   - Qui a accès : **Tout le monde (anonyme)**
   - Cliquer **Déployer**
6. **Copier l'URL** de déploiement (format `https://script.google.com/macros/s/AKfy.../exec`)

---

## 🚀 ÉTAPE 3 — Configurer l'URL GAS dans les fichiers

### Dans `index.html`
Chercher (Ctrl+H) :
```
REPLACE_WITH_YOUR_GAS_URL
```
Remplacer par votre URL GAS.

### Dans `tranoko_admin.html`
L'URL se configure **depuis l'interface** : onglet ⚙️ Paramètres → champ "URL Apps Script" → sauvegarder.

---

## 🚀 ÉTAPE 4 — GitHub Pages

```bash
# Initialiser le dépôt
git init
git add .
git commit -m "🏗 Tranoko v1.0 — deploy"
git branch -M main
git remote add origin https://github.com/VOTRE_USER/tranoko.git
git push -u origin main
```

Puis dans GitHub :
- **Settings → Pages → Source : Deploy from branch**
- Branch : `main` / `/ (root)`
- Cliquer **Save**

Votre app sera disponible sur :
```
https://VOTRE_USER.github.io/tranoko/
https://VOTRE_USER.github.io/tranoko/tranoko_admin.html
```

> 💡 Remplacer aussi `GH_BASE` dans `tranoko_admin.html` par l'URL de votre app :
> ```js
> const GH_BASE = 'https://VOTRE_USER.github.io/tranoko';
> ```

---

## 🔑 Ajouter des licences dans le Sheet (mode manuel)

Vous pouvez aussi ajouter des licences directement dans le Sheet :

**Onglet Licences** — une ligne par licence :
| code | actif | tel_utilisateur | date_activation | note |
|------|-------|-----------------|-----------------|------|
| TNKX7M2P4R | oui | 034xxxxxxxx | 23/03/2026 | Rakoto |

**Onglet Promos** — codes promo pour vos commerciaux :
| code | jours_bonus | actif | note |
|------|-------------|-------|------|
| RAKOTO | 2 | oui | Commercial Rakoto |
| FIDY | 2 | oui | Commercial Fidy |

---

## 💰 Tarification configurée

| Type | Prix |
|------|------|
| Prix catalogue | 150 000 Ar |
| Prix lancement | 120 000 Ar |
| Avec code promo (-10%) | 108 000 Ar |
| Commission < 30 ventes | 15% = 18 000 Ar |
| Commission ≥ 30 ventes | 20% = 24 000 Ar |

---

## 🔐 Codes générés par le GAS

Format : `TNK` + 6 caractères → ex: `TNKX7M2P4R`

Le client saisit ce code dans l'overlay d'activation de l'app.

---

## 🛠 Mise à jour du déploiement

```bash
# Après modification d'un fichier :
git add .
git commit -m "fix: description du changement"
git push
```

GitHub Pages se met à jour automatiquement en ~1 minute.

---

*Tranoko© by Tantsa™ — Propriété intellectuelle de Mandimbimanana©*
