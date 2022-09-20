# webhookz

This repo sends a GitHub webhook to a private server with the [OpenZiti Webhook Action](https://github.com/marketplace/actions/ziti-webhook-action-python) over a zero trust, private network based on [OpenZiti](https://github.com/openziti/ziti#readme) ![GitHub Repo stars](https://img.shields.io/github/stars/openziti/ziti?style=social).

## Use this template repository

Do this first: there's a button in the GitHub UI to "Use this template" which will give you a blank commit history in a repo generated from this one. Clone your new repo and follow the steps below in your Git working copy directory.

## Decide how you will configure your OpenZiti Network

You might prefer to self-host your network or you may use the network provided by the Ziti Edge Developer Sandbox (ZEDS). Self-hosting means you won't need to sign up for anything and you'll have full transparency of the open source software and total control of the network. ZEDS requires a painless and free-forever signup for a private developer sandbox on shared servers and does provide end to end encryption like all OpenZiti Networks.

## Hosting Option 1: Configure a self-hosted OpenZiti Network

The steps for this hosting option will guide you to self-host an OpenZiti Network by following [the "Run on your own server" quickstart](https://openziti.github.io/ziti/quickstarts/network/hosted.html).

With that zero trust overlay in place you can follow these steps on your OpenZiti server to add the necessary entities.

1. Create an identity for the webhook sender i.e. API consumer / client

    ```bash
    ziti edge create identity device github --jwt-output-file github.jwt --role-attributes webhookz-senders
    ```

1. Create an identity for the webhook server

    ```bash
    ziti edge create identity device server --jwt-output-file server.jwt --role-attributes webhookz-servers
    ```

1. Create a config with type `intercept.v1`.

    ```bash
    ziti edge create config webhookz-intercept-config intercept.v1 '{"protocols":["tcp"],"addresses":["webhookz.ziti"], "portRanges":[{"low":80, "high":80}]}'
    ```

1. Create a config with type `host.v1`

    ```bash
    ziti edge create config webhookz-host-config host.v1 '{"protocol":"tcp", "address":"httpbin","port":8080}'
    ```

1. Create a service to associate the two configs

    ```bash
    ziti edge create service webhookz-service --configs webhookz-intercept-config,webhookz-host-config
    ```

1. Create a bind service policy

    ```bash
    ziti edge create service-policy webhookz-bind-policy Bind --service-roles '@webhookz-service' --identity-roles '#webhookz-servers'
    ```

1. Create a dial service policy

    ```bash
    ziti edge create service-policy webhookz-dial-policy Dial --service-roles '@webhookz-service' --identity-roles '#webhookz-senders'
    ```

1. Copy "github.jwt" and "server.jwt" from the OpenZiti server to the computer where you cloned this repo.

## Hosting Option 2: Configure OpenZiti with the Developer Sandbox

We'll use Ziti Edge Developer Sandbox (ZEDS) in this example.

1. Sign up for a free developer account at https://zeds.openziti.org/. We'll create a couple of identities to attach to an OpenZiti network provided by ZEDS.
1. In ZEDS, follow the "build your app" button. Populate the form with an app name like "my webhook app", two identities "github" and "server", and a service named "webhookz". Finish up with the "build my app" button.
1. On the following screen click the edit button for the service "webhookz".
    1. Create a config with type `intercept.v1`.

    ```json
    {
        "addresses": [
            "webhookz.ziti"
        ],
        "protocols": [
            "tcp"
        ],
        "portRanges": [
            {
            "low": 80,
            "high": 80
            }
        ]
    }
    ```

    1. Create a config with type `host.v1`.

    ```json
    {
        "port": 8080,
        "address": "httpbin",
        "protocol": "tcp"
    }
    ```

1. Go back to "manage app" and click the download buttons to get the token files for identities "github" and "server".

## Run the private webhook server

A private webhook server is included in this repo as a Docker Compose project to demonstrate how to publish an unmodified API producer with OpenZiti. These steps will guide you run and publish the server so it can receive the GitHub webhook via OpenZiti. The Compose project provides an OpenZiti sidecar to host the service that provides zero trust ingress to the server in an isolated Docker network.

![webhookz drawio (2)](https://user-images.githubusercontent.com/1434400/191101246-dcb23b17-3e33-4b67-98b7-51d7e4a7126e.svg)

1. Copy "github.jwt" and "server.jwt" into the directory where you cloned this repo.
1. In your terminal, change to the directory where you cloned this repo. Enroll "github" to obtain the identity file "github.json" in the current directory.

    ```bash
    docker run --rm --volume ${PWD}:/mnt/ openziti/quickstart /openziti/ziti-bin/ziti edge enroll /mnt/github.jwt 
    ```

1. In the GitHub UI, create a new GitHub Actions secret named `ZITI_WEBHOOK_IDENTITY` with the contents of "github.json".
1. In your terminal, run the Docker Compose project to start the demo webhook server and OpenZiti tunneler.

   ```bash
   docker compose up --detach
   ```

   You should now have a new file "server.json" in the directory where you cloned this repo. That is the OpenZiti identity file used by the tunneler running in one of the containers.

1. Finally, trigger the GitHub Actions workflow to demonstrate sending a GitHub webhook to your private server. Navigate to "Actions" / "Main Workflow" in the GitHub UI and punch the "Run workflow" button. You may instead trigger it with the GitHub CLI if your current directory is repo.

    ```bash
    gh workflow run main.yml
    ```

1. Optionally, follow the private server's log to see the webhook activity from GitHub

    ```bash
    docker compose logs --follow httpbin
    ```

The result of the triggered workflow will be an HTTP response from the `httpbin` application echoing the payload and headers it received.

## Have questions?

* Follow our [Blog](https://openziti.io/)
* Join [Discussion](https://openziti.discourse.group)
* [Development](https://github.com/openziti)
* [Documentation](https://openziti.github.io)
* Like it? Give us a [star](https://github.com/openziti/ziti)
