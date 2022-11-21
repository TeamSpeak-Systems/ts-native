# TS-native

This repository contains a teamspeak 5 server build for Linux amd64 which does not require docker or docker compose to run.

* https://github.com/TeamSpeak-Systems/ts-native.git

**Note: The concept of ts-native is to be used in combination with ts-services. The administration burden in using ts-native is high as rollouts of ts-native updates have to be closely coordinated with ts-services updates.**

We stongly advice to only use ts-services and then all services are administrated from docker-compose.yaml and your configuration overlay.

# ts-services

* https://github.com/TeamSpeak-Systems/ts-services.git
