# Start Kit for local AI (n8n, ollama, OpenWebUI)

This repository provides a quick setup for a comprehensive local AI and low-code development environment using Podman.

It is based of [self-hosted-ai-starter-kit](https://github.com/n8n-io/self-hosted-ai-starter-kit) and built for Mac Silicon users only.

The core of this kit is a Docker Compose file, pre-configured with network and storage settings,
minimizing the need for additional installations.

It includes:

- Self-hosted n8n - Low-code platform with over 400 integrations and advanced AI components
- Qdrant - Open-source, high performance vector store with an comprehensive API
- PostgreSQL - Workhorse of the Data Engineering world, handles large amounts of data safely.
- Ollama - Cross-platform LLM platform to install and run the latest local LLMs
- OpenWebUI - Web UI for Ollama and for all kinds of LLM solutions

## 1. Ollama (local on Mac)

If you’re using a Mac with an M1 or newer processor, you can't expose your GPU to the Podman or Docker instance.

So the option used here is to run Ollama on your Mac (not in Podman) for faster inference, and connect to that from the n8n instance.

To install Ollama on a Mac, download the installer from the [Ollama website](https://ollama.com/),
extract the .zip file, and drag the Ollama.app to your Applications folder.

Then, open the app and follow the setup wizard to complete the installation.

This will install the server (you should have a new icon in the top bar) and the command line version (ollama).

Install your first model that will be used by n8n defaulty workflow (next section).

```shell
ollama pull llama3.2:latest
```

Once ollama is installed and server is running, external applications can access using:

- "http://localhost:11434" if application runs natively on mac
- "http://host.docker.internal:11434/" is application runs in podman (or docker) containers.

## 2. Podman

This kit uses Podman and not Docker.

### Overview

While "containers are Linux," Podman also runs on Mac and Windows, where it provides a native podman CLI and embeds a guest Linux system to launch your containers.
This guest is referred to as a **Podman machine** and is managed with the `podman machine` command.
Podman on Mac and Windows also listens for Docker API clients, supporting direct usage of Docker-based tools and programmatic access from your language of choice.

### Benefits Over Docker Setup

Using Podman in a corporate environment offers several advantages:

- No root daemon running in the background
- Better security isolation
- Compatibility with corporate security policies
- Similar command syntax to Docker (easy transition)

### Podman Installation - Direct Download

1. Visit the [Podman website](https://podman.io/getting-started/installation)
2. Download the macOS installer package
3. Run the installer and follow the prompts
4. Open Terminal and initialize Podman:

Use the podman installer (and not brew installation that is not providing krunkit binary).

```shell
podman machine init
podman machine start
```

### Podman Desktop GUI

For those who like a graphical interface, Podman Desktop provides an easy-to-use GUI:

1. Download Podman Desktop from [podman-desktop.io](https://podman-desktop.io/)
2. Install and launch the application
3. The GUI allows you to manage containers, images, and volumes without command-line use
4. You can perform all the same operations through the UI interface and command line

### Customizing the podman machine (Optional)

> Note: NO NEED TO DO THIS STEP, just shown as an example if you want to customize your machine.

Another interesting benefit of having easy access to the machine is that you can tweak it for a specific usage, such as one that requires more computational power.

Remove the previous machine (must be stopped)
Important: this command removes all containers and images you may have if this is not the first time you are using Podman on your Mac.

```shell
podman machine rm
```

Create and start a machine with custom settings.

```bash
podman machine init \
  --cpus 4 \
  --memory 2048 \
  --disk-size 100 \
  --now
```

Now your containers run in a considerably more powerful VM.
You can read about the `machine init` options in [Podman's documentation](https://docs.podman.io/en/latest/markdown/podman-machine-init.1.html).

## 3. n8n (Podman)

Podman containers included in the docker-compose file:

- Self-hosted n8n
- Qdrant
- PostgreSQL
- OpenWebUI

### For Mac / Apple Silicon users

Copy .env.example and update secrets and passwords inside:

```shell
cp .env.example .env 
```

The OLLAMA_HOST environment variable is modified (because ollama runs locally on the Mac)

(Set OLLAMA_HOST to host.docker.internal:11434 in your .env file.)

### Start containers

```shell
podman compose up -d
```

### Quick start and usage

After completing the installation steps above and when you see in the n8n container log:

- "Editor is now accessible via: http://localhost:5678/":

 simply follow the steps below to get started.

- Open <http://localhost:5678/> in your browser
- You should land on the Workflows page
- Click on "Credentials" (<http://localhost:5678>)
- Click on "Local Ollama service"
- Change the base URL to "http://host.docker.internal:11434/"
- You should see "Connection tested successfully"
- Open the included workflow: <http://localhost:5678/workflow/srOnR8PAY3u4RSwb>
- Click the Chat button at the bottom of the canvas, to start running the workflow.

### Upgrading

```shell
podman compose pull
podman compose create && podman compose up
```

## 4. OpenWebUI (Podman)

OpenWebUI is one of the containers started with `podman compose up -d` above.

Open your browser to <http://localhost:3000>

During first-time setup, Open WebUI automatically detects Ollama running on http://host.docker.internal:11434.

If automatic detection fails:

- Click on your name in the top right hand corner
- Click Admin Panel
- Click Settings
- Select Connections on the left
- Set Ollama API URL to: http://host.docker.internal:11434
- Click Save


## 5. Appendix: standalone OpenWebUI container (podman)

> NOTE: This is not required. This is only if you want to quickly run OpenWebUI alone with ollama, without n8n.

[Open WebUI](https://github.com/open-webui/open-webui) is the most **popular and feature-rich solution to get a web UI** for Ollama.
The project initially aimed at helping you work with Ollama. 
But, as it evolved, it wants to be a web UI provider for all kinds of LLM solutions.

It supports OpenAI-compatible APIs and works entirely offline.
You can install it quickly using Docker or Kubernetes.
Furthermore, it features a Progressive Web App for mobile, and image generation integrations.

### Installing OpenWebUI (standalone containers)

With Podman ready, we can pull and run OpenWebUI:

```shell
# Pull the latest OpenWebUI image
podman pull ghcr.io/open-webui/open-webui:main

# Run OpenWebUI
podman run -d -p 3000:8080 --name open-webui -v open-webui:/app/backend/data ghcr.io/open-webui/open-webui:main
```

If you get a port conflict, you can use a different port:

```shell
# Run on port 3001 instead
podman run -d -p 3001:8080 --name open-webui -v open-webui:/app/backend/data ghcr.io/open-webui/open-webui:main
```

### Connecting to Ollama

Once OpenWebUI is running:

- Open your browser to `http://localhost:3000` (or `3001` if you changed the port)
- Click on your name in the top right hand corner
- Click Admin Panel
- Click Settings
- Select Connections on the left
- Set Ollama API URL to: http://host.docker.internal:11434
- Click Save 

> Note: if OpenWebUI runs natively on your mac, then ollama url is: `http://localhost:11434`
