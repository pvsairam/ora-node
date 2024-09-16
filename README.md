<h1 align="center">Join the Incentivized Ora Node Network</h1>

<p align="center">
  <img src="https://github.com/user-attachments/assets/a5069cfd-6dc6-4919-8bb3-d71aecebba39" alt="Ora Protocol">
</p>

**Ora Protocol** leverages decentralized AI models to provide oracle services for smart contracts on the blockchain.

- 💰 **Funding**: Successfully raised **$20 million** from top-tier investors like **Polychain** and **Hashkey**.
- 🤝 **Partnerships**: In collaboration with leading blockchain networks such as **Optimism**, **Arbitrum**, and **Base**.

# Setting Up the Tora Node

## Hardware Requirements
| Ram       | CPU       | Disk                     |
| --------- | --------- | ------------------------ |
| `8 GB minimum` | `1 Core` | `20-40 GB SSD` |

## Step 1: Install Dependencies
Ensure Docker is installed and configured:
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker installation
docker --version
```

## Step 2: Acquire Alchemy API Key
* Sign up at: https://dashboard.alchemy.com/
* Create an `Ethereum` API and copy both `HTTPS` and `Websockets` for `Sepolia` & `Mainnet`. 

![Screenshot_8](https://github.com/user-attachments/assets/04946697-9497-43ed-9ce1-51737f8704bc)


## Step 3: Setup the Tora Node
### 1- Create a directory for Tora
```console
mkdir tora
cd tora
```

### 2- Create the `.env` file
```console
nano .env
```

**Insert the following details into the .env file:**

* Replace `PRIV_KEY` with your Metamask wallet's private key.
* Ensure sufficient `ETH` is available on both `Sepolia` and `Mainnet` for gas fees.
* Provide your `Alchemy` API URLs for `MAINNET_WSS`, `MAINNET_HTTP`, `SEPOLIA_WSS`, and `SEPOLIA_HTTP`.
* Set your desired chain confirmation with `CONFIRM_CHAINS`.

```bash
############### Sensitive config ###############

# private key for app transactions

PRIV_KEY=""

############### General config ###############

# provider URLs

MAINNET_WSS=""
MAINNET_HTTP=""
SEPOLIA_WSS=""
SEPOLIA_HTTP=""

# Redis cache time-to-live

REDIS_TTL=86400000 # 1 day

############### App specific config ###############

# chain confirmations

CONFIRM_CHAINS='["sepolia","mainnet"]'
CONFIRM_MODELS='[13]' # OpenLM model
CONFIRM_USE_CROSSCHECK=true
CONFIRM_CC_POLLING_INTERVAL=3000 # 3 sec polling
CONFIRM_CC_BATCH_BLOCKS_COUNT=300
CONFIRM_TASK_TTL=2592000000

```
* Press `CTRL+X`, then `Y`, and `ENTER` to save and exit.

### 3- Create the `docker-compose.yml` file
```console
nano docker-compose.yml
```

**Insert the following code**
```
# ora node docker-compose
version: '3'
services:
  confirm:
    image: oraprotocol/tora:confirm
    container_name: ora-tora
    depends_on:
      - redis
      - openlm
    command: 
      - "--confirm"
    env_file:
      - .env
    environment:
      REDIS_HOST: 'redis'
      REDIS_PORT: 6379
      CONFIRM_MODEL_SERVER_13: 'http://openlm:5000/'
    networks:
      - private_network
  redis:
    image: oraprotocol/redis:latest
    container_name: ora-redis
    restart: always
    networks:
      - private_network
  openlm:
    image: oraprotocol/openlm:latest
    container_name: ora-openlm
    restart: always
    networks:
      - private_network
  diun:
    image: crazymax/diun:latest
    container_name: diun
    command: serve
    volumes:
      - "./data:/data"
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      - "TZ=Asia/Shanghai"
      - "LOG_LEVEL=info"
      - "LOG_JSON=false"
      - "DIUN_WATCH_WORKERS=5"
      - "DIUN_WATCH_JITTER=30"
      - "DIUN_WATCH_SCHEDULE=0 0 * * *"
      - "DIUN_PROVIDERS_DOCKER=true"
      - "DIUN_PROVIDERS_DOCKER_WATCHBYDEFAULT=true"
    restart: always

networks:
  private_network:
    driver: bridge
```
* Press `CTRL+X`, then `Y`, and `ENTER` to save and exit.

## Step 4: Start the Tora Node
```console
docker compose up -d
```

## Step 5: Check logs
```console
cd $HOME && cd tora
docker compose logs -f
```

* To test that the node is running correctly, request inference from OpenLM on **Sepolia** at: https://www.ora.io/app/opml/openlm

* Upon successful request, you will observe a log entry from the Tora validator, confirming that it has received and processed the transaction.
![image](https://github.com/user-attachments/assets/bd308f81-7eb8-443a-a554-acfb58acbe12)

* Now you know, whoever use Ora OLM on `Sepolia` & `Mainnet`, You will verify a transaction and get these logs, and will receive points as rewards 
