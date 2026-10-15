<img src="https://user-images.githubusercontent.com/1423657/55069501-8348c400-5084-11e9-9931-fefe0f9874a7.png" width=150 />

# Testing HOMER with hepgen.js

Not sure if your HOMER setup is operational? No problem!

[hepgen.js](https://github.com/sipcapture/hepgen.js) simulates a HEP capture agent and allows you to quick validate your system. Think of it as a SIPP for HEP packets you can use to simulate various scenarios and features.

### Requirements
* [NodeJS 14.x](https://nodejs.org/en/download/package-manager) or higher

### Installation
Install `hepgen.js` globally using npm
```
npm install -g hepgen.js
```

### Testing
Let's run `hepgen.js` against oue HOMER deployment!

```
hepgen.js -s HOMER_IP -p HOMER_PORT -c $(npm root -g)/hepgen.js/config/b2bcall_rtcp.js
```

Replace `HOMER_IP` _(127.0.0.1)_ and `HOMER_PORT` _(9060)_ to point at your setup. You want to simulate an agent!


### Validation
Login to your HOMER instance, set the time range to the last 10 minutes, and search. Your HEPgen.js call should appear.

Our synthetic HEP session will contain randomly generated IPs and numbers from SIP user `hepgenjs`

![image](https://user-images.githubusercontent.com/1423657/114322818-7f305780-9b22-11eb-922f-b03d5ee07024.png)





For more information, please refer to the [hepgen.js](https://github.com/sipcapture/hepgen.js) repository.
