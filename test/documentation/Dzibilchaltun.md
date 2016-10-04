# Dzibilchaltun

> Telegram

## Hardware Requirements

- None

## Software Requirements

```sh
root@board:~# echo "pip install requests future python-telegram-bot"
root@board:~# echo "opkg install python-dev"
```

## Setup

```sh
root@edison:~# curl https://raw.githubusercontent.com/TheIoTLearningInitiative/CodeLabs/master/Dzibilchaltun/setup.sh -o - | sh
```

## Code

```sh
root@edison:~/CodeLabs/Dzibilchaltun# nano main.c
```

```c
#!/usr/bin/python

import atexit
import ConfigParser
import signal
import sys
import time

import pyupm_grove as grove
import pyupm_grovespeaker as upmGrovespeaker
import pyupm_i2clcd as lcd

from telegram.ext import Updater, CommandHandler, MessageHandler, Filters

credentials = ConfigParser.ConfigParser()
credentialsfile = "credentials.config"
credentials.read(credentialsfile)

button = grove.GroveButton(8)
display = lcd.Jhd1313m1(0, 0x3E, 0x62)
light = grove.GroveLight(0)
relay = grove.GroveRelay(2)

def functionLight(bot, update):
    luxes = light.value()
    bot.sendMessage(update.message.chat_id, text='Light ' + str(luxes))

def functionMessage(bot, update):
    bot.sendMessage(update.message.chat_id, text=message)

def functionRelay(bot, update):
    relay.on()
    time.sleep(2)
    relay.off()
    bot.sendMessage(update.message.chat_id, text='Relay Used!')

def functionEcho(bot, update):
    bot.sendMessage(update.message.chat_id, text=update.message.text)

def SIGINTHandler(signum, frame):
	raise SystemExit

def exitHandler():
	print "Exiting"
	sys.exit(0)

atexit.register(exitHandler)
signal.signal(signal.SIGINT, SIGINTHandler)

if __name__ == '__main__':

    credential = credentials.get("telegram", "token")
    updater = Updater(credential)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("light", functionLight))
    dp.add_handler(CommandHandler("message", functionMessage))
    dp.add_handler(CommandHandler("relay", functionRelay))
    dp.add_handler(MessageHandler([Filters.text], functionEcho))

    updater.start_polling()

    message = "Hi! I'm Dzibilchaltun!"

    while True:

        luxes = light.value()
        luxes = int(luxes)    
        display.setColor(luxes, luxes, luxes)
        display.clear()

        if button.value() is 1:
            display.setColor(255, 0, 0)
            display.setCursor(0,0)
            display.write(str(message))
            relay.on()
            time.sleep(1)
            relay.off()

    updater.idle()
```

## Execution

```sh
root@edison:~/CodeLabs/Dzibilchaltun# 
/usr/lib/python2.7/site-packages/urllib3/util/ssl_.py:318: SNIMissingWarning: An HTTPS request has been made, but the SNI (Subject Name Indication) extension to TLS is not available on this platform. This may cause the server to present an incorrect TLS certificate, which can cause validation failures. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#snimissingwarning.
  SNIMissingWarning
/usr/lib/python2.7/site-packages/urllib3/util/ssl_.py:122: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/security.html#insecureplatformwarning.
  InsecurePlatformWarning
  
```
