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

##  AD & LDAP

**Connecting zimbra to AD with LDAPS** :

*Original topic with theses infos* : https://forums.zimbra.org/viewtopic.php?t=67633

1) Get ldap url : `zmprov gd mydomain.com zimbraAuthLdapURL`  
2) Verify LDAP port is reachable  
3) Export AD CA Certificate :  
```
1.  Open the **Certification Authority** console from any domain-joined computer or server. 
    This console must be attached to the certification authority. 
    The Certification Authority console can be opened by searching for "Certification Authority" (CA) in the start button
    or going to Run and using `certsrv.msc` command.
2.  Right-click on the name of the certification authority and then select **Properties**.
3.  In the **CA certificates** dialog box, choose the **General** tab and select the certificate for the CA you want to access.
4.  Choose **View Certificate**.
5.  In the Certificate dialog box, choose the **Certification Authority** tab. 
    Select the name of the root CA and then choose **View Certificate**.
6.  In the Certificate dialog box, choose the **Details** tab and then choose **Copy to File**.
7.  The Certificate Export Wizard will appear. Choose **Next**; no need to export private key.
8.  On the Export File Format page, select the **Base-64 encoded binary X.509(.CER)**option.
9.  Choose *Next**.
10. In the **File to Export**box, choose the path and name for the certificate, and then choose **Next**.
11. Choose **Finish**. The.cerfile will be created in the location that you specified in the previous step.
12. Finally, a dialog box will appear to inform the user that the export was successful. Choose **OK**to finish.
```
4) Copy the cert on the mailstore, add it to the Zimbra keystore, then restart mailboxd :  
```
vi /tmp/ad-ca.cer then paste the certificate in pem format  
zmcertmgr addcacert /tmp/ad-ca.cer  
zmmailboxdctl restart
```
5) Modify ldap auth url for that domain pointing to ldaps  
`zmprov md mydomain.com zimbraAuthLdapURL "ldaps://myad.mydomain.com:636"`

Some notes :  
```
Some notes:  
- Step 4 must be done for each mailstore that holds mailboxes that could need authentication against that AD  
- If you have multiple domain that authenticate against different ADs, all the steps must be repeated for each one
```