# Description

This role configures [Lido Keys API (KAPI)](https://github.com/lidofinance/lido-keys-api).

The LIDO Keys API offers a convenient way to handle key management tasks for Ethereum validators. It covers operations such as generating, importing, exporting, and signing gossip messages. Please refer to the [official documentation](https://hackmd.io/@lido/S1Li-wXl3) for further details.

## Role Implementation Details

The software runs as a docker container from [`lidofinance/lido-keys-api`](https://hub.docker.com/r/lidofinance/lido-keys-api).

Docker compose file template is in [templates/docker-compose.yml.j2](./templates/docker-compose.yml.j2).

## Environment Variables

Options are configured via environment variables. They are defined at [defaults/main.yml](./defaults/main.yml).

Pay attention that API by default running job for fetching and updating Validators. If you are not planning to use `validators` endpoints, you could disable this job by setting `kapi_validator_registry_enable=false`.

## Self-signed Certificates

To use self-signed certificates you should:

* Make a certificate and save it to the folder.
* Put the path to this folder in a variable called `node_ca_certs_repo_path`.
* If there is a certificate, `ansible` will copy it to a folder under the docker file, but if there isn't, `ansible` will skip this step due to:

    ```yml
    - name: KAPI | Check for cert file
        stat:
        path: '{{ node_ca_certs_repo_path }}'
        register: cert
    ```

## Requirements

1. 2 core CPU
2. 5 GB RAM
   * Keys-API-DB — 500MB
   * Keys-API — 4GB
3. EL Full node
4. CL node for applications like the [LIDO validator ejector](https://github.com/status-im/infra-role-lido-validator-ejector) that use the [validators API](https://hackmd.io/fv8btyNTTOGLZI6LqYyYIg?view#validators). The KAPI currently doesn't work with the Nimbus client. If you use Teku, please use archive mode.
5. The `blockHash` in the metadata of each KAPI module should be equal in order to keep the modules in consistent state.
6. The KAPI will serve data according to the fetched EL state. It will NOT return any state data for arbitrary block numbers. The KAPI is NOT an indexer.
