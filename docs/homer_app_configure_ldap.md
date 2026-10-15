## Configuration block for the LDAPS protocol

```
  "ldap_config": {
    "base": "dc=example,dc=local",
    "host": "example.local",
    "port": 636,
    "usessl": true,
    "skiptls": false,
    "binddn": "CN=homer-svc,OU=ServiceUsers,DC=example,DC=local",
    "bindpassword": "HighlySuperSecretServicePassword",
    "userfilter": "(&(sAMAccountName=%s)(|(memberof:1.2.840.113556.1.4.1941:=CN=homer_user_group,OU=Resources,DC=example,DC=local)(memberof:1.2.840.113556.1.4.1941:=CN=homer_admin_group,OU=Resources,DC=example,DC=local)))",
    "groupfilter": "(&(objectCategory=group)(name=homer*)(member:1.2.840.113556.1.4.1941:=%s))",
    "group_attributes": [
        "cn"
    ],
    "admingroup": "homer_admin_group",
    "adminmode": false,
    "usergroup": "homer_user_group",
    "usermode": false,
    "attributes": ["dn", "givenName", "sn", "mail", "sAMAccountName"],
    "skipverify": false,
    "anonymous": false,
    "userdn": "CN=%s,OU=Users,DC=example,DC=local"
  }
``` 

* Authorization allowed only for users from groups homer_admin_group and homer_user_group (configured by `userfilter`);
* Active Directory should only look for user membership in groups whose names begin with `homer` (configured by `groupfilter`);
* `group_attributes` - a list of attributes by which to search for the names of the groups defined in `admingroup` and `usergroup`.
