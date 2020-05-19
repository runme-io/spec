This example demonstrates how to run [a simple HTTP application](https://hub.docker.com/r/hashicorp/http-echo) which already packed to the docker image.

The specification will use the docker image `hashicorp/http-echo:latest` with command `/http-echo -text test` and publish port `5678` in public internet.

To use this example you have to create `.runme` directory in the root of your repository and copy file `config.yaml` there.
