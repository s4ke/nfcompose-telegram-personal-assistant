telegram-personal-assistant
===========================

A Telegram Personal Assistant built with NF Compose + Node-RED.

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/ed6acfc8-d9c0-4ac5-a3e6-ed1e9ba72d49)

Features:

- [x] 💯 Reminds you of Birthdays
- [x] Nothing else for now

# Installation

## Prerequisites

- Docker >= 20.17
- Docker Compose V2
- Python >= 3.8

## Setup NF Compose + Node RED

First install the NF Compose Client library:

```bash
pip3 install https://github.com/neuroforgede/nfcompose/releases/download/release%2F2.1.0-beta/compose_client-2.1.0-beta.tar.gz
```

Setup NF Compose together with Node-RED by running the local production deployment in an empty folder:

```bash
curl -L https://github.com/neuroforgede/nfcompose/releases/download/release%2F2.1.0-beta/deploy-production-docker-compose-2.1.0-beta.zip | busybox unzip -
bash deploy.sh
```

Note: For a production setup, check out the configuration section in the docker-compose file and adapt all secrets accordingly.

You should now be able to visit the NF Compose instance at http://skipper.localhost:8000/api/ and login with user `admin` and password `admin` (or your custom credentials if you have changed them). 

## Migrate NF Compose Dataseries

Next, we have to prepare the NF Compose dataseries we need for the assistant:

```bash
curl -L https://raw.githubusercontent.com/s4ke/telegram-personal-assistant/main/dataseries.json
compose_cli diff dataseries --base-type compose http://skipper.localhost:8000 ./dataseries.json --base-compose-user admin --base-compose-password admin | compose_cli apply dataseries-diff http://skipper.localhost:8000 --compose-user admin --compose-password admin
```

## Create Telegram Bot

You can create a new bot by chatting with `@BotFather` on Telegram. BotFather will guide you through the creation process.

## Setup flows in Node-RED

Access Node-RED at `http://skipper.localhost:8000/api/flow/system/engine/access/`, create a new project and import the flow from `https://github.com/s4ke/telegram-personal-assistant/blob/main/flows.json`

Then, configure the telegram config. You can access it in the config section:

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/da136752-3882-4f86-9aa6-6fde56828b8c)

Double click on `Personal Assistant`. This will open the configuration for the telegram bot node:

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/9cb2255d-eaf7-4d61-b680-bbdf0afa2e2c)

Enter the token you got from BotFather on Telegram and press `Update`.

## Adapt authorized chat id

Open a chat with your bot and send a message with `/chatId` to get the chat id:

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/0ff0a074-6c3f-4c30-8826-87a493770b2f)

Using this chat id, we can run the following:

```bash
# replace with your chat id
export CHAT_ID="123123"
compose_cli push datapoints http://skipper.localhost:8000 config --compose-user admin --compose-password admin <<EOF
[{
    "external_id": "config",
    "payload": {
        "chat_id": "${CHAT_ID}"
    }
}]
EOF
```

## Upload ics file

Finally, we have to only tell the Telegram bot about our calendar we want it to remind us of. To do so, we simply send it a single `.ics` file:

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/9f8cf571-9c89-44d0-94b4-8a8359f67761)

The bot will now start reminding you of the events in the `.ics` file.

![grafik](https://github.com/s4ke/telegram-personal-assistant/assets/719760/ed6acfc8-d9c0-4ac5-a3e6-ed1e9ba72d49)

