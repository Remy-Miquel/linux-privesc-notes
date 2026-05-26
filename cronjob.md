# Tâches cron — Élévation de privilèges

## Définition

Les tâches cron s'exécutent automatiquement selon un calendrier, souvent
avec des droits élevés (root). Si un script exécuté par cron est modifiable
par un utilisateur non privilégié, ce dernier peut y injecter du code
qui s'exécutera avec les droits de root au prochain déclenchement.

## Énumération

```bash
# Crontabs système
cat /etc/crontab
ls -la /etc/cron.*

# Crontab de l'utilisateur courant
crontab -l

# Rechercher des scripts appelés par cron et vérifier leurs permissions
find / -name "*.sh" -type f 2>/dev/null
ls -la /chemin/vers/script.sh
```

## Vecteurs courants

| Scénario | Risque |
|---|---|
| Script cron inscriptible par l'utilisateur | Injection de commandes exécutées en root |
| Variable PATH exploitable dans `/etc/crontab` | Hijacking de chemin (PATH hijacking) |
| Wildcard dans une commande cron (`tar *`) | Injection via nom de fichier malveillant |

## Remédiations

- Auditer les permissions des scripts exécutés par cron (`ls -la`)
- Aucun script cron root ne doit être modifiable par un utilisateur non root
- Utiliser des chemins absolus dans toutes les commandes cron
- Éviter les wildcards dans les commandes cron exposées
- Limiter les tâches cron root au strict minimum nécessaire
- Vérifier régulièrement `/etc/crontab` et `/etc/cron.*`

## Ce que j'ai appris

- Comment repérer une tâche cron dont le script est modifiable par un utilisateur standard
- L'importance des permissions sur les scripts d'automatisation
- Pourquoi les chemins relatifs et les wildcards dans cron créent des vecteurs d'escalade
