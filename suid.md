# SUID / SGID — Élévation de privilèges

## Définition

Le bit SUID (Set User ID) sur un exécutable permet de l'exécuter avec les droits
du propriétaire du fichier (souvent root), quel que soit l'utilisateur qui le lance.
Si un binaire SUID offre des fonctionnalités permettant d'exécuter du code
ou de lire des fichiers arbitraires, il peut être détourné pour élever les privilèges.

## Énumération

```bash
# Trouver tous les binaires SUID appartenant à root
find / -user root -perm -4000 -type f 2>/dev/null
```

## Binaires SUID courants et risques associés

| Binaire | Risque | Action recommandée |
|---|---|---|
| `bash` avec SUID | Shell root direct — critique | Retirer immédiatement |
| `find` avec SUID | Exécution de commandes via `-exec` | Retirer le bit SUID |
| `python` / `perl` | Exécution de scripts en root | Retirer le bit SUID |
| `cp` / `mv` | Écrasement de fichiers système | Retirer le bit SUID |
| `nano` / `vim` | Édition de fichiers système | Retirer le bit SUID |

## CVE-2021-3156 — Baron Samedit (sudo)

Vulnérabilité dans `sudo` (versions < 1.9.5p2) permettant à un utilisateur local
sans droits sudo d'obtenir un shell root via un buffer overflow dans `sudoedit`.

| Élément | Détail |
|---|---|
| CVE | CVE-2021-3156 |
| Versions affectées | sudo < 1.9.5p2 |
| Impact | Élévation de privilèges locale vers root |
| Remédiation | `apt upgrade sudo` ou `yum update sudo` |

## Référence GTFOBins

GTFOBins répertorie les binaires Unix exploitables pour l'élévation de privilèges
lorsqu'ils sont mal configurés (SUID, sudo, capabilities).

→ https://gtfobins.github.io/

## Remédiations

- Auditer régulièrement les binaires SUID : `find / -perm -u=s -type f 2>/dev/null`
- Retirer les bits SUID non justifiés : `chmod u-s /chemin/binaire`
- Maintenir les packages à jour (CVE-2021-3156 et autres)
- Aucun binaire ne devrait être SUID sans raison documentée et nécessité avérée

## Ce que j'ai appris

- Comment identifier rapidement les binaires SUID dangereux
- L'impact d'un binaire SUID mal configuré sur la sécurité du système
- L'importance de maintenir sudo à jour (CVE-2021-3156)
- GTFOBins comme référence systématique lors d'un audit Linux
