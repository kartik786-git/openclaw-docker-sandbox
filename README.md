# Ollama-GPU and OpenClaw Setup Guide

## Step 1: Ensure Ollama-GPU is Ready
Your Ollama container must be "visible" to other containers.
Restart Ollama with the correct environment variable:
```bash
docker stop ollama-gpu && docker rm ollama-gpu
docker run -d --gpus=all \
 -e OLLAMA_HOST=0.0.0.0 \
 -p 11434:11434 \
 --name ollama-gpu \
 ollama/ollama
```
> **Warning:** Use code with caution.

Pull your preferred model (e.g., Llama 3):
```bash
docker exec -it ollama-gpu ollama pull llama3
```
> **Warning:** Use code with caution.

## Step 2: Create the OpenClaw Container
We use a standard Node.js image to avoid the "Platform/AMD64" errors you saw earlier.
Run the container with port mapping and host bridging:
```bash
docker run -it --name openclaw-sandbox \
 -p 18789:18789 \
 --add-host=host.docker.internal:host-gateway \
 node:22-bookworm /bin/bash
```
> **Warning:** Use code with caution.

## Step 3: Install Dependencies (Inside the Container)
Once inside the root@...:/# prompt, run:
Update and install OpenClaw and Socat (the networking bridge):
```bash
apt update && apt install -y socat
npm install -g openclaw
```
> **Warning:** Use code with caution.

## Step 4: Configure OpenClaw for Local Ollama
Run the onboarder:
```bash
openclaw onboard
```
> **Warning:** Use code with caution.

Use these specific settings during setup:
- **Provider:** Ollama
- **Base URL:** http://host.docker.internal:11434 (Critical: Do NOT use 127.0.0.1)
- **Model ID:** llama3 (or whatever you pulled in Step 1)

## Step 5: The "Networking Hack" (The Most Important Part)
Because OpenClaw 2026 binds to 127.0.0.1 internally, you must bridge it to the Docker external port.
Start OpenClaw on an internal port (18790):
```bash
openclaw gateway --port 18790 &
```
> **Warning:** Use code with caution.

Start the Socat bridge to port 18789:
```bash
socat TCP-LISTEN:18789,fork,reuseaddr TCP:127.0.0.1:18790 &
```
> **Warning:** Use code with caution.

## Step 6: Access the Dashboard
Copy the link provided in the terminal (it will look like http://127.0.0.1...).
Paste it into your Windows browser, but change the port to 18789:
http://localhost:18789/#token=YOUR_TOKEN_HERE

## Summary of "Struggles" Avoided:
- **Platform Error:** Solved by using node:22-bookworm instead of the ARM-only openclaw-dmr image.
- **Syntax Error:** Solved by using Node.js v22 (v12 is too old for modern JavaScript).
- **Ollama Not Found:** Solved by using host.docker.internal instead of localhost.
- **Empty Response:** Solved by using socat to bridge the container's private 127.0.0.1 to the public 0.0.0.0.
