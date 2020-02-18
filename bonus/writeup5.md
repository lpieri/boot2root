# Writeup numéro 5:
## Comment devenir root de la base de données

Une fois connectés en ssh à l'user laurie, on peut afficher les informations de la création de la base de données avec le path que l'on peut trouver sur la documentation de Mylittleforum.

```
~$ cat /var/www/forum/config/db_settings.php
<?php
$db_settings['host'] = 'localhost';
$db_settings['user'] = 'root';
$db_settings['password'] = 'Fg-\'kKXBj87E:aJ$';
$db_settings['database'] = 'forum_db';
$db_settings['settings_table'] = 'mlf2_settings';
$db_settings['forum_table'] = 'mlf2_entries';
$db_settings['category_table'] = 'mlf2_categories';
$db_settings['userdata_table'] = 'mlf2_userdata';
$db_settings['smilies_table'] = 'mlf2_smilies';
$db_settings['pages_table'] = 'mlf2_pages';
$db_settings['banlists_table'] = 'mlf2_banlists';
$db_settings['useronline_table'] = 'mlf2_useronline';
$db_settings['login_control_table'] = 'mlf2_logincontrol';
$db_settings['entry_cache_table'] = 'mlf2_entries_cache';
$db_settings['userdata_cache_table'] = 'mlf2_userdata_cache';
?>
```

On voit que l'user de la base de données est `root` et le mot de passe `Fg-'kKXBj87E:aJ$`.

Nous pouvons nous connecter directement sur `/phpmyadmin` avec ces identifiant pour avoir l'accès root de la base de données.

## Comment devenir admin du forum

Après avoir obtenu les identifiants root pour la base de données, nous pouvons nous connecter et avoir tous le contrôle.

Nous pouvons modifier le `user_type` de `lmezard` dans la table `mlf2_userdata` et le mettre a `2` pour avoir accès admin sur le forum.

![phpmyadmin_userdata](/assets/phpmyadmin_userdata.png)

Si on se connecte au forum avec les identifiants de l'user `lmezard`, on aura maintenant les droits admin et on pourra faire des actions comme gérer les utilisateurs ou même supprimer le forum.

![admin_forum](/assets/admin_forum.png)

## Récupérer le filesysteme

Monter le fichier iso avec `mount BornToSecHackMe-v1.1.iso iso/ -o loop` ou (sur mac) clique droit -> ouvrir avec -> DiskImageMounter.  
Récupérer le fichier du filesysteme compressé `casper/filesystem.squashfs`. 
Décompresser le fichier avec `unsquashfs filesystem.squashfs`. 
Un dossier `squashfs-root`.  
On peut acceder à l'ensemble des fichiers présents sur l'ISO.  
