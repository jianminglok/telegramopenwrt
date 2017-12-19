# Telegram Scripts for OpenWrt

This is a set of scripts with a plugins API written in bash, you can use it to manage and get informations on OpenWRT Routers.

### How its works ?

First of all, clone the repo (the unifi branch) into your router's /root directory. 
In case you're wondering how to install GitHub on your router, run:
```opkg update
   opkg install git git-http ca-bundle
```

and to clone this repo:

```
cd /root
git clone https://github.com/jianminglok/telegramopenwrt.git -b unifi ./
```

Next, Create a bot for yourself! Refer to guides on Google etc. (Hint: It's within the Telegram app)
https://core.telegram.org/bots/api#authorizing-your-bot

With the bot created, you need to replace "[PUT YOUR BOT KEY HERE]" in variables file with your bot key.

Second, you need to send an initial message (not running the commands) to your bot in Telegram App.
After you send the message, run command below on your OpenWRT router in SSH client of your choice:

``` 
curl -s -k -X GET https://api.telegram.org/bot<YOUR BOT ID>/getUpdates | grep -oE "\"id\":[[:digit:]]+" | head -n1 | awk -F : '{print $2}'
```

Get the number and replace "[PUT ID OF THE CHAT THAT YOU START WITH BOT]" in the variables file.
Also replace "[PUT YOUR MODEL HERE]" in the same file with your router's name

Finally,

#### init.d directory

Contains the necessary files for the scripts to be started at the router boot, just move them to the /etc/init.d/ of the router and run:
```
/etc/init.d/lanports enable
/etc/init.d/telegram_bot enable
/etc/init.d/ip_monitor enable
/etc/init.d/hosts_scan enable
```

Hint: Just disable any file other than /etc/init.d/telegram_bot you don't like by not copying them to /etc/init.d or running `/etc/init.d/${filename} disable` to disable them, it's up to you.

And then:

```
/etc/init.d/lanports start
/etc/init.d/telegram_bot start
/etc/init.d/ip_monitor start
/etc/init.d/hosts_scan start
```

And you're good to go!

### Directory structure

```
+--- LICENSE
+--- README.md
+--- init.d/
	+--- lanports
	+--- telegram_bot
+--- lanports
+--- plugins/
	+--- block
	+--- fwlist
	+--- gethosts
	+--- getip
	+--- help/
		+--- block
		+--- fwlist
		+--- gethosts
		+--- getip
		+--- getwifi
		+--- start
		+--- unblock
		+--- wifilist
	+--- start
	+--- unblock
	+--- wifilist
+--- telebot
+--- telegram\_bot
```
#### lanports file

This file reads the router logs with the logread -f command, and sends messages via the bot if a router port is turned off / on or the router delivers an IP address via DHCP (when a client has successfully connected and obtained IP Address from DHCP).

###  ip_monitor file

Contains a script which loops continuously to monitor your UniFi connection for downtime, timestamp included in case of long downtimes so you can submit this to TM as proof of downtime. Also provides info through the bot about newly obtained IP addresses during initial startup or reboot.

#### plugins directory

This is the main directory, it contains all the commands that the telegram bot can execute.

Here are some pre-built commands, which are:

 * Block: Creates a REJECT rule for a host (by macaddr) to make NAT for the internet.
 * Fwlist: Lists all firewall rules.
 * Gethosts: Lists the hosts that are in dhcp.leases, if the argument is a host, will list only the host found by regex.
 * Getip: WAN IP.
 * Start: Creates command help.
 * Unblock: Removes a firewall rule created by Telegram, without an argument, all rules created by Telegram are deleted.
 * Wifilist: List all devices connected to WiFi.
 
There's more to explore in the plugin directory, you can also create your own ones.

#### help directory inside plugins

This is the directory containing the help files, with the same name as the command, to be displayed by the start command.

#### telebot file

This file sends the telegram bot messages generated by lanports


#### telegram_bot

The telegram_bot script is a loop that receives the updates every second and checks to see if there is a command to execute. If there is a command, the script checks to see if there is a file with the same name as the command inside the plugins directory and runs it, if it exists. The output of the executed script is sent as a response message from the command.
Inside the plugins directory, there is the special command "start", which returns a message with the commands and a brief help on each command.
For this command to work properly, you need to create a file inside the plugins / help directory, with the name equal to the command.


