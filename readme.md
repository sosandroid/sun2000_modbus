
# Sun2000Modbus

Ce script Python connecte un onduleur Huawei Sun2000 via [ModbusTCP](https://fr.wikipedia.org/wiki/Modbus) pour lire les différents registres de l'onduleur et récupérer les données. 
Elles sont mise en forme et transmises à EmonCMS, Jeedom, PVOutput.org  et/ou BDPV.fr

L'onduleur Sun2000 de Huawei propose une interface ModbusTCP. Il faut l'activer depuis l'application FusionSolar pour qu'elle soit accessible par tous. 
Le fichier de configuration permet de l'adapter à votre installation.

Pour ceux qui souhaitent aller plus loin, la [doc Huawei](./doc/Huawei-Modbus) et [ModbusTool](https://github.com/ClassicDIY/ModbusTool) a bien servi pour vérifier la lecture correcte des données via le script Python.

__Attention, la partie Jeedom n'est pas testée__


## Prérequis
- Python 3.10+
- [pyModbus](https://pypi.org/project/pymodbus/)
- à minima, un des quatre comptes ci-dessous
    - _optionnel_ compte pvoutput.org
    - _optionnel_ compte Emoncms
    - _optionnel_ compte BDPV.fr
	- _optionnel_ compte Jeedom
- Interface Modbus activée sur l'onduleur

## Installation

### Python3
Python est proposé par défaut sur un serveur Ubuntu, veillez à le mettre à jour. Nous avons développé et testé avec une version 3.10
```
sudo apt update
sudo apt -y upgrade
```
Installez `pip`
````
sudo apt install -y python3-pip
````
### pyModbus
C'est la biblothèque qui permet de communiquer avec l'onduleur.
````
sudo pip install pymodbus
````
### Sun2000_modbus
Copiez le fichier `sun2000_modbus.py` et `sun2000-sample.conf` dans votre dossier `/home/<user>`

## Le fichier de configuration
La partie général permet de configurer le niveau de débug. A minima, pour commencer le niveau `debug = True` et `senddata = False` permet de s'assurer que les données sont correctement récupérées et mise en forme. Pour activer la transmission, passez `senddata = True`.

La partie __emoncms__ se personnalise avec:
- Activation ou non de cette partie
- L'adresse du serveur Emoncms (IP ou nom de domaine, http ou https)
- La clef d'API de votre compte
- Le nom du node sur lequel poster les données

La partie __jeedom__ se personnalise avec:
- Activation ou non de cette partie
- L'adresse du serveur Jeedom (IP ou nom de domaine, http ou https)
- La clef d'API de votre compte
- Le nom de la variable _Puissance Instantannée_
- Le nom de la variable _Energie du jour_
- Le nom de la variable _Energie depuis démarrage_
- Le nom de la variable _Température interne_

La partie __PVOutput.org__
- Activation ou non de cette partie
- votre clef d'API pour poster les données
- l'ID du site déclaré dans PVOutput
- Le nombre d'envoi par heure en fonction de la souscription ou non

La partie __BDPV__
- Activation ou non de cette partie
- Votre nom d'utilisateur
- Votre clef d'API
- L'heure à laquelle envoyer l'index une fois par jour - exprimé en secondes depuis minuit

Pour le sun2000, seule son adresse IP est nécessaire normalement. Si vous le désactivez (`enabled = False`) des données nulles seront envoyées.

Sauvegardez ce fichier sous le nom `sun2000.conf` à coté du script. 


````ini
[general]
#Manage debug level
modbusdebug = False
debug = True
senddata = False

[emoncms]
enabled = True
#set your EmonCMS address, api key, nodename, input labels
url = http://127.0.0.1/input/post
apikey = your-api-key
nodename = your-node-name

[jeedom]
enabled = False
url = http://127.0.0.1/core/api/jeeApi.php
apikey = your-api-key
instantpowername = your-variable-name
dailyenergyname = your-variable-name
alltimeenergyname = your-variable-name
internaltempname = your-variable-name

[pvoutput]
enabled = True
url = https://pvoutput.org/service/r2/addstatus.jsp
apikey = your-api-key
siteid = site-id-interger
#300 if paid plan. To be adapted to your setup
hitsperhour = 60
#used to manage next update according to above parameter - updates after each call
nextapicall_timestamp = 0

[bdpv]
enabled = True
url = https://www.bdpv.fr/webservice/majProd/expeditionProd_v3.php
user = your-user
api_key = you-api-key
source = perso
# 'onduleur' or 'compteur'
typereleve = onduleur
#Each day 2:00 am
dailyhour = 7200
#used to manage next update according to above parameter - updates after each call
nextapicall_timestamp = 0

[sun2000]
enabled = True
#Set your Sun2000 IP address
address = 127.0.0.1
port = 502
unit = 1
#Start register
registers_start = 32069
#Number of registers read
registers_len = 47
#Indexes in modbus response
voltagel1_index = 0
voltagel2_index = 1
voltagel3_index = 2
voltage_ratio = 10
currentl1_index = 3
currentl2_index = 5
currentl3_index = 7
current_ratio = 1000
internaltemp_index = 18
internaltemp_ratio = 10
dailyenergy_index = 45
dailyenergy_ratio = 100
lifeenergy_index = 37
lifeenergy_ratio = 100
activepower_index = 11
activepower_ratio = 1000
gridfrequency_index = 16
gridfrequency_ratio = 100
efficiency_index = 17
efficiency_ratio = 100
devicestatus_index = 20
devicestatus_ratio = 1

````


## Fonctionnement

### Test
Modifiez les droits pour permettre l'exécution
````
chmod a+x sun2000_modbus.py
````
Lancez le script
````
python3 ./sun2000_modbus.py
````
Vous devriez obtenir une réponse comme cella là :
````
{'InstantPower': 0.0, 'InternalTemp': 0.0, 'AllTimeEnergy': 4239.09, 'DailyEnergy': 20.36, 'VoltageL1': 0.0, 'VoltageL2': 0.0, 'VoltageL3': 0.0, 'CurrentL1': 0.0, 'CurrentL2': 0.0, 'CurrentL3': 0.0, 'ActivePower': 0.0, 'GridFrequency': 0.0, 'Efficiency': 0.0, 'DeviceStatusCode': '0xa000'}

Emondata: {'PUI_PROD': 0.0, 'TEMP_INT': 0.0, 'AllTimeEnergy': 4239.09, 'DailyEnergy': 20.36, 'VoltageL1': 0.0, 'VoltageL2': 0.0, 'VoltageL3': 0.0, 'CurrentL1': 0.0, 'CurrentL2': 0.0, 'CurrentL3': 0.0, 'ActivePower': 0.0, 'GridFrequency': 0.0, 'Efficiency': 0.0}

Emon data sent <Response [200]>
````
Elle donne les données lues sur l'onduleur, la mise en forme EmonCMS, le résultat de l'envoi. Si PVOutput et BDPV sont actifs, les données transmises sont également affichées ainsi que le résultat de transmission.  
Vérifiez que les données arrivent correctement sur chaque plateforme. Si c'est OK, il suffit d'automatiser

### Automatisation
Désactivez le mode débug dans le fichier de configuration.  

Vérifiez le chemin complet dans lequel se trouve le script. Comme vu ci-dessous, dans notre cas nous sommes dans `/home/emoncms/onduleur`
```sh
emoncms@emoncms:~/onduleur$ pwd
/home/emoncms/onduleur
```
Créer un fichier `sun2000.sh` dans lequel vous ajoutez les lignes ci-dessous (à adapter). Appelé chaque minute, il lance 4 exécutions séparées de 15 secondes.

````sh
#!/bin/bash
/usr/bin/python3 /home/emoncms/onduleur/sun2000_modbus.py &
sleep 15 && /usr/bin/python3 /home/emoncms/onduleur/sun2000_modbus.py &
sleep 30 && /usr/bin/python3 /home/emoncms/onduleur/sun2000_modbus.py &
sleep 45 && /usr/bin/python3 /home/emoncms/onduleur/sun2000_modbus.py &
````
Rendez le exécutable `chmod a+x sun2000.sh`
Vérifiez le fonctionnement `./sun2000.sh`, 4 jeux de données devraient arriver en 1 minute.

Ajoutez le au crontab `crontab -e`  
`* * * * * /usr/bin/sh /home/emoncms/onduleur/sun2000.sh`

C'est terminé, les données arrivent toutes les 15 secondes.

## A faire
- Améliorer le script en diminuant les appels `Global`
- Mieux utiliser la variable `Config` pour en faire un seul élément


