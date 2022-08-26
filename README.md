# TeamSpeak Server Provisioning

go-serverprovisioning implements a REST interface to manage https://gitlab.int.teamspeak.com/teamspeak/teamspeak_server instances.

    git clone https://gitlab.int.teamspeak.com/teamspeak/go-serverprovisioning

## Server side implementation of go-serverprovisioning

go-serverprovisioning offers a REST interface, basically a JSON-API, which consists of several routes.
### REST admin interface 
One can query it like this:
```sh
# Requires admin api key
# Add User
    curl \
     -X POST \
     -H 'API_KEY: ee3658e4150adf9782a5e41812436c7a' \
     -d '{
        "operation": "add",
        "payload": {
            "username": "myts"
        }
     }' \
    localhost:5000/user
    
# Add Location
    curl -i \
    -X POST \
    -H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
    -d '{
        "operation": "add",
        "payload": {
            "locationname": "ger" 
        }
    }' \
    localhost:5000/location
    
# Add TS Server instance to locationid
    curl -i \
    -X POST \
    -H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
    -d '{ 
        "operation": "add", 
        "payload": {
            "locationid":1,
            "instancename":"dev-ts-test",
            "queryhost":"dev-ts-test.teamspeak.com",
            "queryport":10080,
            "queryapikey":"eqavFsx5lxJvoEOQNN7AqAhE8i7tvvnb"
        }
    }' \
    localhost:5000/instance
```
### REST interface /server
The server API can be reached at e.g. localhost:5000/server. It is required to use POST and to provide the API_KEY in the header. Currently it its possible to perform the following operations: create, status, stop, start, delete
```sh
# Create new server
curl \
-H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
-d '{ 
    "operation": "create", 
    "payload": {
        "virtualserver_name":"testserver1",
        "virtualserver_welcomemessage":"hello",
        "virtualserver_maxclients":12,
        "virtualserver_password":"testpass"
        }
    }' \
-X POST localhost:5000/server
# Response:
{
   "data":{
      "virtualserver_uuid":"01bda880-dcc0-5973-a135-5f4656898d0b",
      "virtualserver_hostname":"dev-ts-test.teamspeak.com",
      "virtualserver_port":10024,
      "virtualserver_token":"fsdf534sayxcyw4qesfwrwexcvxcvxvc"
   }
}

##################
# Request status of vserver by UUID
curl -i \
-H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
-d '{
   "operation":"status",
   "payload":{
      "virtualserver_uuid":"872e2d0a-882f-575e-92ee-2b6f49b828ec"
   }
}' \
-X POST localhost:5000/server
# Response:
{
   "body":{
      "virtualserver_uuid":"872e2d0a-882f-575e-92ee-2b6f49b828ec",
      "virtualserver_status":"online",
      "virtualserver_uptime":63,
      "virtualserver_clientsonline":0,
      "virtualserver_total_ping":0
   }
}

##################
# Request start / stop / delete of vserver by UUID
curl -i \
-H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
-d '{
   "operation":"stop",
   "payload":{
      "virtualserver_uuid":"f386f67a-88a7-58c9-b739-617d30af3303"
   }
}' \
-X POST localhost:5000/server
# Response
{
  "body": {
    "virtualserver_uuid": "f386f67a-88a7-58c9-b739-617d30af3303",
    "virtualserver_status": "offline"
  }
}
# Delete
curl -i \
-H "API_KEY: ee3658e4150adf9782a5e41812436c7a" \
-d '{
   "operation":"delete",
   "payload":{
      "virtualserver_uuid":"872e2d0a-882f-575e-92ee-2b6f49b828ec"
   }
}' \
-X POST localhost:5000/server

```

## Client side implementation of go-serverprovisioning

The go-serverprovisioning client implementation calls the teamspeak_server REST API using:
* https://gitlab.int.teamspeak.com/teamspeak/teamspeak_documents/-/blob/master/server/webquery.md

## Run in docker compose

```console
docker compose build
docker compose up -d
```

## Client tests from Windows Powershell

```
curl -X POST -H 'API_KEY: ee3658e4150adf9782a5e41812436c7a' -d  '{\"operation\": \"add\", \"payload\": {\"username\": \"foobar5001\"}}' localhost:5000/user -i

curl -i -H "API_KEY: ee3658e4150adf9782a5e41812436c7a" -d '{"operation": "add", "payload": {"username": "achimcoolio"}}' -X POST localhost:5000/user

HTTP/1.1 200 OK
Content-Type: application/json
Date: Fri, 22 Jul 2022 16:38:48 GMT
Content-Length: 79

{"data":{"username":"achim","apikey":"GhUFOeIdNKlluDURjk7qBFmt0oBlua1v"}}
```

# Database

## Flush the contents of the postgresql state

If you want to recreate the database, basically delete the volume and then the database gets created automatically:

If go-serverprovisioning is still running:

    docker compose down

Let's see the volume, should be something like this:

    docker volumes ls
    local     go-serverprovisioning_postgresql

Then kill it with fire:

    docker volume rm --force go-serverprovisioning_postgresql
    go-serverprovisioning_postgresql

Afterwards basically start working with it as usual, i.e.:

    docker compose up -d

Which should create a completely new database. Introspect the logs with:

    docker container logs  -f -n 200  go-serverprovisioning_postgresql_1

Note: Normally this should not be needed.

## Accessing the database

    docker compose up -d
    docker exec -it go-serverprovisioning_postgresql_1 sh -c 'psql -U postgres -d serverprovisioning'
    \dt
    \l
    select * from xxx;

## Use pgadmin to access the database

You need to enable it in the docker-compose.yaml and then:

    docker compose up -d

pgadmin might be nice to build bigger DB requests. In general, I prefer psql for its simplicity as pgadmin's session
frequently break with the docker compose restarts from the goland IDE.
