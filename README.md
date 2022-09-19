# webhookz

This repo sends a GitHub webhook to a private server with the [OpenZiti Webhook Action](https://github.com/marketplace/actions/ziti-webhook-action-python) over a zero trust, private network based on [OpenZiti](https://github.com/openziti/ziti#readme) ![GitHub Repo stars](https://img.shields.io/github/stars/openziti/ziti?style=social).

![webhookz drawio](https://user-images.githubusercontent.com/1434400/191099368-64161664-1c81-4f99-8020-e6bf5ae10adb.svg)

## Use this template repository

Do this first: there's a button in the GitHub UI to "Use this template" which will give you a separate commit history based on this repo. Clone your new repo and follow the steps below.

## Configure OpenZiti

A private webhook server is included in this repo as a Docker Compose project. These steps will guide you run and publish the server so it can receive the GitHub webhook via OpenZiti. We'll use Ziti Edge Developer Sandbox (ZEDS) in this example, but you also have the option to self-host an OpenZiti Network by running [the quickstart](https://openziti.github.io/ziti/quickstarts/quickstart-overview.html).

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

1. Finally, trigger the GitHub Actions workflow to demonstrate sending a GitHub webhook to your private server. Navigate to the workflow in the GitHub UI and punch the "Run workflow" button. You may instead trigger it with the GitHub CLI.

    ```bash
    gh workflow run main.yml
    ```

1. Optionally, follow the private server's log to see the webhook activity from GitHub

    ```bash
    docker compose logs --follow httpbin
    ```

## Have questions?

* Follow our [Blog](https://openziti.io/)
* Join [Discussion](https://openziti.discourse.group)
* [Development](https://github.com/openziti)
* [Documentation](https://openziti.github.io)
* Like it? Give us a [star](https://github.com/openziti/ziti)
