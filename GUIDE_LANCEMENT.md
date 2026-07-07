# DETAK LO KER — Guide de lancement complet

Ce dossier contient tout le site. Trois fichiers :

- `index.html` — la page principale (tout est dedans : design, carte, formulaire)
- `mentions.html` — les mentions légales
- `apps-script-invitations.gs` — le script pour envoyer les invitations depuis le Google Sheet

Le site se pilote entièrement depuis UN Google Sheet. Tu ne toucheras au code qu'une seule fois (étape 2.3), pour coller l'adresse de ton API.

---

## Le choix d'architecture (ta question Vercel/GitHub vs Google Sheet)

**Recommandation : tout Google Sheet + Netlify.** Voici pourquoi.

L'option Vercel + GitHub est faite pour des équipes qui modifient du code régulièrement : chaque changement passe par un dépôt Git, un commit, un déploiement. C'est puissant mais ça ajoute trois outils à apprendre, et ça ne résout pas la question des données — il faudrait quand même un Google Sheet ou une base derrière. Et "recevoir les infos via Excel" (le vrai Excel Microsoft) demanderait Power Automate ou des scripts Office, bien plus lourds à mettre en place que Google Sheets.

L'option tout Google Sheet te donne exactement ce que tu décris : les gens s'inscrivent sur le site → la ligne tombe dans le Sheet → tu cliques sur un menu dans le Sheet → les invitations partent par email avec la date et le lieu du ronkozé de leur commune. Zéro code au quotidien, gérable depuis ton téléphone, et tu peux exporter le Sheet en fichier Excel (.xlsx) à tout moment (Fichier → Télécharger → Microsoft Excel) si tu as besoin du format Excel pour quelqu'un.

Pour l'hébergement, Netlify et Vercel se valent pour un site statique comme celui-ci. Netlify a le glisser-déposer le plus simple : tu déposes le dossier, c'est en ligne. On part là-dessus. (Si un jour tu veux passer sur GitHub + Vercel pour versionner le code, la migration prend 15 minutes, rien n'est perdu.)

---

## Étape 1 — Le Google Sheet (le cerveau du site)

1.1. Va sur sheets.google.com et crée une feuille nommée **DETAK LO KER**.

1.2. Crée **3 onglets** (clic droit sur l'onglet en bas → renommer / +) nommés EXACTEMENT :
`inscriptions` · `ronkoze` · `config`

1.3. Remplis la ligne 1 de chaque onglet :

**inscriptions** (le formulaire écrit ici) :
`date | prenom | nom | telephone | email | commune | quartier | jours | moments | intention | invitation`

**ronkoze** (le site lit ici → la carte) :
`commune | statut | prochain | lieu | participants`

**config** :
`cle | valeur`
puis en ligne 2 : `ronkozes_organises` | `0`

1.4. Ajoute une première ligne de test dans `ronkoze`, par exemple :
`Saint-Louis | programme | Samedi 26 septembre · 14h | Salle des fêtes | 18`

Règles de l'onglet ronkoze :
- `statut` = `programme` → la commune devient VERTE (date confirmée)
- `statut` = `mobilisation` → la commune devient BLEUE (ça se prépare)
- ligne absente ou statut vide → la commune reste grise ("Bientôt chez toi")
- `participants` = le compteur "X personnes ont déjà pris la parole ici"

---

## Étape 2 — SheetDB (le pont entre le site et le Sheet)

2.1. Dans le Google Sheet : bouton **Partager** → "Tous les utilisateurs disposant du lien" → **Lecteur**. Copie le lien.

2.2. Va sur **sheetdb.io** → crée un compte → **Create API** → colle le lien du Sheet. SheetDB te donne une URL du type `https://sheetdb.io/api/v1/abc123xyz`. Copie-la.

2.3. **La seule modification de code, à vie** : ouvre `index.html` avec un éditeur de texte (Bloc-notes, TextEdit, VS Code), cherche `TON_ID_ICI` (Ctrl+F / Cmd+F, une seule occurrence) et remplace toute l'URL par la tienne. Enregistre.

2.4. Test local : double-clique sur `index.html`. La commune Saint-Louis doit être verte (donnée lue depuis TON Sheet). Remplis le formulaire → une ligne doit apparaître dans l'onglet `inscriptions`.

⚠️ Quota : le plan gratuit SheetDB = environ 500 requêtes/mois, et chaque visite du site consomme 2 lectures. Suffisant pour démarrer et tester. Si le site dépasse ~200 visites/mois, prends le plan payant SheetDB (quelques €/mois) — ou dis-le-moi et je bascule la lecture de la carte sur l'API Google native, gratuite.

---

## Étape 3 — Les invitations automatiques depuis le Sheet

3.1. Dans le Google Sheet : **Extensions → Apps Script**.

3.2. Efface le contenu, colle tout le fichier `apps-script-invitations.gs`, clique sur 💾.

3.3. Recharge le Google Sheet : un menu **🟢 DETAK LO KER** apparaît en haut.

3.4. Fonctionnement : quand tu cliques sur "Envoyer les invitations en attente", le script parcourt l'onglet `inscriptions`, envoie un email (depuis ton Gmail) à chaque personne dont la case `invitation` est vide, en y glissant automatiquement la date et le lieu du prochain ronkozé de SA commune (lus dans l'onglet `ronkoze`), puis note "Envoyée le …" pour ne jamais envoyer deux fois.

3.5. À la première utilisation, Google demande d'autoriser le script : accepte (il tourne sur ton compte, rien ne sort de chez toi). Limite Gmail : ~100 emails/jour en compte gratuit.

Le texte de l'email est modifiable dans le script (variable `corps`).

---

## Étape 4 — Mise en ligne sur Netlify

4.1. Va sur **app.netlify.com** → crée un compte (avec ton email).

4.2. **Add new site → Deploy manually** → glisse le dossier `detakloker` (celui qui contient `index.html` et `mentions.html`).

4.3. 30 secondes plus tard, le site est en ligne sur une adresse du type `quelquechose.netlify.app`. Teste tout : carte, formulaire, mentions légales.

4.4. Pour mettre à jour le site plus tard (rare — seulement si on change le design) : Deploys → glisse le nouveau dossier. Les données, elles, se changent dans le Sheet, jamais ici.

---

## Étape 5 — Le domaine detakloker.re

5.1. Le `.re` s'achète chez un registrar qui le propose : **OVH**, **Gandi** ou **Ionos** (vérifie chez Hostinger si tu veux tout regrouper, mais le .re n'y est pas toujours disponible). Compte ~10–15 €/an.

5.2. Une fois acheté, dans **Netlify** : Site configuration → **Domain management** → Add a domain → `detakloker.re`. Netlify t'affiche les enregistrements DNS à créer.

5.3. Chez ton registrar (OVH/Gandi), dans la zone DNS du domaine, crée ce que Netlify demande — typiquement :
- un enregistrement **A** pour `detakloker.re` → `75.2.60.5` (l'IP indiquée par Netlify)
- un **CNAME** pour `www` → `ton-site.netlify.app`

(L'alternative encore plus simple : déléguer les DNS à Netlify — Netlify te donne 4 serveurs de noms à coller chez le registrar, et il gère tout.)

5.4. Attends la propagation (de 10 minutes à quelques heures). Netlify active le **HTTPS automatiquement** (certificat gratuit).

---

## Étape 6 — Tests de bout en bout avant d'annoncer le site

- Ouvre detakloker.re sur ordinateur ET sur téléphone
- Clique sur 3 communes de la carte (une verte, une bleue, une grise) : les infos et le compteur doivent changer correctement
- Fais une vraie inscription test avec ton email → vérifie la ligne dans le Sheet
- Lance "Envoyer les invitations en attente" → vérifie que tu reçois l'email avec la bonne date
- Modifie une ligne de `ronkoze` (change une date) → recharge le site → la carte doit refléter le changement
- Vérifie l'email de contact du footer (actuellement info@mazynasyon.org — tes mentions légales disent infos@, confirme le bon)

---

## Au quotidien (zéro code)

- **Un nouveau ronkozé se prépare ?** Ajoute/modifie une ligne dans l'onglet `ronkoze`. La carte se met à jour toute seule.
- **Un ronkozé a eu lieu ?** Mets à jour `participants` et incrémente `ronkozes_organises` dans `config`.
- **Des inscrits ?** Ils tombent dans `inscriptions`. Menu 🟢 → invitations parties.
- **Besoin d'un fichier Excel ?** Fichier → Télécharger → Microsoft Excel (.xlsx).
- **RGPD :** les inscrits peuvent demander la suppression de leurs données (info@mazynasyon.org) → supprime simplement leur ligne dans le Sheet.

Bon lancement ! 🟢
