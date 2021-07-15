# WIP: SugarFunge Graph

Experiment by modifying `schema.graphql` and the mappings in the `mappings` folder, defined in `manifest.yml`.

## 1. Bootstrap

Run

```bash
yarn && yarn bootstrap
```

and generate the model files as defined in `schema.graphql`, create the database and run all the necessary migrations in one shot.

NB! Don't use in production, as it will delete all the existing records.


## 2. Generate Types for events and extrinsics

A separate tool Hydra Typegen can be used for generating Typescript classes for the event handlers (the _mappings_).  
Run

```bash
yarn typegen
```
to run the [typegen](https://github.com/Joystream/hydra/tree/master/packages/hydra-typegen/README.md) for events and extrinsics defined in `manifest.yml` (it fetches the metadata from an RPC endpoint and blockhash defined there). 


## 3. Build Mappings

Mappings is a separated TypeScript module created in the mappings folder. The handlers exported by the module should match the ones defined in `manifest.yml` in the mappings section. Once the necessary files are generated, build it with

```bash
yarn workspace sample-mappings install
yarn mappings:build
```

## 4. Run the processor and the GraphQL server

Then run the processor:

```bash
yarn processor:start
```

Afterwards start the GraphQL server in a separate terminal (opens a GraphQL playground at localhost by default):

```bash
yarn query-node:start:dev
```

## 5. Locally hosted indexer

The Hydra Indexer endpoint used by Hydra processor is defined as environment variable `INDEXER_ENDPOINT_URL` sourced from `.env`. There are publicly available Hydra indexers for Polkadot and Subsocial. For other chains, a self-hosted indexer should be used.

The simplest way to run an indexer locally is to run `docker-compose-indexer.yml` with `docker-compose`. The following environment variables must be provided:

- Database connection settings: DB_NAME, DB_HOST, DB_PORT, DB_USER, DB_PASS
- Chain RPC endpoint: WS_PROVIDER_ENDPOINT_URI
- If non-standard types are being used by the Substrate runtime, map type definitions in the json format as an external volume

Follow the links for more information about the [indexer](https://github.com/Joystream/hydra/tree/master/packages/hydra-indexer/README.md) service and [indexer-api-gateway](https://github.com/Joystream/hydra/tree/master/packages/hydra-indexer-gateway/README.md).

## Running in Docker

Docker files are located in `./docker`. First, build the builder image:

```bash
$ docker build . -f docker/Dockerfile.builder -t builder
```

Images for the GraphQL query node and the processor depend on the `builder` image which is now available. 
Build with

```bash
$ docker build . -f docker/Dockerfile.query-node -t query-node:latest
$ docker build . -f docker/Dockerfile.processor -t processor:latest
```

In order to run the docker-compose stack, we need to create the schema and run the database migrations. 

```bash
$ docker-compose up -d db 
$ yarn docker:db:migrate
```

The last command runs `yarn db:bootstrap` in the `builder` image. A similar setup strategy may be used for Kubernetes (with `builder` as a starter container).

Now everything is ready:

```bash
$ docker-compose up
```
