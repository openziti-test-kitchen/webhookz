# webhookz

Template repository for sending a GitHub webhook to a private server

## Use this template repository

Do this first: there's a button in the GitHub UI to "Use this template" which will give you a separate commit history based on this repo. Clone your new repo and follow the steps in the remaining sections of this readme.

## Configure OpenZiti

A private HTTP server is included in this repo. These steps will guide you run and publish the server so it can receive the GitHub webhook via OpenZiti.

1. Sign up for a free account in the Ziti Edge Developer Sandbox (ZEDS) at https://zeds.openziti.org/. ZEDS will provide the OpenZiti network.
1. In ZEDS, follow the "build your app" button to skip the guide. Populate the form with an app name like "my webhook app", two identities "github" and "server", and a service named "webhookz". Finish up with the "build my app" button.
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

1. In the GitHub UI, create a new GitHub Actions secret named `ZITI_WEBHOOK_ACTION_ID` with the contents of "github.json".
1. Create another GitHub Actions secret named `ZITI_WEBHOOK_SECRET` with a long, random string. You may generate the string with Python:

    ```bash
    python -c "import os, binascii; print(binascii.hexlify(os.urandom(20)).decode('utf-8'))"
    ```
1. In your terminal, run the Docker Compose project to start the demo webhook server and OpenZiti tunneler.

   ```bash
   docker compose up --detach
   ```

   You should now have a new file "server.json" in the directory where you cloned this repo. That is the OpenZiti identity file used by the tunneler running in one of the containers.