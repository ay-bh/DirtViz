# DirtViz

DirtViz is a project to visualize data collected from sensors deployed in sensor networks. The project involves developing web based plotting scripts to create a fully-fledged DataViz tool tailored to the data collected from embedded systems sensor networks.

## Integrations

Dirtviz is only a frontend and it needs some way of importing data into the backend database. Currently there are csv importers, chirpstack HTTP integration, and postgresql access.

### Postgresql

By default the `postgresql` instance is exposed on the host machine that can be connected with the following options
```
ip: localhost
port: 5432
user: dirtviz
password: password
table: dirtviz
```

### CSV Importers

There are csv importers that can be used to populate the database. Python utilities currently exist to import RocketLogger and TEROS data. These are available as modules under dirtviz. More information on used can be found by running the modules with the `--help` flag. It is recomended to setup a virtual enviroment to install the required packages listed in `requirements.txt`.

```
python -m dirtviz.db.utils.import_rl_csv
python -m dirtviz.db.utils.import_teros_csv
```

Before running the moduels the `DB_URL` must be set as a enviorment variable. By default you can set it with
```
export DB_URL=postgresql://dirtviz:password@postgresql/dirtviz
```

### HTTP

For Chirpstack HTTP integration, the docker compose command should be run with the HTTP overwrite. It sets up the HTTP server to accept connections from the chirpstack application server and connects to the chirpstack application server network. This integration is intedned to be used with the modified compose file in the [jlab-sensing/chirpstack-docker](https://github.com/jlab-sensing/chirpstack-docker) repo. Use the following command to startup Dirtviz with HTTP integration
```
docker compose -f docker-compose.yml -f docker-compose.http.yml up -d
```

> If you get the following error `network chirpstack-external declared as external, but could not be found`, you have not started up the chirpstack server.

The chirpstack application server HTTP integration should be setup with
```
hostname: http://http-integration:8090/
```


### Setting up a Development Environment

The DirtViz application is designed to be run from docker. There are multiple container that all need to work together to the end user website up. Luckily for you all of it is defined within the `docker-compose.yml` file. Run the following command to start all the containers.

```
docker compose up -d
```

When adding functionality to the website there is a need to have sample data so that the graphs displayed are not blank. There is data included in the repo and can be imported using the `import_example_data.py` script. It is recommended to run the script from outside the container using the commands listed below to setup a virtual environment, install the necessary packages, and import the data. Due to the way docker mounts function, it is far quicker to import the data over a TCP connection to the postgresql container rather than mounting folder containing the data within the container. The other option would be to add the data to the dirtviz image, but that would increase the image size considerably. For now this is the best option as this functionality is not required in a production environment.

```
export DB_URL=postgresql://dirtviz:password@postgresql/dirtviz
`python3 -m venv .venv
source .venv/bin/activate
./import_example_data.py
```

To clear all the tables in the database run the following command. Note that the `id` fields are not reset and new objects might not start at `id=1`.

```
docker compose exec postgresql psql -U dirtviz -c "TRUNCATE TABLE cell, power_data, logger, teros_data CASCADE;"
```

If you want the database to be recreated the following can be run to delete postgresql data

```
docker volume rm dirtviz_postgresqldata
```
