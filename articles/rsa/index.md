# RSA, un algorithme de chiffrement asymétrique

Bonjour à tous, j'ai commencé à m'interesser à la cryptographie il n'y à pas si longtemps que ça et, franchement, c'est incroyable !! Du coup, je vous propose aujourd'hui de découvrir le fonctionnement de RSA et comment le casser bien sûr !

(Note: je suppose que vous avez tous les bases mathématiques nécessaire, rien de bien compliqué niveau collège ça passe)

D'abord, RSA kézako ? Eh bien c'est un chiffrement plutôt sûr (utilisé notamment dans les protocoles ssh et https). Ce chiffrement est asymétrique, ce qui veux dire qu'il est constituer d'un couple de deux clé chiffrement/déchiffrement (nommé clé publique pour chiffrer et clé privé pour déchiffrer). 

Pour que ça sois plus clair avec la cryptographie asymétrique je vous propose un exemple, deux ami Robert, et Michel (très original ...) qui souhaitent échanger des messages chiffrés, d'abord, ils font chacun une paire de clé publique/privé, ensuite ils s'échangent leurs clé publique. Michel souhaite envoyer "Je t'aime" à Robert (pas de jugement s'il vous plait mdr, je tiens à préciser que je ne suis pas pd), pour celà il chiffre son message avec la clé publique de Robert puis envoie le message chiffré, si Annie, la copine de Robert intercèpte le message elle ne pourra rien faire car pour déchiffrer le message il faut la clé privé de Robert, le message arrive donc chez Robert qui le déchiffre avec sa clé privé et découvre les sentiments de Michel pour lui. Malheuresement, Robert laisse le message déchiffré sur son bureau et Annie le découvre .. (Excuse, je voulais que l'histoire finisse mal)

## Et RSA dans tout ça ? 

Eh bien passons au cher sujet ! Comment fonctionne RSA ? Alors on va faire ça en étape, reprennons Robert qui faisait sa pair clé publique/privée, disons que c'était du RSA.. Comment avait il fais pour génerer ses clé : 

```
1. D'abord, il choisis deux nombre premier qu'on nomme par convention p et q
2. Il note n=p*q , n représente le "module de chiffrement"
3. Il prend aussi Phi(n)=(p-1)*(q-1), c'est "l'indicatrice d'Euler" pour n, cette fonction donne le nombre d'entier premier avec n entre 1 et n 
4. Il prend un nombre e, premier avec Phi(n), c'est "l'exposant de chiffrement"
5. Il prend le nombre d, inverse de e mod Phi(n) (mod correspond au modulo, reste de la division)

Et voilà ! La clé publique est (e, n) et la clé privée est (d, n) :)
```

Super mais comment on chiffre et déchiffre avec ces clés ? 
On va juste noter M le texte en clair et C le texte chiffré (Les normes ¯\_(ツ)_/¯ )

Pour obtenir C à partir de M on fais: C=M^e mod n (le "^" correspond à puissance)
Et pour déchiffrer M=C^d mod n

## Et en pratique ?

Bon, evidemment on ne va pas faire ça à la main ! On va utiliser OpenSSL, rien de bien complexe juste deux commandes à connaitre:
```
openssl genrsa -out key.pem nombre_de_bits
```

Celà génère tout simplement une clé privée, et pour la clé publique on va faire:

```
openssl rsa -in key.pem -outform PEM -pubout -out public.pem
```

Pour chiffrer un message:
```
openssl rsautl -encrypt -pubin -inkey public.pem -in crypto.txt -out crypto.enc
```

Dans mon cas:
```
astate@sec:~$ openssl genrsa -out key.pem 100
Generating RSA private key, 100 bit long modulus
..++++++++++++++++++++++++++++++++++
.++++++++++++++++++++++++++++++++++
e is 65537 (0x010001)
astate@sec:~$ cat key.pem
-----BEGIN RSA PRIVATE KEY-----
MFICAQACDQuQTGzMW4J0xgM4OX8CAwEAAQINALoZk91lAukP0NOjAQIHA2lwQF7G
cQIHA2OlrMNW7wIGWegeGCNhAgcBH5QYMR6tAgcCqnJlVH0l
-----END RSA PRIVATE KEY-----
astate@sec:~$ openssl rsa -in key.pem -outform PEM -pubout -out public.pem
writing RSA key
astate@sec:~$ cat public.pem
-----BEGIN PUBLIC KEY-----
MCgwDQYJKoZIhvcNAQEBBQADFwAwFAINC5BMbMxbgnTGAzg5fwIDAQAB-----BEGIN PUBLIC KEY-----
MCgwDQYJKoZIhvcNAQEBBQADFwAwFAINC5BMbMxbgnTGAzg5fwIDAQAB
-----END PUBLIC KEY-----
-----END PUBLIC KEY-----
astate@sec:~$ 
```
Enfait, vous ne devrez pas faire ça.. La clé est beaucoup trop courte et on peut la cracker "par factorisation", voyons ça !

## Craquer RSA par factorisation !

Immaginons un scénario, reprennons Annie, Robert et Michel.

Robert et Michel ont reussi à convaincre Annie que l'histoire du "Je t'aime" n'étais qu'une blague, Annie demanda cependant la clé publique de Robert son copain. Il accepta en se disant "Hahaha, de toute façon on ne peux pas casser RSA et il faut ma clé privée pour déchiffrer les messages de Michel". Sauf que les clés publiques et privées de Robert sont trop courtes.

Clé publique de Robert: 
```
-----BEGIN PUBLIC KEY-----
MDUwDQYJKoZIhvcNAQEBBQADJAAwIQIaALfs7164q7PNWxxr8SubgHr1cuIy6GeE
fqUCAwEAAQ==
-----END PUBLIC KEY-----
```

Annie en se reveillant le matin trouve encore un message sur l'ordinateur de son copain, sauf que là il est chiffré, avec la clé publique de Robert... 

Le message est illisible, elle décide de faire un hexdump dessus pour voir à quoi ça ressemble:

```
0000000 d037 7d68 24de 570b 5317 27aa d7be 86a8
0000010 7434 814a 1b80 2f66 003e               
0000019
```

Annie est convaincue que ce message est bizarre, que contient il ! Elle dois trouver cette clé privé ! 

Rappelez vous la clé publique c'est (e, n) et la clé privé c'est (d, n), il suffit donc de trouver d pour que Annie puisse lire tout les messages de Michel..

Vous voyez sur la clé publique qu'on ne reconnais pas directement le e et le n, pour les retrouver on va encore utiliser openssl avec la commande suivante:
```openssl rsa -in public.pem -pubin -text -modulus```

Eh bien ça donne:

```
astate@sec:~$ openssl rsa -in public.pem -pubin -text -modulus
Public-Key: (200 bit)
Modulus:
    00:b7:ec:ef:5e:b8:ab:b3:cd:5b:1c:6b:f1:2b:9b:
    80:7a:f5:72:e2:32:e8:67:84:7e:a5
Exponent: 65537 (0x10001)
Modulus=B7ECEF5EB8ABB3CD5B1C6BF12B9B807AF572E232E867847EA5
writing RSA key
-----BEGIN PUBLIC KEY-----
MDUwDQYJKoZIhvcNAQEBBQADJAAwIQIaALfs7164q7PNWxxr8SubgHr1cuIy6GeE
fqUCAwEAAQ==
-----END PUBLIC KEY-----
astate@sec:~$ 

```
Alors notre n, c'est simplement B7ECEF5EB8ABB3CD5B1C6BF12B9B807AF572E232E867847EA5 (c'est de l'hexadécimal)
Et notre e c'est 65537

Enfait on va essayer de trouver les p et q à partir de n (si celui-ci est trop grand, ça sera impossible)

On va juste mettre notre chiffre en décimal avec ruby :
```
astate@sec:~$ irb 
irb(main):001:0> 0xB7ECEF5EB8ABB3CD5B1C6BF12B9B807AF572E232E867847EA5
=> 1154519247829685809469889775025194060140524275470441627549349
irb(main):002:0> 
``` 

Très bien, maintenant il est facile de trouver p et q à partir de factordb: [Lien vers FactorDB](https://factordb.com) 
On trouve donc:
p = 1048382826719471131832915989793
q = 1101238229399779097627028780293

C'est bon ! Tout est fais Annie décide d'écrire un script en python pour retrouver la clé privée (je voulais le faire en ruby mais dommage, ça marchais pas :( )

Je suppose que si vous avez déjà fais du python, vous avez surement utiliser gmpy, sinon vous pouvez l'installer via ```sudo pip3 install gmpy```. Annie aura aussi besoin de la librairie Crypto, elle est par défaut installé

Voici le script de Annie pour retrouver la clé privée (le code est commenté, vous ne devriez pas avoir de problème à comprendre)

```python
from Crypto.PublicKey import RSA
from math import *
import gmpy

# p et q, cracké par factorisation via factodb
p = 1048382826719471131832915989793
q = 1101238229399779097627028780293

# n = p*q
n = 1154519247829685809469889775025194060140524275470441627549349

# Le phi
phi = (p-1)*(q-1)

# Notre exposant de chiffrement
e = 65537

# Et enfin le d, ce qu'on cherchait ;-)
d = int(gmpy.invert(e, phi))

# On recolle le tout et on print la clé privée
key = RSA.construct((n, e, d))
print(str(key.exportKey()))
```

![Like a Boss](image.png)

Voilà ! Doonc il nous reste plus qu'a remmettre en place un peu:
On passe à ça
```
-----BEGIN RSA PRIVATE KEY-----
MIGKAgEAAhoAt+zvXrirs81bHGvxK5uAevVy4jLoZ4R+pQIDAQABAhkeDSR/QYUs
d50vY6OboYbrkl+Fi7bR3foBAg0N5krpxPdvKAl4YnUFAg0NO4Hy5EmIYtdXchUh
Ag0EV+Uq3XiwTGOBMcr9Ag0J1rpLuJQ4NY36NhghAg0Dw4L8rwZMJs2WtGep
-----END RSA PRIVATE KEY-----
```
(Google -> RSA key beautifier)

Annie met tout ça dans key.pem et utilise une dernière fois openssl puis cat, elle retient son soufle :
```openssl rsautl -decrypt -inkey key.pem -in crypto.enc -out crypto.out```
Elle a à enfin le courage de faire un ```cat crypto.out```


Elle cours vers la cuisine, prend un couteau, retrouve Michel et Robert en train de jouer à la Xbox au salon, elle les tue, coupe leurs oreilles, les lance dans la poubelle puis se suicide ...

Triste fin .. (Je vous ai dit que je kiff bien ça). La dernière commande de Annie fut cat crypto.out...
Morale de l'Histoire: Si vous êtes homosexuel, utiliser des clé publique/privée RSA assez longue ..

Sur ce je vous laisse ! A plus ! Si vous avez besoin de me contacter:
Twitter: @AstateSec
Discord: Astate#4251
