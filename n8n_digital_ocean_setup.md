# How to Install n8n on a Digital Ocean Server

This guide provides step-by-step instructions for installing n8n on a Digital Ocean droplet using Docker. This method is recommended as it simplifies the installation and future maintenance.

**Assumptions:**
*   You have a Digital Ocean droplet running a common Linux distribution (e.g., Ubuntu).
*   You are logged into your server via SSH with a user that has `sudo` privileges.
*   Your server's IP address is `159.203.138.32`.

---

### Step 1: Connect to Your Server

First, connect to your server using the `ssh` command in your local terminal.

```bash
ssh root@159.203.138.32
```
*(Replace `root` with your username if you are not using the root user)*

---

### Step 2: Install Docker and Docker Compose

If you do not have Docker installed, these commands will install it and the required Docker Compose plugin.

1.  **Update your server's package list:**
    ```bash
    sudo apt-get update
    ```

2.  **Install Docker Engine:**
    ```bash
    sudo apt-get install -y docker.io
    ```

3.  **Install Docker Compose:**
    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```
    *Note: This command installs a specific version of Docker Compose. You can check for a newer version on the [Docker Compose releases page](https://github.com/docker/compose/releases).*

---

### Step 3: Create the n8n Directory and Configuration File

Create a dedicated directory on your server to store your n8n data and configuration.

1.  **Create the directory and navigate into it:**
    ```bash
    mkdir ~/n8n
    cd ~/n8n
    ```

2.  **Create the `docker-compose.yml` file using a text editor like `nano`:**
    ```bash
    nano docker-compose.yml
    ```

3.  **Paste the following configuration into the file.** This configuration tells Docker how to run n8n. Crucially, the `volumes` section ensures that your workflow data is saved to a local directory on your server (`~/n8n/n8n_data`), so it will persist even if you restart or update n8n.

    ```yaml
    version: '3.7'

    services:
      n8n:
        image: n8nio/n8n
        restart: always
        ports:
          - "5678:5678"
        environment:
          # Replace with your timezone to ensure correct scheduling
          - GENERIC_TIMEZONE=America/New_York
          - TZ=America/New_York
        volumes:
          - ./n8n_data:/home/node/.n8n
    ```

4.  **Save and exit the editor.** (In `nano`, press `Ctrl+X`, then `Y`, then `Enter`).

---

### Step 4: Start the n8n Service

With the configuration file in place, you can now start the n8n service.

```bash
sudo docker-compose up -d
```

This command starts the n8n container in "detached" mode (`-d`), so it will run in the background. Docker will download the latest n8n image, which may take a minute or two.

---

### Step 5: Access Your n8n Instance

Your n8n instance is now running. You can access it by opening a web browser and navigating to your server's IP address on port `5678`:

**[http://159.203.138.32:5678](http://159.203.138.32:5678)**

The first time you access this URL, n8n will prompt you to create an owner account. This will be your primary administrative user.

Congratulations, your self-hosted n8n service is now live!
