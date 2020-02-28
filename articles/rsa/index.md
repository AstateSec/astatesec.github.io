# RSA, un algorithme de chiffrement asymétrique

Bonjour à tous, j'ai commencé à m'interesser à la cryptographie il n'y à pas si longtemps que ça et, franchement, c'est incroyable !! Du coup, je vous propose aujourd'hui de découvrir le fonctionnement de RSA et comment le casser bien sûr !

(Note: je suppose que vous avez tous les bases mathématiques nécessaire, rien de bien compliqué niveau collège ça passe)

D'abord, RSA kézako ? Eh bien c'est un chiffrement plutôt sûr (utilisé notamment des les protocoles ssh et https). Ce chiffrement est asymétrique, ce qui veux dire qu'il est constituer d'un couple de deux clé chiffrement/déchiffrement (nommé clé publique pour chiffrer et clé privé pour déchiffrer). 

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
