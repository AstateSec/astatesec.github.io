# RSA, un algorithme de chiffrement asymétrique

Bonjour à tous, j'ai commencé à m'interesser à la cryptographie il n'y à pas si longtemps que ça et, franchement, c'est incroyable !! Du coup, je vous propose aujourd'hui de découvrir le fonctionnement de RSA et comment le casser bien sûr !

(Note: je suppose que vous avez tous les bases mathématiques nécessaire, rien de bien compliqué niveau collège ça passe)

D'abord, RSA kézako ? Eh bien c'est un chiffrement plutôt sûr (utilisé notamment dans les protocoles ssh et https). Ce chiffrement est asymétrique, ce qui veux dire qu'il est constituer d'un couple de deux clé chiffrement/déchiffrement (nommé clé publique pour chiffrer et clé privé pour déchiffrer). 

Pour que ça sois plus clair avec la cryptographie asymétrique je vous propose un exemple, deux ami Robert, et Michel (très original ...) qui souhaitent échanger des messages chiffrés, d'abord, ils font chacun une paire de clé publique/privé, ensuite ils s'échangent leurs clé publique. Michel souhaite envoyer "Je t'aime" à Robert (pas de jugement s'il vous plait mdr, je tiens à préciser que je ne suis pas pd), pour celà il chiffre son message avec la clé publique de Robert puis envoie le message chiffré, si Annie, la copine de Robert intercèpte le message elle ne pourra rien faire car pour déchiffrer le message il faut la clé privé de Robert, le message arrive donc chez Robert qui le déchiffre avec sa clé privé et découvre les sentiments de Michel pour lui. Malheuresement, Robert laisse le message déchiffré sur son bureau et Annie le découvre .. (Excuse, je voulais que l'histoire finisse mal)

## Et RSA dans tout ça ? 

Eh bien passons au cher sujet ! Comment fonctionne RSA ? Alors on va faire ça en étape, reprennons Robert qui faisais ça pair clé publique/privée, disons que c'était du RSA.. Comment avait il fais pour génerer ses clé : 

```
1. D'abord, il choisis deux nombre premier qu'on nomme par convention p et q
2. Il note n=p*q , n représente le "module de chiffrement"
3. Il prend aussi Phi(n)=(p-1)*(q-1), c'est "l'indicatrice d'Euler" pour n
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
openssl -rsault -encrypt -pubin -inkey public.pem -in crypto.txt -out crypto.enc
```

Dans mon cas:
```astate@sec:~$ openssl genrsa -out key.pem 100
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

Robert et Michel ont reussi à convaincre Annie que l'histoir du "Je t'aime" n'étais qu'une blague, Annie demanda cependant la clé publique de Robert son copain. Il accepta en se disant "Hahaha, de toute façon on ne peux pas casser RSA et il faut ma clé privée pour déchiffrer les messages de Michel". Sauf que les clé publique et privée de Robert sont trop courtes.

Clé publique de Robert: 
```-----BEGIN PUBLIC KEY-----
MCgwDQYJKoZIhvcNAQEBBQADFwAwFAINC5BMbMxbgnTGAzg5fwIDAQAB
-----END PUBLIC KEY-----
```

Rappelez vous la clé publique c'est (e, n) et la clé privé c'est (d, n), il suffit donc de trouver d pour que Annie puisse lire tout les messages de Michel..

Annie en se reveillant le matin trouve encore un message sur le bureau de son copain, sauf que là il est chiffrer, avec la clé publique de Robert... 

