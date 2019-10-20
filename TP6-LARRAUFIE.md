## Exercice 1. Disques et partitions

### 1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM
On entre dans configuration et on ajoute un nouveau disque de 5Go.

### 2. Vérifiez que ce nouveau disque dur est bien détecté par le système
On utilise la commande suivante pour verifier que le disque est bien détecté: **lsblk**.

### 3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83),et une seconde partition de 3 Go en NTFS (n°7)
Commande pour acceder au menu de partition:
```bash
fdisk /dev/sdb
```


Voici les etapes pour partitionner notre disque:
- Premier disque
```bash
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

```

- deuxieme disque
```bash
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2):
First sector (4196352-10485759, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-10485759, default 10485759):

Created a new partition 2 of type 'Linux' and of size 3 GiB.

```
On change le type de la partition en **NTFS**
```bash
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 7

Changed type of partition 'Linux' to 'HPFS/NTFS/exFAT'.
```


### 4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers.A l’aide de la commande mkfs, formatez vos deux partitions ( pensez à consulter le manuel !)

On formate une partition grace à la commande suivante:
>mkfs.ext4 /dev/sdb1

- Partition 1
```bash
tom@serveur:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.44.6 (5-Mar-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 528efe18-fba0-45a1-9091-a0a76849f89e
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```
- partition 2
```bash
tom@serveur:~$ sudo mkfs.ntfs /dev/sdb2
[sudo] password for tom:
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
```


### 5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ?

On ne peut pas afficher le type de système de fichier car ce dernier n'est pas monté  sur les partitions.


### 6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

On entre dans le fichier /etc/fstab et on inscrit ceci :

```bash
/dev/sdb1 /data ext4 defaults 0 0
/dev/sdb2 /win ntfs defaults 0 0
```
On créer les repertoires suivants:
```bash
tom@serveur:~$ sudo mkdir /data /win
```


### 7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration
On met a jour les modifications appliquées
```bash
tom@serveur:~$ sudo mount -a
```

### 8. Montez votre clé USB dans la VM

### 9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox

## Exercice 2. Partitionnement LVM
Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler
les disques et les partitions.

### 1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab
On supprime les lignes correspondantes du fichier /etc/fstab.

### 2. Supprimez les deux partitions du disque, et créez une patition unique de type LVM 
> La création d’une partition LVM n’est pas indispensable, mais vivement recommandée quand on utilise LVM sur un disque entier. En effet, elle permet d’indiquer à d’autres OS ou logiciels de gestion de disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données sur
ce disque.

 Acces au menu des partitions: **sudo fdisk /dev/sdb** 
```bash
Command (m for help): d
Partition number (1,2, default 2): 1

Partition 1 has been deleted.
```

```bash

Command (m for help): d
Selected partition 2
Partition 2 has been deleted.
```

Création de la partition LVM:
```bash
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

Created a new partition 1 of type 'Linux' and of size 5 GiB.
```
changement du type de la partition:
```bash
Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): w
```

 Attention à ne pas supprimer la partition système !

### 3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay.
>Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant
les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.
```bash
tom@serveur:~$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
tom@serveur:~$ sudo pvdisplay /dev/sdb1
  "/dev/sdb1" is a new physical volume of "<5,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               <5,00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               IQGIWA-Wbah-SYqx-7VvM-HU7G-dC2n-kbwECf
```

### 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.

>Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice
du groupe de volume, en commençant par 00, puis 01...)

On créer le groupe de volume **vg00** avec la commande **vgcreate**
```bash
tom@serveur:~$ sudo vgcreate vg00 /dev/sdb1
  Volume group "vg00" successfully created
```
On verifie la création avec **vgdisplay**
```bash
tom@serveur:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               vg00
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5,00 GiB
  PE Size               4,00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0
  Free  PE / Size       1279 / <5,00 GiB
  VG UUID               LPIthk-n3Mu-sepf-Eg1d-4ppH-7D1j-f2dhsb
```

### 5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.
>On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par
exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l
60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour
utiliser la totalité de l’espace libre.

Création du volume logique
```bash
tom@serveur:~$ sudo lvcreate -l 100%FREE -n lvData vg00
  Logical volume "lvData" created.
```



### 6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.

>A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par
exemple agrandir une partition à l’aide d’un nouveau disque.
```bash
tom@serveur:/$ sudo fdisk /dev/mapper/vg00-lvData
tom@serveur:~$ sudo mkfs.ext4 /dev/mapper/vg00-lvData
```

On ouvre **/etc/fstab** avec la commande: **tom@serveur:~$ sudo nano /etc/fstab**
et on ajoute la ligne suivante:
```bash
/dev/mapper/vg00-lvData /data ext4 defaults 0 0
```
### 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque.
Création du second disque de 5Go:
```bash
sdc               8:32   0    5G  0 disk
```
Création de la partition LVM
```bash
tom@serveur:~$ sudo fdisk /dev/sdc
Created a new partition 1 of type 'Linux' and of size 5 GiB.
Changed type of partition 'Linux' to 'Linux LVM'.
```
Création du volume physique:
```bash
tom@serveur:~$ sudo pvcreate /dev/sdc1
```


### 8. Utilisez la commande vgextend <nom_vg> <nom_pv> pour ajouter le nouveau disque au groupe de volumes
Ajout du nouveau disque dans le groupe de volumes:
```bash
tom@serveur:~$ sudo vgextend vg00 /dev/sdc1
  Volume group "vg00" successfully extended
  ```
### 9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs.
>Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par
bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de
présenter les fonctionnalités de base.

On aggrandit le volume logique **lvData**
Puis on le redimensionne:
```bash
tom@serveur:~$ sudo lvextend -l 100%FREE /dev/vg00/lvData
  New size (1279 extents) matches existing size (1279 extents).
tom@serveur:~$ sudo resize2fs /dev/vg00/lvData
resize2fs 1.44.6 (5-Mar-2019)
The filesystem is already 1309696 (4k) blocks long.  Nothing to do!
```

## Exercice 3. Exécution de commandes en différé : at et cron
### 1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez
entre temps que la tâche est bien programmée.
### 2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par
exemple en vous aidant des logs, du manuel...)
### 3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple,
l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes.
### 4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure
### 5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18
heures les 1er et 15 du mois :
### 6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures
### 7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un
fichier de log situé dans votre dossier personnel
### 8. Videz votre crontab
