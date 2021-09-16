# keeping track of usefull command in zimbra

## Rights management

**Check all the rights in a domain** :   
`zmprov gg -t domain <DomainName>`

**Add the end user right to create a DL** :  
`zmprov grantRight domain <DomainName> usr <account> createDistList`

**Easy one-line with a file for mass integration** :  
`for i in $(cat /tmp/list); do zmprov grantRight domain <<DomainName usr $i createDistList; done`

Zimbra documentation recommend clearing cache after the command :  
`zmprov fc -a account user@domain.com`


this does not work in a script :    `ERROR: service.INVALID_REQUEST (invalid request: unknown cache type: user@domain.com`