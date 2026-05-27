# Linux Privilege Escalation Notes

Notes d'apprentissage sur l'élévation de privilèges Linux en environnement autorisé.  
Objectif : comprendre les erreurs de configuration courantes et les mesures de durcissement associées.

---

## Contexte

| Élément | Détail |
|---|---|
| Type | Notes de formation et d'entraînement |
| Environnement | Labs Linux autorisés |
| Périmètre | Autorisé et contrôlé |
| Référence terrain | Lab EvilCorp — Jedha Essential, mai 2026 (équipe de 4) |

---

## Sujets couverts

| Fiche | Sujet |
|---|---|
| `suid.md` | Binaires SUID/SGID — détection et risques |
| `sudo-rights.md` | Mauvaise configuration sudoers |
| `cronjob.md` | Tâches cron exploitables |
| `permissions.md` | Permissions Linux et fichiers sensibles |
| `mitigations.md` | Synthèse des mesures de durcissement |

---

## Compétences démontrées

- Énumération Linux (permissions, binaires SUID, cron, sudo)
- Identification d'erreurs de configuration système
- Compréhension des vecteurs d'élévation de privilèges
- Application du principe du moindre privilège
- Durcissement de configuration Linux

---

## Avertissement

Notes rédigées dans un cadre éducatif et légal.  
Toutes les techniques documentées ont été observées sur des labs autorisés.  
Notes issues du lab EvilCorp (Jedha Essential, mai 2026) et de plateformes d'entraînement.
