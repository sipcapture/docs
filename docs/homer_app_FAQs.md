
####  1) What are the credentials for Homer dashboard login?

      Username: admin
     Password: sipcapture


####  2) how do I upgrade the UI changes alone without affecting the homer-app core functionalities?

       Homer-App UI is driven by the dist folder in the source code. to upgrade the UI part we can take the release version that we want and replace the dist folder with homer-ui-<version>.tar.gz dist folder and files


#### 3) How data retention policy is taken care of within Homer setup  - PostgreSQL Data?

      Currently, this is controlled from Heplify-server **DBDropDays = 14**. We maintain 2 weeks of data. The rest of the data will be purged automatically.
      ### File: heplify-server.toml


#### 4) Can we set a retention policy for different types of data? say calls 14 days and Registration 4 days?

      Yes, The same can be achieved via Heplify-server config. Below are the params for the same which we need to change in the below-mentioned file.

      `DBDropDaysCall = 14`
      `DBDropDaysRegister = 4`

     ### File: heplify-server.toml

#### 5) How data is stored with PostgreSQL integration? How do view the data for Call and Registration SIP traces?

      Homer stores the SIP traces with respect to **`Call`** and **`Registration`** data in a seperate tables which can be partitioned by hourly or daily(upto our convenient). So in order to view the data in the dashboard we need to select respective HEP Proto.
     **For Example:**
             i. To view the Call SIP traces we need to select the Proto as **SIP - call**.
             ii.To view the Registration SIP traces we need to select the Proto as **SIP - registration**.

#### 6) Can you change the date format on the Widget Result tab?

Yes, click Settings -> Advanced -> Add, create a Category called 'system' and Param 'dateTimeFormat' with this structure for JSON object:

```
{
    "format": "YYYY/MM/DD HH:mm:ss",
    "custom": {
        "dateTimeResults": "YYYY/MM/DD HH:mm:ss",
        "dateTimeTransaction": "YYYY/MM/DD HH:mm:ss",
        "dateTimePreferences": "YYYY/MM/DD HH:mm:ss",
        "dateTimeTransactionSubTypes": {
            "flow": "YYYY/MM/DD HH:mm:ss",
            "recordings": "YYYY/MM/DD HH:mm:ss",
            "messages": {
                "date": "YYYY/MM/DD",
                "time": "HH:mm:ss"
            },
            "media_reports": "YYYY/MM/DD HH:mm:ss",
            "message_content": "YYYY/MM/DD HH:mm:ss"
        }
    }
}
```

Every field in custom is optional. Change the fields that you're interested in, for the Widget Result tab, change property 'dateTimeResults'. Tokens used for formatting: https://momentjs.com/docs/#/parsing/string-format/
