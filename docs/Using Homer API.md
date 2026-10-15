# HOMER 7 API

The HOMER 7 API is documented using swagger and available directly within the ADMIN UI

![image](https://user-images.githubusercontent.com/1423657/145407324-dc05160e-74a2-42b9-a79e-fe2b949074ef.png)

-------------

# HOMER 5 API (OUTDATED!)

For full documentation, please install and consult the dedicated [APIDOC](https://github.com/sipcapture/homer-api/tree/master/apidoc) available on the [API repository](https://github.com/sipcapture/homer-api). A Few basic illustrative examples are available on the main [repository](https://github.com/sipcapture/homer/tree/master/utils/scripts)

## Introduction
In order for requests to be accepted and processed by the API, a Client session ID needs to be authorized with the API Backend. Requests without an authorized or expired Cookie will always be rejected forcing Client re-authentication.

## Auth

### Auth - create a session
``` 
http://localhost/api/v1/session
```
Example usage:
```
curl -v --cookie "HOMERSESSID=tcuass65ejl2lifoopuuurpmq7; path=/" -X POST -H "Content-Type: application/json" \
-d '{"username":"admin","password":"test123"}' \
http://localhost/api/v1/session
```

#### cookie

| Field	| Type	| Description |
| ------------- |:-------------:| -----:|
| HOMERSESSID	| String | cookie session id |

Parameter

| Field	| Type	| Description |
| ------------- |:-------------:| -----:|
| username	| String | Login for session creation|
| password	| String | Password for session creation|


#### Success-Response:

The returned *session UUID* should be used in cookie request for subsequent API calls:
```
HTTP/1.1 200 OK
{
    "status": 200,
    "sid": "tcuass65ejl2lifoopuuurpmq7",
    "auth": "true",
    "message": "ok",
    "data": {
        "uid": "3",
        "username": "admin",
        "gid": "10",
        "grp": "users,admins",
        "firstname": "Alexandr",
        "lastname": "Dubovikov",
        "email": "admin@sipcapture.org",
        "lastvisit": "2015-06-18 08:25:55"
}
```
#### Error-Response:
```
HTTP/1.1 200 OK
Set-Cookie: HOMERSESSID=tcuass65ejl2lifoopuuurpmq7; path=/
Content-Type: application/json; charset=UTF-8

{
	"sid":"tcuass65ejl2lifoopuuurpmq7"
 	"auth":"false",
	"status":"wrong-session"
}
```
