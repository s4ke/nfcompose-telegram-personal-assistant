nfcompose-telegram-personal-assistant
===========================

A Telegram Personal Assistant built with [NF Compose](https://github.com/neuroforgede/nfcompose) + [Node-RED](https://github.com/node-red/node-red).

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/c86e0f1b-ffa9-4898-9f74-48ad2c1223eb)

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
curl -L https://raw.githubusercontent.com/s4ke/nfcompose-telegram-personal-assistant/main/dataseries.json
compose_cli diff dataseries --base-type compose http://skipper.localhost:8000 ./dataseries.json --base-compose-user admin --base-compose-password admin | compose_cli apply dataseries-diff http://skipper.localhost:8000 --compose-user admin --compose-password admin
```

## Create Telegram Bot

You can create a new bot by chatting with `@BotFather` on Telegram. BotFather will guide you through the creation process.

## Setup Node-RED Project

### Manual

Install the following nodes:

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/9fb0bfdf-8348-423c-ae51-fd85e981ca51)

Access Node-RED at `http://skipper.localhost:8000/api/flow/system/engine/access/`, create a new project and import the flow from `https://github.com/s4ke/nfcompose-telegram-personal-assistant/blob/main/flows.json`

### Clone this repo in the UI

Alternatively, you can also clone this repository via the Node-RED git integration.

## Configure Node-RED Flows

Then, configure the telegram config. You can access it in the config section:

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/c83cdb24-ff75-43f4-94d2-b28a2b366f7a)

Double click on `Personal Assistant`. This will open the configuration for the telegram bot node:

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/2ed4e6fc-cb77-4183-b6da-5adff8a1fb8d)

Enter the token you got from BotFather on Telegram and press `Update`.

## Adapt authorized chat id

Open a chat with your bot and send a message with `/chatId` to get the chat id:

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/14879c3f-ceb2-467d-a1da-83ced6b439dd)

Using this chat id, we can run the following to configure the chat id in the telegram bot:

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

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/c9be0da4-dc58-4643-a465-ecec15758eb5)

The bot will now start reminding you of the events in the `.ics` file.

![grafik](https://github.com/s4ke/nfcompose-telegram-personal-assistant/assets/719760/9dc5b0ba-3c89-4d82-88e2-b241a4c01502)

