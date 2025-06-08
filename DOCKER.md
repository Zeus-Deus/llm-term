# 🐳 Docker Build Instructions for LLM-Term

If you do not want to install Rust on your system, you can build `llm-term` using Docker. You can also run Ollama in a container for a hybrid workflow where `llm-term` runs locally and Ollama runs in a container:

## 1. Clone the Repository

```bash
git clone https://github.com/Zeus-Deus/llm-term.git
cd llm-term
```

## 2. Create a Dockerfile

Create a file named `Dockerfile` in the project root with the following content:

```Dockerfile
FROM rust:1.76-slim as builder
WORKDIR /build
COPY . .
RUN cargo build --release

FROM debian:stable-slim
WORKDIR /app
COPY --from=builder /build/target/release/llm-term .
ENTRYPOINT ["/app/llm-term"]
```

## 3. Build the Docker Image

```bash
docker build -t llm-term-builder .
```

## 4. Extract the Compiled Binary

```bash
docker create --name extract-llm-term llm-term-builder
docker cp extract-llm-term:/app/llm-term .
docker rm extract-llm-term
```

## 5. Install the Binary Locally

```bash
mkdir -p ~/.local/bin
mv llm-term ~/.local/bin/
chmod +x ~/.local/bin/llm-term
```

Ensure `~/.local/bin` is in your `$PATH` so you can run `llm-term` from any terminal.

## Running Ollama in a Container

You can run Ollama in a Docker container instead of installing it locally. For example:

```bash
docker run -d --name ollama -p 11434:11434 ollama/ollama
```

This will start Ollama and expose it on port 11434. Make sure this port is accessible to your local `llm-term` installation (i.e., `llm-term` should be able to reach `localhost:11434`).

## Connecting LLM-Term to Containerized Ollama

- By default, `llm-term` expects Ollama to be available at `localhost:11434`.
- If you change the port or run Ollama on a different host, update your configuration accordingly.
- See Docker documentation for details on exposing and forwarding ports.

## Ollama Model Name Note

Currently, `llm-term` is hardcoded to use the `llama3.1` model with Ollama. You cannot change the model name from within the program.

If you download a specific parameter model (for example, `llama3.1:8b`), you must create an alias named `llama3.1` so that `llm-term` can use it. For example:

```bash
docker exec -it ollama ollama cp llama3.1:8b llama3.1
```

This step is required because `llm-term` only recognizes the default `llama3.1` model name when connecting to Ollama.
