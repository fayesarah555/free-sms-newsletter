# 📱 SMS Notifier — InfiniReach
> Script Python pour envoyer des SMS en masse via ton téléphone Android avec gestion automatique des STOP.

---

## 📁 Structure du projet

```
sms/
├── envoyer_sms.py       ← Script principal
├── contacts.csv         ← Ta liste de contacts
├── stop_list.txt        ← Numéros ayant dit STOP (généré automatiquement)
├── log_YYYYMMDD_HHMMSS.txt  ← Fichier de log (généré à chaque envoi)
└── README.md            ← Ce fichier
```

---

## ⚙️ Prérequis

### 1. Python 3.8 ou supérieur
Vérifie ta version dans le terminal :
```bash
python --version
```
Si Python n'est pas installé → https://www.python.org/downloads/

### 2. La librairie `requests`
```bash
pip install requests
```

### 3. Un compte InfiniReach actif
- Site : https://app.infinireach.io
- L'app Android **SMS Gateway by InfiniReach** installée sur ton téléphone
- Ton téléphone lié à ton compte (via QR code)

---

## 🔧 Configuration

Ouvre `envoyer_sms.py` et remplis uniquement la section `CONFIGURATION` en haut du fichier :

```python
# Ta clé API (disponible dans InfiniReach → Integrations)
API_KEY = "smsrelay_XXXX..."

# Ton numéro de téléphone Android enregistré dans InfiniReach
# Format international obligatoire : +33 suivi des 9 chiffres (sans le 0)
# Exemple : 06 12 34 56 78 → +33612345678
MON_NUMERO = "+33651753658"

# Ton message (160 caractères max pour un SMS simple)
MESSAGE = "Bonjour, nous avons changé de plateforme de réservation. Retrouvez-nous ici : [LIEN]. Pour ne plus recevoir nos messages, répondez STOP."
```

> ⚠️ **Ne partage jamais ta clé API.** Ne la mets pas sur GitHub ou dans un email.
> Si tu penses qu'elle a été compromise, régénère-la dans InfiniReach → Integrations.

---

## 📋 Préparer le fichier contacts.csv

Le fichier doit avoir **une colonne obligatoire** nommée `numero`.

### Format attendu :
```
numero
0612345678
0698765432
+33712345678
06 23 45 67 89
```

Les formats suivants sont tous acceptés et convertis automatiquement :
| Format saisi | Converti en |
|---|---|
| `0612345678` | `+33612345678` |
| `06 12 34 56 78` | `+33612345678` |
| `06-12-34-56-78` | `+33612345678` |
| `+33612345678` | `+33612345678` (inchangé) |

### Depuis Excel :
1. Ouvre ton fichier Excel
2. Assure-toi que la colonne des numéros s'appelle **`numero`**
3. **Fichier → Enregistrer sous → CSV UTF-8**
4. Place le fichier dans le même dossier que `envoyer_sms.py`

---

## 🚀 Lancer le script

### Étape 1 — Ouvre un terminal dans le dossier du projet
Sous Windows : clic droit dans le dossier → "Ouvrir dans le terminal"
```bash
cd C:\Users\sarah.faye\Downloads\sms
```

### Étape 2 — Lance le script
```bash
python envoyer_sms.py
```

### Étape 3 — Suis les étapes affichées

Le script va :
1. **Vérifier les STOP** reçus depuis le dernier envoi via l'API
2. **Afficher un résumé** (nombre de contacts, exclusions, durée estimée)
3. **Te demander confirmation** avant d'envoyer quoi que ce soit
4. **Envoyer les SMS** un par un avec un délai d'1 seconde entre chaque
5. **Générer un log** avec le résultat de chaque envoi

### Exemple d'affichage :
```
=======================================================
  📱  ENVOI SMS — InfiniReach
=======================================================

🔍 Vérification des STOP reçus via l'API...
  ✅ Aucun nouveau STOP.

🛑 STOP list       : 3 numéro(s)
👥 Contacts total  : 1000
📤 À envoyer       : 997 (dont 3 ignoré(s))
⏱️  Durée estimée   : ~16 min 37 sec

💬 Message (136 car.) :
   Bonjour, nous avons changé de plateforme...

▶️  Lancer l'envoi ? (oui/non) : oui

--- Début de l'envoi ---

[1/997] +33612345678... ✅
[2/997] +33698765432... ✅
[3/997] +33712345678... ❌
...
```

---

## 🛑 Système STOP

### Comment ça fonctionne

Quand un contact répond **STOP** à ton SMS :
1. InfiniReach reçoit le message entrant sur ton téléphone
2. À chaque prochain lancement du script, il consulte l'API pour détecter les nouveaux STOP
3. Le numéro est automatiquement ajouté à `stop_list.txt`
4. Ce numéro est exclu de tous les envois suivants

### Le fichier stop_list.txt

Créé automatiquement, il contient un numéro par ligne :
```
+33612345678
+33698765432
```

Tu peux aussi **ajouter manuellement** un numéro dans ce fichier si quelqu'un te demande d'arrêter par un autre canal (appel, email...).

### Ajouter un STOP manuellement
Ouvre `stop_list.txt` avec un éditeur de texte et ajoute le numéro :
```
+33612345678
+33698765432
+33612345999   ← ajouté manuellement
```

---

## 📄 Lire les logs

À chaque lancement, un fichier `log_YYYYMMDD_HHMMSS.txt` est créé.

### Exemple de contenu :
```
[22:16:42] Début — 997 SMS
[22:16:42] Message : Bonjour, nous avons changé...
[22:16:43] OK — +33612345678
[22:16:44] OK — +33698765432
[22:16:45] ECHEC — +33712345678
...
[22:33:19] Fin — 994 succès, 3 échecs
```

Les lignes `ECHEC` peuvent être dues à des numéros invalides, éteints ou hors réseau.

---

## ⚡ Performances

| Paramètre | Valeur |
|---|---|
| Délai entre SMS | 1 seconde |
| Vitesse d'envoi | ~60 SMS/minute |
| Limite API | 100 requêtes/minute |
| Temps pour 1000 SMS | ~17 minutes |

> ⚠️ **Ne descends pas en dessous de 1 seconde** de délai entre les SMS,
> tu risques d'atteindre la limite de l'API (100 req/min) et d'avoir des erreurs.

---

## ❌ Erreurs fréquentes et solutions

### `Timeout`
**Cause :** L'API met trop de temps à répondre (envoi d'un vrai SMS)
**Solution :** Vérifie que `MON_NUMERO` correspond exactement au numéro enregistré dans InfiniReach. Assure-toi que l'app Android est ouverte et connectée.

### `401 Unauthorized`
**Cause :** Clé API incorrecte ou expirée
**Solution :** Regénère ta clé dans InfiniReach → Integrations et mets-la à jour dans le script

### `ModuleNotFoundError: No module named 'requests'`
**Cause :** La librairie n'est pas installée
**Solution :**
```bash
pip install requests
```

### `FileNotFoundError: contacts.csv`
**Cause :** Le fichier contacts.csv n'est pas dans le même dossier que le script
**Solution :** Place `contacts.csv` dans `C:\Users\sarah.faye\Downloads\sms\`

### `KeyError: 'numero'`
**Cause :** La colonne de ton CSV ne s'appelle pas exactement `numero`
**Solution :** Ouvre le CSV et vérifie que la première ligne est bien `numero` (minuscule, sans accent, sans espace)

---

## 🔒 Bonnes pratiques légales (France)

En France, l'envoi de SMS commerciaux/informationnels est encadré par le **RGPD** et les règles **ARCEP** :

- ✅ Tu dois mentionner un moyen de se désinscrire (ex: "répondez STOP")
- ✅ Tu ne peux contacter que des personnes ayant donné leur consentement
- ✅ Le STOP doit être pris en compte immédiatement
- ✅ Tu ne peux pas envoyer entre 20h et 8h, ni le dimanche et les jours fériés
- ❌ Interdiction d'envoyer à des numéros achetés ou récupérés sans consentement

---

## 📞 Support InfiniReach

- Documentation API : https://infinireach.io/docs/api
- Dashboard : https://app.infinireach.io
- Support : support@infinireach.io
