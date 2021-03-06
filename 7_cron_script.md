# Step 7 : Cron script

## monitoring.sh
```
L’architecture de votre système d’exploitation ainsi que sa version de kernel.
• Le nombre de processeurs physiques.
• Le nombre de processeurs virtuels.
• La mémoire vive disponible actuel sur votre serveur ainsi que son taux d’utilisation
sous forme de pourcentage.
• La mémoire disponible actuel sur votre serveur ainsi que son taux d’utilisation
sous forme de pourcentage.
• Le taux d’utilisation actuel de vos processeurs sous forme de pourcentage.
• La date et l’heure du dernier redémarrage.
• Si LVM est actif ou pas.
• Le nombre de connexions actives.
• Le nombre d’utilisateurs utilisant le serveur.
• L’adresse IPv4 de votre serveur, ainsi que son adresse MAC (Media Access Control).
• Le nombre de commande executées avec le programme sudo.
Dès le lancement de votre serveur, le script écrira des informations toutes les 10 minutes 
sur tous les terminaux (jetez un œil du côté de wall). La bannière est facultative.
```

1. Créer le fichier monitoring.sh dans dossier root.
` $ vim monitoring.sh `

2. Développer le script en bash
` #!/bin/bash `

```
$ wall [-n] [-t TIMEOUT] [file]
$ echo "test message" | wall
```

-n : without the header (banniere)

___

## My script

```
#!/bin/bash
archi=$( uname -a )
pcpu=$( lscpu | awk '/^CPU.s.:/ {print $NF}' )
vcpu=$( lscpu | awk '/^Core.s. per socket:/ {cores=$NF} /^Socket.s.:/ {sockets=$NF} END {print cores * sockets}' )
ramfree=$( free -m | awk '/^Mem:/ {printf("%.0f"), $4}' )
ramtotal=$( free -m | awk '/^Mem:/ {print $2}' )
ramused=$( free -m | awk '/^Mem:/ {printf("%.2f"), $3*100/$2}' )
romfree=$( df -h --total | awk '/^total/ {printf("%.1f"), $3}' )
romtotal=$( df -h --total | awk '/^total/ {printf("%d"), $2}' )
romused=$( df -h --total | awk '/^total/ {printf("%d"), $5}' )
lcpu=$( mpstat | grep all | awk '{printf("%.2f"), 100-$13}' )
boot=$( who -b | awk '{print $3, $4}' )
lvm=$( if [ $( lsblk | grep 'lvm' | wc -l ) -eq 0 ]; then echo "no"; else echo "yes"; fi )
tcp=$( ss -s | awk '/^TCP:/ {printf("%d"), $4}' )
user=$( users | wc -w )
ip=$( hostname -I )
#mac=$( ifconfig | grep ether | head -1 | awk '{print $2}' )
mac=$( ip a | grep link/ether | awk '{print $2}' )
sudo=$( cat /var/log/sudo/commandslog | grep COMMAND | wc -l )

wall -n "
#Architecture : $archi
#CPU physical : $pcpu
#vCPU : $vcpu
#Memory usage : $ramfree/"$ramtotal"MB free  ( $ramused% used )
#Disk usage : $romfree/"$romtotal"Gb free ( $romused% used )
#CPU load : $lcpu%
#Last boot : $boot
#LVM use : $lvm
#Connections TCP : $tcp ESTABLISHED
#Users log : $user
#Network : IP $ip  ( $mac )
#sudo : $sudo cmd
"
```

___


### Command descriptions

```
#### shebang : en-tête d'un fichier texte qui indique au système d'exploitation (de type Unix) 
que ce fichier n'est pas un fichier binaire mais un script (ensemble de commandes) ; 
sur la même ligne est précisé l'interpréteur permettant d'exécuter ce script.
#!/bin/bash

NOMVARIABLE=$( INSTRUCTIONS )
uname -a : print all system information, in the following order
archi=$( uname -a )

lscpu: display information about the CPU architecture
awk '/^CPU.s.:/ {print $NF}':
print: [printf vs print](https://www.gnu.org/software/gawk/manual/html_node/Basic-Printf.html)
pcpu=$( lscpu | awk '/^CPU.s.:/ {print $NF}' )


vcpu=$( lscpu | awk '/^Core.s. per socket:/ {cores=$NF} /^Socket.s.:/ {sockets=$NF} END {print cores * sockets}' )
ramfree=$( free -m | awk '/^Mem:/ {printf("%.0f"), $4}' )
# arrondir
ramtotal=$( free -m | awk '/^Mem:/ {print $2}' )
ramused=$( free -m | awk '/^Mem:/ {printf("%.2f"), $3*100/$2}' )
# Two numbers after comma
romfree=$( df -h --total | awk '/^total/ {printf("%.1f"), $3}' )
romtotal=$( df -h --total | awk '/^total/ {printf("%d"), $2}' )
romused=$( df -h --total | awk '/^total/ {printf("%d"), $5}' )
lcpu=$( mpstat | grep all | awk '{printf("%.2f"), 100-$13}' )
boot=$( who -b | awk '{print $3, $4}' )
lvm=$( if [ $( lsblk | grep 'lvm' | wc -l ) -eq 0 ]; then echo "no"; else echo "yes"; fi )
# exit is for if there is more than one line, it prints just one time instead of for each line.
tcp=$( ss -s | awk '/^TCP:/ {printf("%d"), $4}' )
user=$( users | wc -w )
ip=$( hostname -I )
#mac=$( ifconfig | grep ether | head -1 | awk '{print $2}' )
mac=$( ip a | grep link/ether | awk '{print $2}' )
sudo=$( cat /var/log/sudo/commandslog | grep COMMAND | wc -l )

#### wall: write a message to all users -n: suppress the banner
wall -n "
#Architecture : $archi
#CPU physical : $pcpu
#vCPU : $vcpu
#Memory usage : $ramfree/"$ramtotal"MB free  ( $ramused% used )
#Disk usage : $romfree/"$romtotal"Gb free ( $romused% used )
#CPU load : $lcpu%
#Last boot : $boot
#LVM use : $lvm
#Connections TCP : $tcp ESTABLISHED
#Users log : $user
#Network : IP $ip  ( $mac )
#sudo : $sudo cmd
"
```

___


## What to install
```
Install sysstat for mpstat
$sudo apt install sysstat
```

## Droit sur fichier monitoring.sh
    
chmod 755 monitoring.sh

## Setting Up a cron Job
Configure cron as root
` $ sudo crontab -u root -e `

option 2
To schedule a shell script to run every 10 minutes, replace below line
` 23 # m h  dom mon dow   command `
with:
`23 */10 * * * * sh /root/monitoring.sh `
or (for more tests)
```
\# m h  dom mon dow   command
#*/1 * * * * /root/monitoring.sh
#* * * * * sleep 30; /root/monitoring.sh
*/10 * * * * /root/monitoring.sh
```

Check root's scheduled cron jobs 
` $ sudo crontab -u root -l `
Choose second option

## Interrompre l’exécution du script sans le modifier
```
systemctl stop cron
systemctl start cron
```


Rédiger des [scripts](https://debian-facile.org/doc:programmation:shells:debuter-avec-les-scripts-shell-bash "debian-facile.org") sous bash
Introduction aux [scripts](https://openclassrooms.com/fr/courses/43538-reprenez-le-controle-a-laide-de-linux/42867-introduction-aux-scripts-shell "openclassroom.com") shell
