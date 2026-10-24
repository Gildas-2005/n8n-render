# Déployer n8n sur Render (Blueprint + Postgres gratuit)

Ce dossier contient le fichier `render.yaml` officiel de Render pour déployer
**n8n** (service web + base PostgreSQL) en une seule commande via un **Blueprint**.

> Source : https://render.com/docs/deploy-n8n

---

## Étape 1 — Mettre le fichier sur GitHub

Deux options, choisis la plus simple pour toi :

### Option A : utiliser le template officiel (le plus rapide)
1. Ouvre https://github.com/render-examples/n8n
2. Clique **Use this template → Create a new repository**
3. Nomme le repo (ex. `n8n-render`), laisse en **Public**, puis **Create repository**.

### Option B : pousser le fichier de ce dossier
Depuis ce dossier (`n8n-render/`), dans un terminal :
```bash
git init
git add render.yaml
git commit -m "Add n8n Render blueprint"
git branch -M main
# Créer un repo vide sur GitHub, puis :
git remote add origin https://github.com/<TON_USER>/n8n-render.git
git push -u origin main
```

---

## Étape 2 — Créer le Blueprint sur Render

1. Va sur https://dashboard.render.com (connecte-toi avec GitHub si demandé).
2. Clique **New → Blueprint**.
3. Si ce n'est pas déjà fait, **connecte ton compte GitHub** à Render (bouton *Connect*).
4. Choisis le repo créé à l'étape 1, puis **Connect**.
5. Renseigne un **Blueprint Name** (ex. `n8n`) et vérifie la **Branch** (`main`).
6. Clique **Deploy Blueprint**.

Render crée automatiquement :
- le service web `n8n-service` (image Docker `n8nio/n8n:latest`)
- la base `n8n-db` (PostgreSQL)
- toutes les variables d'environnement de connexion (générées).

> ⚠️ Chaque workspace n'a droit qu'à **une seule base Postgres gratuite**.
> Si Render refuse la création de la base, supprime ta base gratuite existante
> ou passe la base en plan payant dans `render.yaml`.

---

## Étape 3 — Accéder à n8n

Une fois le déploiement terminé, ouvre l'URL `https://<ton-service>.onrender.com`
affichée en haut de la page du service web. Tu arrives sur l'écran de création
de compte n8n (owner) — crée ton utilisateur/admin.

---

## Étape 4 — Configurer les webhooks (important)

Pour que les nœuds **Webhook** fonctionnent :
1. Page du service web → **Environment** (colonne de gauche).
2. **Edit → Add → New variable**.
3. Key = `WEBHOOK_URL`
   Value = `https://<ton-service>.onrender.com/`
4. **Save and Deploy**.

---

## Limites du plan gratuit (à connaître)

- Le **service web gratuit** se met en veille après 15 min d'inactivité
  (première requête = réveil lent).
- La **base Postgres gratuite expire après 30 jours** → passe-la en payant
  avant cette date pour ne pas perdre tes workflows.

### Passer en payant (recommandé pour la prod)
Dans `render.yaml`, modifie les champs `plan` puis push :
```yaml
services:
  - type: web
    plan: 1c-2g        # recommandé pour n8n
databases:
  - name: n8n-db
    plan: 0.1c-256mb   # plus petit plan payant, suffisant
    # diskSizeGB: 5    # optionnel, multiple de 5 Go
```

---

## Note sécurité

- La variable `N8N_ENCRYPTION_KEY` est **générée automatiquement** par Render.
  **Ne la modifie jamais après le premier déploiement**, sinon tu perds
  l'accès aux credentials chiffrés existants.
- Ne partage jamais ta clé API Render ni tes mots de passe dans un chat.
