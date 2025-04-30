# LDAP Enumeration

## Tools
- [LDAPSearch](https://ldapsearch.sourceforge.net/)
- [python-ldapdomaindump](https://www.kali.org/tools/python-ldapdomaindump/)
- [msLDAPDump](https://github.com/dievus/msLDAPDump)
- etc TODO


### Comandos ldapsearch

#### Anonimo
- `ldapsearch -LLL -x -H ldap://test.local -b'' -s base '(objectclass=\*)'`
- `ldapsearch -LLL -x -H ldap://test.local -b'' -s base '(objectclass=\*)' namingContexts`
- `ldapsearch -LLL -x -H ldap://test.local -b'' -s base '(objectclass=\*)' defaultNamingContext`
- Ahora con el namingContext podemos hacer un search:
    - `ldapsearch -LLL -x -H ldap://test.local -b 'DC=test,DC=local' '(objectclass=\*)'`
    - `ldapsearch -LLL -x -H ldap://test.local -b 'DC=test,DC=local'`