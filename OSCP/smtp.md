# SMTP/POP/IMAP Mail Server Pentesting

## Reconocimiento en imap

- **Chequear emails:**
    - `nc IP 143`
    - `tag login mail password`: Para loguearse
    - `tag LIST "" "*"`: Para listar los mails
    - `tag SELECT INBOX`: Para seleccionar el inbox
    - `tag STATUS INBOX (MESSAGES)`: Para ver la cantidad de mails
    - `tag fetch 1 (BODY[1])`: Para ver el mail
    - `tag fetch 2:5 BODY[HEADER] BODY[1]`: Para ver el header de los mails
    - `tag FETCH 1:5 ENVELOPE`: Para ver el envelope de los mails

### Recon automatico de imap
- `imapsync`: Herramienta para sincronizar cuentas de imap. Puede ser útil para hacer un dump de los mails.
- [imap-recon tool](https://github.com/G4sp4rCS/imap-recon)