# Mauvaise configuration sudo

## Définition

Le fichier `/etc/sudoers` définit les commandes qu'un utilisateur peut exécuter
avec les droits d'un autre utilisateur (généralement root).
Une mauvaise configuration permet d'exécuter du code arbitraire avec des droits élevés.

## Énumération

```bash
# Vérifier ses droits sudo
sudo -l
```

## Configurations dangereuses

| Configuration | Risque | Priorité |
|---|---|---|
| `user ALL=(ALL) NOPASSWD: ALL` | Shell root sans mot de passe | Critique |
| `user ALL=(ALL) /usr/bin/vim` | Édition de fichiers système, shell via `:!bash` | Élevé |
| `user ALL=(ALL) /usr/bin/find` | Exécution de commandes via `find -exec` | Élevé |
| `user ALL=(ALL) /usr/bin/python3` | Exécution de scripts Python en root | Élevé |
| `user ALL=(ALL) /usr/bin/less` | Lecture de fichiers root, shell via `!bash` | Moyen |

## Remédiations

- Auditer `/etc/sudoers` régulièrement (`sudo -l` pour chaque compte)
- N'autoriser que les commandes strictement nécessaires
- Éviter `NOPASSWD` sauf cas documenté et justifié
- Utiliser `visudo` pour éditer le fichier (validation syntaxique intégrée)
- Préférer les groupes sudo aux règles individuelles
- Vérifier les binaires accordés sur GTFOBins avant toute autorisation

## Ce que j'ai appris

- Comment `sudo -l` révèle immédiatement les vecteurs d'escalade disponibles
- Pourquoi des binaires courants (vim, find, python) sont dangereux avec NOPASSWD
- L'importance d'auditer sudoers après chaque changement de configuration système
