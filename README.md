# Runme specification

- [Runme specification](#runme-specification)
  * [Examples](#examples)
  * [Options](#options)
    + [version](#version)
      - [Example](#example)
    + [publish](#publish)
      - [Example](#example-1)
    + [services](#services)
      - [Example](#example-2)
    + [image](#image)
      - [Example](#example-3)
    + [build](#build)
      - [Example](#example-4)
    + [command](#command)
      - [Examples](#examples-)
    + [environment](#environment)
      - [Example](#example-5)
    + [port](#port)
      - [Example](#example-6)
    + [volumes](#volumes)
      - [Example](#example-7)

Runme specification is a YAML config file to describe how to build and run an application.

Specification requirements:
1. the specification must be placed in the directory `.runme` and must has to filename `config.yaml`;
2. options `version`, `publish`, `services` are required;
3. the specification must contain at least one service and this service must be set as main in `publish` option;
4. the main service must contain `ports` section with one container port and one public port;
5. the main service must have section `image` or section `build` or both sections;
6. non-main service cannot has `ports` section or `build` section.

Specification limits:
1. nowadays Runme support only version `1.0` of specification;
2. Runme can launch up to 5 services only

All the specification options also have own requirements and limit please take a look description below.

## Examples
Example of simple specification when your application already has built docker image and runme can use it:

```yaml
version: 1.0
publish: app
services:
  app:
    # your_image:some_tag - it's prepared docker image in public registry
    image: your_image:some_tag
    ports:
    # 3000 - it's port which application will listen
    # inside container for public connections
    - container: 3000
      port: 80
```

In this specification `your_image:some_tag` - it's prepared docker imaged placed in **public** docker registry. Port `3000` - it's port which application will expose for connections. This port will be mapped to the public port `80` (nowdays runme support only this port). The fields `version`, `publish`, `services` are required in any type of specifications. Fields `ports` and `image` (or `built`) required for the main service.

Example of simple specification when your application doesn't have built docker image and runme should prepare it:

```yaml
version: 1.0
publish: app
services:
  app:
    build:
      type: dockerfile
      # path_to_dockerfile - it's path where Runme can find
      # Dockerfile to prepare docker image
      config: path_to_dockerfile
    ports:
    # 3000 - it's port which application will listen
    # inside container for public connections
    - container: 3000
      port: 80
```

In this specification `path_to_dockerfile` - it's path where runme can find Dockerfile inside the repository to build docker image with the application. All other fileds the same as in previous exmaple.

Example of specification to run real project Rocket.Chat in Runme:

```yaml
version: 1.0
publish: rocketchat
services:
  mongo:
    image: bitnami/mongodb:4.0
    environment:
      MONGODB_REPLICA_SET_MODE: primary
      MONGODB_REPLICA_SET_NAME: rs0
      MONGODB_ROOT_PASSWORD: password
      MONGODB_REPLICA_SET_KEY: replicasetkey123
    volumes:
    - name: data
      mount: /data/db

  rocketchat:
    image: rocketchat/rocket.chat:latest
    build:
      type: dockerfile
      config: .runme/Dockerfile
    command: ["bash", "-c", "set -x; for i in {1..30}; do node main.js && s=$$? && break || s=$$?; echo \"Tried $$i times. Waiting 5 secs...\"; sleep 2; done; (exit $$s)"]
    environment:
      PORT: 3000
      ROOT_URL: http://localhost:3000
      MONGO_URL: mongodb://root:password@mongo:27017/rocketchat?replicaSet=rs0&authSource=admin
      MONGO_OPLOG_URL: mongodb://root:password@mongo:27017/local?replicaSet=rs0&authSource=admin
      MAIL_URL: smtp://smtp.email
    ports:
    - container: 3000
      public: 80
    volumes:
    - name: data
      mount: /data/db
```

This specification has second service - MongoDB to run the main service (Rocker.Chat) with database. Also this psecification has volumes and environment variables for the services.

## Options
### version
`version` - it is **required option** to set version of the specification. Now Runme support only one version - `1.0`.

#### Example
```yaml
version: 1.0
```

### publish
`publish` - it is **required option** to set a name of the main service form the list of services (option `services`).

#### Example
```yaml
version: 1.0
publish: app_name
services:
  ...
```
Where `app_name` is name of service from `services` section which should be provided as the main service.

### services
`services` - it is **required section** of the specification, sets the list of service that will be run. This secton should contain as least one service and this service should be set as main service in `publish` option.

#### Example
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    ...

  some_second_app:
    ...
```

### image
`image` - it is **required option** for non-main services but is not required for the main one. If the main service does not contain `image` option it must contain `build` option. Image option sets docker image which will be used to run service. `build` option will be ignored if this option is not empty;

#### Example
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    ...
```

### build
`build` - this section **allowed only for the main service**, and sets the path to Dockerfile to prepare docker image. This section contains two options: `type` and `config`. For `type` option Runme supports only `dockerfile` value so far. `config` option must contain a relative path from the root of repository to find Dockerfile. Runme will ignore `build` option if will find `image` section.

#### Example
specification with build section:
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    build:
      type: dockerfile
      config: ./.runme/Dockerfile
    ...
```
in this case the main service will be built from `./.runme/Dockerfile` Dockerfile.

specification with build and image section:
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_image:some_tag
    build:
      type: dockerfile
      config: ./.runme/Dockerfile
    ...
```
in this case the main service will be run from image `some_image:some_tag` and `build` section will be ignored.

### command
`command` - it is option to set command that should be executed inside container to run the service. This option can be string or array of strings.

#### Examples
specification with string command:
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    command: npm run
    ...
```

specification with array command (to set special shell for command)
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    command: ["/bin/bash", "-c", "npm run"]
    ...
```

### environment
`environment` - it's a map to set environment variables inside the container. This variables will not be used on the build stage. So if your application should be built with some environment variables you should add it inside Dockerfile.

#### Example
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    environment:
      KEY1: VAL1
      KEY2: VAL2
    ...
```

### port
`port` - it's section to set port that will be exposed from the main service. The section has array format but Runme allows to use this option only for the main service, and in the current release **only one port can be set**. Each initialization should contains two options `container` and `public` numbers of port. `container` port represents number of port which application will listen inside container to provide interface/service. `public` port represents number of port which will be exposed to public internet (in current release it can be only port `80`).

#### Example
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    ports:
    - container: 3000
      public: 80
    ...
```
in this specification, the application inside docker conteiner will listen port 3000, but Runme will expose this port to public internet as 80.

### volumes
`volumes` - it is option to set list of volumes for the services. If this option set Runme will create temporary volume in memory with name from option `name` and mount this volume inside container to the path from option `mount`.

#### Example
```yaml
version: 1.0
publish: some_app
services:
  some_app:
    image: some_app_image:some_tag
    ports:
    - container: 3000
      public: 80
    volumes:
    - name: data
      mount: /var/data
```
in this specification Runme will create temporary volume with name `data` and mount it inside container to directory `/var/data`.

