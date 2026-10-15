<img src="https://user-images.githubusercontent.com/1423657/55069501-8348c400-5084-11e9-9931-fefe0f9874a7.png" width=150>

## HOMER + LuaJIT _(HEPjit)_
One of hottest features in **HOMER 7** is packet manipulation using **LuaJIT** :boom:

This feature delivers unprecedented power and flexibility in processing **HEP** messages to extract custom headers, anonymize or encrypt user data, tag traffic based on custom logic and beyond only limited by your imagination and skills - all performing at native speed! 

This idea has been incubating for a few years and @adubovikov now released it to heplify-server ❤️ 

--------

### *So how does this work?*

### Requirements
* heplify-server w/ LuaJIT ([git](https://github.com/sipcapture/heplify-server/tree/script_lua))
* liblua5.1-dev

### Configuration
A few simple steps are required to enable and configure your first HEP Lua script.

#### Heplify-Server
###### TOML
Enable scripting and configure your entrypoint LuaJIT script, by default inside the `lua` directory. 
```toml
ScriptEnable   = true
ScriptFolder   = "lua/"
```
### ScriptFile
The entrypoint LuaJIT script will be called for each matching HEP packet on its way to the core. The following default example provides a few usage examples for *HEP Type 1* (SIP) to extract a custom header using Regex, Add custom tags and Replace existing message fields, all commented -- inline.

```lua
-- this function will be executed first
function checkRAW()
	
	local protoType = GetHEPProtoType()

	-- Check if we have SIP type 
	if protoType ~= 1 then
		return
	end

	-- original SIP message Payload
	local raw = GetRawMessage()
	-- Logp("DEBUG", "raw", raw)

	-- Create lua table 
	local headers = {}
	headers["X-test"] = "Super TEST Header"

	-- local _, _, name, value = string.find(raw, "(Call-ID:)%s*:%s*(.+)")
	local name, value = raw:match("(CSeq):%s+(.-)\n")

	if name == "CSeq" then
		headers[name] = value
	end

	SetCustomSIPHeader(headers)

	return 

end

-- this function will be executed second
function checkSIP()

	-- get the parsed SIP struct
	local sip = GetSIPStruct()

	-- a struct can be nil so better check it
	if (sip == nil or sip == '') then
		return
	end

	if sip.FromHost == "127.0.0.1" then
		-- Logp("ERROR", "found User-Agent:", sip.UserAgent)
	end

	SetSIPHeader("FromHost", "1.1.1.1")

	-- Full SIP messsage can be changed
	-- SetRawMessage("SIP 2/0")

	return 

end

-- this function will be executed third
function changeNodeIDtoNameFast()

	-- get only nodeID
	local nodeID = GetHEPNodeID()
	if nodeID == 0 then
		SetHEPField("NodeName","TestNode")
	end

	return 

end

-- this function will be executed fourth
function changeNodeIDtoNameSlow()
	-- get the parsed HEP struct
	local hep = GetHEPStruct()

	-- a struct can be nil so better check it
	if (hep == nil or hep == '') then
		return
	end

	if hep.NodeID == 0 then
		hep.NodeName="TestNode"
	end

	return 

end
```

### Add Fields to Mapping
In order to display and search any added field, implement in Mapping for the desired transaction type:
```
{
        "id": "CSeq",
        "type": "string",
        "index": "secondary",
        "name": "CSeq Extraction",
        "form_type": "input",
        "position": 99,
        "sid_type": true,
        "hide": false
    }
```

* Checkout the list of [standard preparsed headers](#standard-header-list)
* Thinking of complex logic and routing? The entrypoint script can also include and use sub-scripts.

### Done
That's it, You're ready to go!

If you find this powerful feature interesting and use it in your setup, please report any bugs and contribute your scripts or changes back to the community to grow a fantastic library of HEPxperiments!

----

### Hot reload lua script
You can reload your lua script without interruption with following command:
```
killall -HUP heplify-server
```

### Standard Function List
The following functions can be used. <br>
```
GetHEPStruct()
GetSIPStruct()
GetHEPProtoType()
GetHEPSrcIP()
GetHEPSrcPort()
GetHEPDstIP()
GetHEPDstPort()
GetHEPTimeSeconds()
GetHEPTimeUseconds()
GetHEPNodeID()
GetRawMessage()
SetRawMessage(value string)
SetCustomSIPHeader(map luatable)
SetHEPField(field string, value string)
SetSIPHeader(header string, value string)
HashString(hashfunc string, text string)
Logp(level string, message string, data interface{})
Print(text string)
```

### Standard Header List
The following headers can be directly manipulated. <br>
For custom extractions, use Regex against the RAW header.
```
"ViaOne"
"FromUser"
"FromHost"
"FromTag"
"ToUser"
"ToHost"
"ToTag"
"CallID"
"XCallID"
"ContactUser"
"ContactHost"
"Authorization.Username"
"UserAgent"
"Server"
"PaiUser"
"PaiHost"
```
