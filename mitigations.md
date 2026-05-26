# Synthèse — Durcissement Linux contre l'élévation de privilèges

---

## Checklist de durcissement

| Domaine | Action | Commande |
|---|---|---|
| SUID | Auditer les binaires SUID | `find / -perm -u=s -type f 2>/dev/null` |
| SUID | Retirer les bits SUID non nécessaires | `chmod u-s /chemin/binaire` |
| sudo | Auditer les droits sudo de chaque compte | `sudo -l` |
| sudo | Supprimer les `NOPASSWD` inutiles | `visudo` |
| Cron | Vérifier les permissions des scripts cron | `ls -la` sur chaque script |
| Cron | Utiliser des chemins absolus | Éditer `/etc/crontab` |
| Permissions | Auditer les fichiers world-writable | `find / -writable -type f` |
| Permissions | Corriger les fichiers sensibles mal configurés | `chmod` + `chown` |
| Mises à jour | Maintenir les packages à jour | `apt update && apt upgrade` |
| Principe | Moindre privilège sur tous les comptes | Révision régulière |

---

## Outils d'énumération utilisés en lab

| Outil | Usage |
|---|---|
| LinPEAS | Audit automatisé complet de configuration Linux |
| GTFOBins | Référence des binaires exploitables |
| `sudo -l` | Droits sudo de l'utilisateur courant |
| `find` | Audit de permissions et de binaires SUID |

---

## Références

- GTFOBins : https://gtfobins.github.io/
- HackTricks Linux PrivEsc : https://book.hacktricks.wiki/linux-hardening/privilege-escalation
- NIST SP 800-123 : Guide to General Server Security
- CIS Linux Benchmark : https://www.cisecurity.org/benchmark/distribution_independent_linux
