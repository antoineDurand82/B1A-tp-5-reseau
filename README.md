# B1A-tp-5-reseau


## 1. Préparation VMs

C'est beau

![C'est beau](gns.png)
**Réseaux :**

* `net1` : `10.5.1.0/24`
* `net2` : `10.5.2.0/24`
* `net12` : **votre choix** (à justifier)

**Machines :**

Machine | `net1` | `net2` | `net12`
--- | --- | --- | ---
`client1.tp5.b1` | X | `10.5.2.10` | X
`client2.tp5.b1` | X | `10.5.2.11` | X
`router1.tp5.b1` | `10.5.1.254` | X | `10.5.12.1`
`router2.tp5.b1` | X | `10.5.2.254` | `10.5.12.2`
`server1.tp5.b1` | `10.5.1.10` | X | X

# II. Lancement et configuration du lab

Lancez toutes les machines (ou une par une). Je vous conseille de vous posez tranquillement, et de vous conformez à une liste d'étapes pour ce faire. Ici encore je la fais pour vous, **habituez-vous à utiliser ce genre de petites techniques pour gagner en rigueur**.  

**Prenez des notes de ce que vous faites.**  

### Checklist IP VMs 

On parle de `client1.tp5.b1`, `client2.tp5.b1` et `server1.tp5.b1` :
* [X] Désactiver SELinux
  * déja fait dans le patron
* [X] Installation de certains paquets réseau
  * déja fait dans le patron
* [X] Désactivation de la carte NAT
  * déja fait dans le patron
* [ ] [Définition des IPs statiques](../../cours/procedures.md#définir-une-ip-statique)
* [ ] La connexion SSH doit être fonctionnelle
  * une fois fait, vous avez vos trois fenêtres SSH ouvertes, une dans chaque machine
* [ ] [Définition du nom de domaine](../../cours/procedures.md#changer-son-nom-de-domaine)

### Checklist IP Routeurs 

On parle de `router1.tp5.b1` et `router2.tp5.b1` :
* [ ] [Définition des IPs statiques](../../cours/procedures-cisco.md#définir-une-ip-statique)
* [ ] [Définition du nom de domaine](../../cours/procedures-cisco.md#changer-son-nom-de-domaine)

### Checklist routes 

On parle de toutes les machines :
* [ ] `router1.tp5.b1`  
  * directement connecté à `net1` et `net12`  
  * [route à ajouter](../../cours/procedures-cisco.md#ajouter-une-route-statique) : `net2`  
* [ ] `router2.tp5.b1`
  * directement connecté à `net2` et `net12`  
  * [route à ajouter](../../cours/procedures-cisco.md#ajouter-une-route-statique) : `net1`  
* [ ] `server1.tp5.b1`  
  * directement connecté à `net1`  
  * [route à ajouter](../../cours/procedures.md#ajouter-une-route-statique) : `net2`
  * [fichiers `hosts`](../../cours/procedures.md#editer-le-fichier-hosts) à remplir : `client1.tp5.b1`, `client2.tp5.b1`
* [ ] `client1.tp5.b1`
  * directement connecté à `net2`  
  * [route à ajouter](../../cours/procedures.md#ajouter-une-route-statique) : `net1`
  * [fichiers `hosts`](../../cours/procedures.md#editer-le-fichier-hosts) à remplir : `server1.tp5.b1`, `client2.tp5.b1`
* [ ] `client2.tp5.b1`
  * directement connecté à `net2`  
  * [route à ajouter](../../cours/procedures.md#ajouter-une-route-statique) : `net1`
  * [fichiers `hosts`](../../cours/procedures.md#editer-le-fichier-hosts) à remplir : `server1.tp5.b1`, `client1.tp5.b1`
* [ ] `router1.tp5.b1`  
  * directement connecté à `net2`  
  * [route à ajouter](../../cours/procedures.md#ajouter-une-route-statique) : `net1`  

Pour tester : 
* remplir [les fichiers `hosts`](../../cours/procedures.md#editer-le-fichier-hosts) des VMs Linux
* les deux clients doivent pouvoir `ping server1.tp5.b1`
* et réciproquement :fire:

> **Notez que les clients/serveurs n'ont pas de route vers `net12`**. Et ui. C'est un réseau privé que seuls les routeurs connaissent. 

# III. DHCP
Attribuer des IPs statiques et des routes sur les VMs c'est chiant non ? **Serveur [DHCP](../../cours/lexique.md#dhcp--dynamic-host-configuration-protocol)** à la rescousse.  

Une section dédiée popera dans le cours d'ici peu.  

Un serveur [DHCP](../../cours/lexique.md#dhcp--dynamic-host-configuration-protocol) :
* permet d'attribuer dynamiquement des IPs
  * on a pas besoin de les définir à la main
* est principalement utilisé pour des [clients](../..//cours/3.md#clientserveur)
  * on préfère avoir des IPs fixes (statiques) pour les [serveurs](../../cours/3.md#clientserveur) et les équipements réseaux (comme les [routeurs](../../cours/lexique.md#routeur))
  * ce serait un peu le dawa s'ils changeaient tout le temps
* permet aussi de distribuer d'autres infos aux [clients](../..//cours/3.md#clientserveur)
  * comme des routes !

## 1. Mise en place du serveur DHCP

On va recycler `client2.tp5.b1` pour ça (pour économiser un peu de ressources).  

**1. [Renommer la machine](../../cours/procedures.md#changer-son-nom-de-domaine**)
  * pour porter le nom `dhcp-net2.tp5.b1`

**2. Installer le serveur DHCP** en faisant un peu de crasse : 
  * éteindre la VM dans GNS3
  * ouvrir VirtualBox
  * ajouter une carte NAT à la VM
  * démarrer la VM dans VirtualBox
  * allumer la carte NAT
  * `sudo yum install -y dhcp` 
  * shutdown la VM

**3. Rallumer la VM dans GNS**

**4. Configuration du serveur DHCP**
* le fichier de configuration se trouve dans `/etc/dhcp/dhcpd.conf`
  * [un modèle est trouvable ici](./dhcp/dhcpd.conf)

**5. Faire un test**
* avec une nouvelle VM ou `client1.tp5.b1`
  * [configurer l'interface en DHCP, en dynamique (pas en statique)](../../cours/procedures.md#définir-une-ip-dynamique-dhcp)
  * utiliser [`dhclient`](../../cours/lexique.md#dhclient-linux-only)
* dans un cas comme dans l'autre, vous devriez récupérer une IP dans la plage d'IP définie dans `dhcpd.conf`

## 2. Explorer un peu DHCP
Le principe du protocole DHCP est le suivant : 
* on a un serveur sur un réseau, il attend que des clients lui demande des IPs
* des clients peuvent arriver sur le réseau (câble, WiFi, ou autres) et demander une IP
* le serveur attribuera une IP dans une plage prédéfinie
* le serveur va créer un **"bail DHCP"** par client, pour s'en souvenir
  * **dans le bail il y a écrit "j'ai donné telle IP à telle MAC"**
  * comme ça, si le même client revient, il garde son IP

---

La discussion entre le client et le serveur DHCP se fait en 4 messages simples, **"DORA"** :
* **"Discover"** : du client vers le serveur
  * le client cherche un serveur DHCP en envoyant des Discover en broadcast
* **"Offer"** : du serveur vers le client
  * Si un serveur reçoit un "Discover" il peut répondre un "Offer" au client
  * Il propose une IP au client
* **"Request"** : du client vers le serveur
  * Permet de demander une IP au serveur
  * C'est celle que le serveur lui a proposé
* **"Acknowledge"** : du serveur vers le client
  * Le serveur attribue l'adresse IP au client
  * Il crée un bail DHCP en local
  * Il peut aussi fournir au client d'autres infos comme l'adresse de gateway

---

**OKAY**, le but : 
* faire une demande DHCP
  * avec [`dhclient`](../../cours/lexique.md#dhclient-linux-only)
  * capturer avec Wireshark l'échange du DORA
    * vous pouvez `tcpdump` sur le `client1.tp5.b1` ou sur `dhcp-net2.tp5.b1`
    * ou vous pouvez clic-droit sur un lien dans GNS3 et lancer une capture

---

### IV. Bonus