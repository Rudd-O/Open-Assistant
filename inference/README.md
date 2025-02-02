# OpenAssistant Inference

Preliminary implementation of the inference engine for OpenAssistant. This is
strictly for local development, although you might find limited success for your
self-hosting OA plan. There is no warranty that this will not change in the
future — in fact, expect it to change.

## Development Variant 1 (docker compose)

The services of the inference stack are prefixed with "inference-" in the
[unified compose descriptor](../docker-compose.yaml). <br/> Prior to building
those, please ensure that you have Docker's new
[BuildKit](https://docs.docker.com/build/buildkit/) backend enabled. See the
[FAQ](https://projects.laion.ai/Open-Assistant/docs/faq#enable-dockers-buildkit-backend)
for more info.

To build the services, run:

```shell
docker compose --profile inference build
```

Spin up the stack:

```shell
docker compose --profile inference up -d
```

Tail the logs:

```shell
# cd /root/of/this/repository
docker compose logs -f    \
    inference-server      \
    inference-worker
```

> **Note:** The compose file contains the bind mounts enabling you to develop on
> the modules of the inference stack, and the `oasst-shared` package, without
> rebuilding.

> **Note:** You can change the model by editing variable `MODEL_CONFIG_NAME` in
> the `docker-compose.yaml` file. Valid model names can be found in
> [model_configs.py](../oasst-shared/oasst_shared/model_configs.py).

> **Note:** You can spin up any number of workers by adjusting the number of
> replicas of the `inference-worker` service (in the `docker-compose.yaml` file)
> to your liking.

> **Note:** Please wait for the `open-assistant-inference-worker` services to
> finish downloading models before continuing to the next step.

Spin up the text-client, and start chatting:

```shell
# cd /root/of/this/repository
cd inference/text-client
pip3 install --user -r requirements.txt
PYTHONPATH=$PWD/../../oasst-shared python3 __main__.py
# You'll soon see a `User:` prompt, where you can type your prompts.
```

## Development Variant 2 (tmux terminal multiplexing)

Ensure you have `tmux` installed on you machine and the following packages
installed into the Python environment;

- `uvicorn`
- `worker/requirements.txt`
- `server/requirements.txt`
- `text-client/requirements.txt`
- `oasst_shared`

You can run development setup to start the full development setup.

```bash
cd inference
./full-dev-setup.sh
```

> Make sure to wait until the 2nd terminal is ready and says
> `{"message":"Connected"}` before entering input into the last terminal.

## Development Variant 3 (you'll need multiple terminals)

Run a postgres container:

```bash
docker run --rm -it -p 5432:5432 -e POSTGRES_PASSWORD=postgres --name postgres postgres
```

Run a redis container (or use the one of the general docker compose file):

```bash
docker run --rm -it -p 6379:6379 --name redis redis
```

Run the inference server:

```bash
cd server
pip install -r requirements.txt
DEBUG_API_KEYS='0000,0001,0002' uvicorn main:app --reload
```

Run one (or more) workers:

```bash
cd worker
pip install -r requirements.txt
API_KEY=0000 python __main__.py

# to add another worker, simply run
API_KEY=0001 python __main__.py
```

For the worker, you'll also want to have the text-generation-inference server
running:

```bash
docker run --rm -it -p 8001:80 -e MODEL_ID=distilgpt2 \
    -v $HOME/.cache/huggingface:/root/.cache/huggingface \
    --name text-generation-inference ghcr.io/yk/text-generation-inference
```

Run the text client:

```bash
cd text-client
pip install -r requirements.txt
python __main__.py
```

## Distributed Testing

We run distributed load tests using the
[`locust`](https://github.com/locustio/locust) Python package.

```bash
pip install locust
cd tests/locust
locust
```

Navigate to http://0.0.0.0:8089/ to view the locust UI.
