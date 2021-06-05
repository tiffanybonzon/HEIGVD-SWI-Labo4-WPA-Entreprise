- [Livrables](#livrables)

- [Échéance](#%c3%89ch%c3%a9ance)

- [Quelques éléments à considérer](#quelques-%c3%a9l%c3%a9ments-%c3%a0-consid%c3%a9rer-pour-les-parties-2-et-3)

- [Travail à réaliser](#travail-%c3%a0-r%c3%a9aliser)

# Sécurité des réseaux sans fil

## Laboratoire 802.11 Sécurité WPA Entreprise

__A faire en équipes de deux personnes__

### Objectif :

1.	Analyser les étapes d’une connexion WPA Entreprise avec une capture Wireshark
2.	__(optionnel)__ Implémenter une attaque WPE (Wireless Pwnage Edition) contre un réseau WPA Entreprise
1.  __(optionnel)__ Implémenter une attaque GTC Dowgrade contre un réseau WPA Entreprise


## Quelques éléments à considérer pour les parties 2 et 3 :

Les parties 2 et 3 sont optionnelles puisque vous ne disposez pas forcement du matériel nécessaire pour les réaliser.

En principe, il devrait être possible de démarrer vos machines en Kali natif (à partir d'une clé USB, avec une distro live par exemple) ou d'employer une autre version de Linux. Si vous n'avez pas une interface WiFi USB externe, __vous ne pouvez pas faire ces parties dans une VM Linux__. 

Dans le cas où vous arriverais à tout faire pour démarrer un Linux natif, il existe toujours la possibilité que votre interface WiFi ne puisse pas être configurée en mode AP, ce qui à nouveau empêche le déroulement des parties 2 e 3.

Ces deux parties sont vraiment intéressantes et __je vous encourage à essayer de les faire__, si vous avez les ressources. Malheureusement je ne peux pas vous proposer un bonus si vous les faites, puisqu'il risque d'y avoir des personnes qui n'auront pas la possibilité de les réaliser pour les raisons déjà expliquées.

Si vous vous lancez dans ces deux parties, voici quelques informations qui peuvent vous aider :

- Solution à l’erreur éventuelle « ```Could not configure driver mode``` » :

```
nmcli radio wifi off
rfkill unblock wlan
```
-	Pour pouvoir capturer une authentification complète, il faut se déconnecter d’un réseau et attendre 1 minute (timeout pour que l’AP « oublie » le client) 
-	Les échanges d’authentification entreprise peuvent être facilement trouvés utilisant le filtre d’affichage « ```eap``` » dans Wireshark


## Travail à réaliser

### 1. Analyse d’une authentification WPA Entreprise

Dans cette première partie, vous allez analyser [une connexion WPA Entreprise](files/auth.pcap) avec Wireshark et fournir des captures d’écran indiquant dans chaque capture les données demandées.

> Afin de mieux filtrer les paquets de cette capture (car il y en a quand meme presque 44'000...), il faut commencer par récupérer les adresses MAC des appareils concernés par l'authentification
>
> STA (supplicant): 30:74:96:70:df:32
>
> AP (authenticator): dc:a5:f4:60:bf:50
>
> Puis ajouter ces adresses aux filtres

Comparer [la capture](files/auth.pcap) au processus d’authentification donné en théorie (n’oubliez pas les captures d'écran pour illustrer vos comparaisons !). En particulier, identifier les étapes suivantes :
- Requête et réponse d’authentification système ouvert

  ![](./images/slides_openAuth.png)

  Nous avons ici la requête (0/1)

  ![](./images/wireshark_openAuthReq.png)

  Et en interchangeant les adresses source et de destination, nous obtenons la réponse (0/2)

  ![](./images/wireshark_openAuthRes.png)

 	- Requête et réponse d’association (ou reassociation)
	
	![](./images/slides_assoc.png)
	
	En filtrant pour obtenir les requêtes d'association ou de réassociation, nous obtenons une requête de réassociation
	
	![](./images/wireshark_reassocReq.png)
	
	Afin d'obtenir la réponse, nous filtrons afin d'obtenir une réponse de réassociation
	
	![](./images/wireshark_reassocRes.png)
	
- Négociation de la méthode d’authentification entreprise

  ![](./images/wireshark_negocOverview.png)

  L'AP commence par proposer à la STA d'utiliser EAP-TLS, mais le client refuse avec un `Legacy Nak` (https://datatracker.ietf.org/doc/html/rfc3748#section-5.3.1) et lui propose en retour d'utiliser EAP-PEAP

  ![](./images/wireshark_negocLegacyNak.png)

  L'AP va ensuite ré-envoyer une requête, mais cette fois-ci, avec le protocole EAP-PEAP

- Phase d’initiation. Arrivez-vous à voir l’identité du client ?

  L'Authenticator commence par demander l'identité du client

  ![](./images/wireshark_identityReq.png)

  Le client va ensuite répondre (einet\joel.gonin)

  ![](./images/wireshark_identityRes.png)

- Phase hello :

  ![](./images/slides_tls.png)

  ![](./images/slides_hellos.png)

  Le client commence avec son message `Client Hello` dans le quel il indique, entre autres, la version TLS utilisée, les cipher suites et méthodes de compression qu'il supporte, son nonce et sa session ID. Ici le client indique qu'il supporte également la version TLS 1.2 qui ne sera pas utilisée.

  ![](./images/wireshark_tlsClientHello.png)

  Le serveur répond avec un message `Server Hello` qui contient également la version TLS utilisée, le nonce, la session ID, ainsi que la cipher suite et compression utilisée

  Le paquet montré ici est reconstitués à partir des differents fragments (21498, 21502, 21508, 21512, 21517 --> Server Hello, Certificate, Server Hello Done)

  ![](./images/wireshark_tlsServerHello.png)

  - Version TLS

    - `TLS 1.0`

  - Suites cryptographiques et méthodes de compression proposées par le client et acceptées par l’AP

    - Proposées par le client

      ![](./images/wireshark_tlsCliCipherSuites.png)

    - Acceptée par le serveur

      ![](./images/wireshark_tlsServCipherSuite.png)

  - Nonces

    - Client: `955bf5b716e24a729c4b60609b8ce482014ac38f1e9cb8cf2bf8fd30bf8995f1`
    - Serveur: `003b6c2676ffd79814e56c065e5b0c39cb26600148ca1e9b3e8af83426d46e11`

  - Session ID

    - Client: `9f1bbf1e90b88366a836db08d659f906a637ac31920e06f622762ca6c522a64f`
    - Serveur: `ad41641ec2a7d1d5a9f6586c05703a8cbdbf6ef0053ad517f6e69b286804f5f2`

- Phase de transmission de certificats

  Ici, les messages entourés en rouge ne sont pas présents car la méthode utilisée est EAP-PEAP, et non EAP-TLS

    - Echanges des certificats

    ![](./images/slides_tls_certs.png)

    Les certificats du serveurs sont montrés dans le paquet recomposé `Hello Server, Certificate, Hello Done`

    ![](./images/wireshark_tlsServCert.png)

  - Change cipher spec

    ![](./images/slides_tls_changeCipherSpec.png)

    C'est ici le client qui reçoit ce message indiquant qu'il faut maintenant utiliser le tunnel TLS

    ![](./images/wireshark_tlsChangeCipherSpec.png)

- Authentification interne et transmission de la clé WPA (échange chiffré, vu comme « Application data »)

  ![](./images/slides_AuthInterne.png)

  ![](./images/wireshark_tlsData.png)

- 4-way handshake

  ![](./images/wireshark_4wayHandshake.png)

### Répondez aux questions suivantes :

> **_Question :_** Quelle ou quelles méthode(s) d’authentification est/sont proposé(s) au client ?
>
> **_Réponse :_** Le client se voit proposer les méthodes EAP-TLS et EAP-PEAP. 
>
> L'Authenticator commence par proposer au client EAP-TLS, mais le client refuse cette méthode et indique qu'il désire utiliser EAP-PEAP. L'AP va donc lui envoyer une deuxième proposition qui est, cette fois, EAP-PEAP.

---

> **_Question:_** Quelle méthode d’authentification est finalement utilisée ?
>
> **_Réponse:_** La méthode utilisée est EAP-PEAP. Ceci peut être vérifié dans le message `Client Hello`
>
> ![](./images/answers_authMethod.png)

---

> **_Question:_** Lors de l’échange de certificats entre le serveur d’authentification et le client :
> 
> - a. Le serveur envoie-t-il un certificat au client ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Le serveur envoie son certificat afin que le client puisse l'authentifier
> 
> - b. Le client envoie-t-il un certificat au serveur ? Pourquoi oui ou non ?
> 
> **_Réponse:_** Non car la méthode d'authentification est EAP-PEAP et non EAP-TLS. EAP-PEAP utilise MSCHAPv2 pour authentifier le client grâce à un challenge-response à la place d'un certificat.

---

### 2. (__Optionnel__) Attaque WPA Entreprise (hostapd)

Les réseaux utilisant une authentification WPA Entreprise sont considérés aujourd’hui comme étant très surs. En effet, puisque la Master Key utilisée pour la dérivation des clés WPA est générée de manière aléatoire dans le processus d’authentification, les attaques par dictionnaire ou brute-force utilisés sur WPA Personnel ne sont plus applicables. 

Il existe pourtant d’autres moyens pour attaquer les réseaux Entreprise, se basant sur une mauvaise configuration d’un client WiFi. En effet, on peut proposer un « evil twin » à la victime pour l’attirer à se connecter à un faux réseau qui nous permette de capturer le processus d’authentification interne. Une attaque par brute-force peut être faite sur cette capture, beaucoup plus vulnérable d’être craquée qu’une clé WPA à 256 bits, car elle est effectuée sur le compte d’un utilisateur.

Pour faire fonctionner cette attaque, il est impératif que la victime soit configurée pour ignorer les problèmes de certificats ou que l’utilisateur accepte un nouveau certificat lors d’une connexion.

Pour implémenter l’attaque :

- Installer ```hostapd-wpe``` (il existe des versions modifiées qui peuvent peut-être faciliter la tâche... je ne les connais pas. Dans le doute, utiliser la version originale). Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```hostapd-wpe``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence, sachant que dans le cas d'une attaque réelle, il faudrait utiliser le vrai SSI du réseau de la cible
- Lancer une capture Wireshark
- Tenter une connexion au réseau (ne pas utiliser vos identifiants réels)
- Utiliser un outil de brute-force (```john```, par exemple) pour attaquer le hash capturé (utiliser un mot de passe assez simple pour minimiser le temps)

### Répondez aux questions suivantes :

> **_Question :_** Quelles modifications sont nécessaires dans la configuration de hostapd-wpe pour cette attaque ?
> 
> **_Réponse :_** 

---

> **_Question:_** Quel type de hash doit-on indiquer à john pour craquer le handshake ?
> 
> **_Réponse:_** 

---

> **_Question:_** Quelles méthodes d’authentification sont supportées par hostapd-wpe ?
> 
> **_Réponse:_**


### 3. (__Optionnel__) GTC Downgrade Attack avec [EAPHammer](https://github.com/s0lst1c3/eaphammer) 

[EAPHammer](https://github.com/s0lst1c3/eaphammer) est un outil de nouvelle génération pour les attaques WPA Entreprise. Il peut en particulier faire une attaque de downgrade GTC, pour tenter de capturer les identifiants du client en clair, ce qui évite le besoin de l'attaque par dictionnaire.

- Installer ```EAPHammer```. Lire la documentation du site de l’outil ou d’autres ressources sur Internet pour comprendre son utilisation
- Modifier la configuration de ```EAPHammer``` pour proposer un réseau semblable au réseau de l’école ou le réseau de votre préférence. Le but est de réaliser une GTC Downgrade Attack.
- Lancer une capture Wireshark
- Tenter une connexion au réseau


### Répondez aux questions suivantes :

> **_Question :_** Expliquez en quelques mots l'attaque GTC Downgrade
> 
> **_Réponse :_** 

---

> **_Question:_** Quelles sont vos conclusions et réflexions par rapport à la méthode hostapd-wpe ?
> 
> **_Réponse:_** 


## Livrables

Un fork du repo original . Puis, un Pull Request contenant :

-	Captures d’écran + commentaires
-	Réponses aux questions

## Échéance

Le 9 juin 2021 à 23h59
