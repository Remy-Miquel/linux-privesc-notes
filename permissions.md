# Permissions Linux — Fichiers sensibles

## Rappel permissions Linux

```
rwxrwxrwx
 |||
 ||└── autres (world)
 |└─── groupe
 └──── propriétaire (user)

r = 4  |  w = 2  |  x = 1
Exemple : 755 = rwxr-xr-x
```

## Fichiers critiques à vérifier

```bash
# Permissions des fichiers système sensibles
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/crontab

# Fichiers world-writable (modifiables par tous)
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# Clés SSH accessibles
find / \( -name "*.pem" -o -name "id_rsa" -o -name "id_ed25519" \) 2>/dev/null

# Fichiers de configuration avec secrets potentiels
find / \( -name "*.conf" -o -name "*.cfg" -o -name ".env" \) 2>/dev/null
find / -name "wp-config.php" 2>/dev/null
```

## Cas dangereux courants

| Fichier | Permission dangereuse | Risque |
|---|---|---|
| `/etc/passwd` | World-writable | Ajout d'un compte root sans mot de passe |
| `/etc/shadow` | World-readable | Récupération de hashs → crack hors ligne |
| `/home/user/.ssh/` | Accessible en lecture | Exfiltration de clé privée SSH |
| Scripts cron | World-writable | Injection de commandes exécutées en root |
| Fichiers de config | World-readable | Exposition de secrets, mots de passe, tokens |

## Permissions recommandées

| Fichier / Dossier | Permission correcte |
|---|---|
| `/etc/shadow` | `640` (root:shadow) |
| Clés privées SSH | `600` (propriétaire uniquement) |
| Scripts cron | `750` minimum, propriété root |
| Fichiers de config sensibles | `600` ou `640` |
| Répertoires home | `700` |

## Remédiations

- Vérifier régulièrement les permissions des fichiers sensibles
- Corriger avec `chmod` et `chown` les fichiers mal configurés
- Auditer les fichiers world-writable après chaque déploiement
- Principe du moindre privilège sur tous les fichiers de configuration
- Supprimer les clés SSH non utilisées et les comptes inactifs

## Ce que j'ai appris

- Dans le lab EvilCorp, l'audit des permissions a permis d'identifier des fichiers world-writable utilisés comme vecteur par d'autres membres de l'équipe — la combinaison avec un cron root a suffi à escalader
- Les fichiers critiques à vérifier systématiquement lors d'un audit de configuration
- Comment les permissions trop larges créent des vecteurs d'escalade simples même sans exploit
- L'importance d'automatiser cet audit (`find / -writable`) en première phase de post-accès
