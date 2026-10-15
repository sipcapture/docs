# HOMER Authentication

## Local _(default)_
By default, HOMER uses its internal database for users and permissions:
```javascript
 "auth_settings": {
    "_comment": "The type param can be internal, ldap, http_auth",
    "token_expire": 1200,
    "type": "internal"
  }
```

### LDAP
The following example contains an LDAP integration template for generic services:
```javascript
"auth_settings": {
    "token_expire": 1200,
    "type": "ldap"
},
"ldap_config": {
    "admingroup": "admin",
    "adminmode": true,
    "anonymous": false,
    "attributes": [
      "dn",
      "givenName",
      "sn",
      "mail",
      "uid"
    ],
    "base": "dc=example,dc=com",
    "binddn": "uid=readonlysuer,ou=People,dc=example,dc=com",
    "bindpassword": "readonlypassword",
    "groupattribute": "cn",
    "groupfilter": "(memberUid=%s)",
    "host": "ldap.example.com",
    "port": 389,
    "skiptls": true,
    "skipverify": true,
    "userdn": "uid=%s,ou=People,dc=example,dc=com",
    "userfilter": "(uid=%s)",
    "usergroup": "HOMER_user",
    "usermode": true,
    "usessl": false
  },
```

### HTTP/S AUTH
Authentication can be delegated to an external API using the HTTP/S AUTH type and JSON User objects
```
"auth_settings": {
    "token_expire": 1200,
    "type": "http_auth"
},
"http_auth": {
    "skipverify": true,
    "url": "http://localhost:1323"
},
```

##### Request
```
{
   "username": "httpuser",
   "password": sha1(password),
}
```
##### Response
```
   "id": 122345
   "username": "httpuser",
   "partid": 10,
   "email": "homer@rocks",
   "firstname": "Homer",
   "lastname": "SIP",
   "department": "Beer",
   "usergroup": "family",
   "guid": UUID
```


## OAuth2
The following example contains an OAuth2 integration with a generic service. This works in parallel to other methods.
```javascript
"oauth2": {
    "enable": true,
    "client_id": "XXXXXXXXXXXX-xxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com",
    "client_secret": "XXXXXX-XXXXXXXXXXXXXXXXXXXXXXXXXXXX",
    "project_id": "Homer OAuth",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url":  "https://www.googleapis.com/oauth2/v1/certs",
    "redirect_uri": "http://YOUR_HOMER_URL/api/v3/oauth2/auth",
    "profile_uri": "https://www.googleapis.com/oauth2/v1/userinfo",
    "provider_name": "google",
    "provider_image": ""
}
```
