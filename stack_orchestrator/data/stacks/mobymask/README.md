# MobyMask

The MobyMask watcher is a Laconic Network component that provides efficient access to MobyMask contract data from Ethereum, along with evidence allowing users to verify the correctness of that data. The watcher source code is available in [this repository](https://github.com/cerc-io/mobymask-watcher-ts) and a developer-oriented Docker Compose setup for the watcher can be found [here](https://github.com/cerc-io/mobymask-watcher). The watcher can be deployed automatically using the Laconic Stack Orchestrator tool as detailed below:

## Deploy the MobyMask Watcher

The instructions below show how to deploy a MobyMask watcher using laconic-stack-orchestrator (the installation of which is covered [here](https://github.com/cerc-io/stack-orchestrator#install)).

This deployment expects that ipld-eth-server's endpoints are available on the local machine at http://ipld-eth-server.example.com:8083/graphql and http://ipld-eth-server.example.com:8082. More advanced configurations are supported by modifying the watcher's [config file](../../config/watcher-mobymask/mobymask-watcher.toml).

## Clone required repositories

```bash
$ laconic-so --stack mobymask setup-repositories
```

## Build the watcher container

```bash
$ laconic-so --stack mobymask build-containers
```

This should create a container with tag `cerc/watcher-mobymask` in the local image registry.

## Create a deployment

```bash
$ laconic-so --stack mobymask deploy init --output mobymask-spec.yml
$ laconic-so deploy create --spec-file mobymask-spec.yml --deployment-dir mobymask-deployment
```

External `ipld-eth-server` endpoint can be set in watcher config file in the deployment directory:
```
mobymask-deployment/config/watcher-mobymask/mobymask-watcher.toml
```

## Start the stack

First the watcher database has to be initialized. Start only the mobymask-watcher-db service:

```bash
$ laconic-so deployment --dir mobymask-deployment start mobymask-watcher-db
```

Next find the container's id using `docker ps` then run the following command to initialize the database:
```bash
$ docker exec -i <mobymask-watcher-db-container> psql -U vdbm mobymask-watcher < mobymask-deployment/config/watcher-mobymask/mobymask-watcher-db.sql
```

Finally start the remaining containers:

```bash
$ laconic-so deployment --dir mobymask-deployment start
```

Correct operation should be verified by following the instructions [here](https://github.com/cerc-io/mobymask-watcher/tree/main/mainnet-watcher-only#run), checking GraphQL queries return valid results in the watcher's [playground](http://127.0.0.1:3001/graphql).

## Clean up

Stop all the services running in background:

```bash
$ laconic-so deployment --dir mobymask-deployment stop
```

## Data volumes

Container data volumes are bind-mounted to specified paths in the host filesystem.
The default setup (generated by `laconic-so deploy init`) places the volumes in the `./data` subdirectory of the deployment directory:
```
$ cat mobymask-spec.yml
stack: mobymask
ports:
  mobymask-watcher-db:
   - 0.0.0.0:15432:5432
  mobymask-watcher-job-runner:
   - 0.0.0.0:9000:9000
  mobymask-watcher-server:
   - 0.0.0.0:3001:3001
   - 0.0.0.0:9001:9001
volumes:
  mobymask_watcher_db_data: ./data/mobymask_watcher_db_data
```

The directory can be changed before `laconic-so deploy create`