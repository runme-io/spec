This example demonstrates how to run multi-container application on an example [Rocket.Chat](https://github.com/RocketChat/Rocket.Chat).

The specification will run two containers: MongoDB and Rocket.Chat in common network and then publish port `3000` of Rocket.Chat container in public internet.

To use this example you have to create `.runme` directory in the root of your repository and copy file `config.yaml` there.
