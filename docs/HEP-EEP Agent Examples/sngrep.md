![](https://camo.githubusercontent.com/fbfe5cb64b1df2bf64f0d0616a24c36bfa17220b/687474703a2f2f69726f6e7465632e6769746875622e696f2f736e677265702f696d616765732f6c6f676f2e706e67)

# sngrep
Irontec's awesome *sngrep* 1.x+ introduces command line option (-H) and settings (eep.send) to send capture data in HEP/EEP to Homer and to run headless as a capture agent:

* ```-H or --eep-send```: Send captured data to other Homer (udp:10.10.10.10:9060)
* ```-N or --no-interface```: Don't display sngrep interface, just capture
* ```-q or --quiet```: Don't print captured dialogs in no interface mode

### Example: SIP
Mirror all SIP packets from all devices with src||dst port 5060 to Homer
<pre>
sngrep port 5060 -H udp:10.10.10.10:9060 --no-interface -q
</pre>

### Example: TLS
Mirror all SIP/TLS packets from all devices with src||dst portrange 5060-5061 to Homer
<pre>
sngrep portrange 5060-5061 -k ./privkey.pem -H udp:10.10.10.10:9060 --no-interface -q
</pre>

<img src="http://i.imgur.com/dId782W.png" />

For further information please visit the [sngrep](https://github.com/irontec/sngrep/wiki#how-to-use) wiki.
