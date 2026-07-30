# Docker Setup For Ollama

Docker Compose setup for running Ollama with the Open WebUI frontend, with optional NVIDIA GPU support and a locked-down private network.

Save either block below as `docker-compose.yml` and start it with `docker compose up -d`. Open WebUI is then available at `http://localhost:3000`.

## Standard Setup
```yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ollama_storage:/root/.ollama
    # If you don't have an NVIDIA GPU, delete the 'deploy' section below
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - webui_storage:/app/backend/data
    depends_on:
      - ollama

volumes:
  ollama_storage:
  webui_storage:
```

## Setup with Private Network
```yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ollama_storage:/root/.ollama
    networks:
      - private_llm_net
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: always
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - webui_storage:/app/backend/data
    networks:
      - private_llm_net
    depends_on:
      - ollama

networks:
  private_llm_net:
    internal: true  # This blocks all external internet traffic
```

> **NOTE** 
> To download a model, the `internal` flag on the network needs to be set to `false` (or removed) so the container can reach the internet.

## Downloading a model

```bash
docker exec -it ollama ollama run gemma4
```