# 🌐 Aztec Sequencer Node Setup 

## 💡 Hardware Requirements

| **RAM** | **CPU**   | **Disk**    |
| ------- | --------- | ----------- |
| 8-16 GB | 4-9 Cores | 100+ GB SSD |

---

### 🔹 Step 1: Update the System

```bash
sudo apt update -y && sudo apt upgrade -y
```

* Updates your system to the latest available packages.

---

### 🔹 Step 2: Install Docker

```bash
# Remove old Docker versions if any
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker repository
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

---

### 🔹 Step 3: Install AZTEC CLI

```bash
bash -i <(curl -s https://install.aztec.network)
```

```bash
echo 'export PATH=$PATH:$HOME/.aztec/bin' >> ~/.bashrc
source ~/.bashrc
aztec-up alpha-testnet
aztec --version
```

---

### 🔹 Step 4: Wallet Setup

* Create an Ethereum-compatible wallet (like MetaMask).
* Store your wallet information securely.

---

### 🔹 Step 5: Get Testnet Ethereum (Sepolia)

* Obtain 3 to 5 Sepolia ETH from these faucets:

  * [QuickNode Faucet](https://faucet.quicknode.com/ethereum/sepolia)
  * [Alchemy Faucet](https://www.alchemy.com/faucets/ethereum-sepolia)
  * [Trade Faucet](https://faucet.trade/sepolia-eth-faucet)
  * [Google Cloud Faucet](https://cloud.google.com/application/web3/faucet)
  * [PK910 Faucet](https://sepolia-faucet.pk910.de/)
* If you can't get from faucets, buy from [Gas ZIP](https://www.gas.zip/)

---

### 🔹 Step 6: Configure RPC for Sepolia Network

* Get valid **free** RPC URLs from the following links:

  * **Sepolia:** [Ankr RPC](https://www.ankr.com/rpc/)
  * **Sepolia BEACON:** [DRPC Dashboard](https://drpc.org/dashboard/)

---

### 🔹 Step 7: Get Your Server IP

```bash
curl ipv4.icanhazip.com
```

---

### 🔹 Step 8: Firewall Configuration (Optional)

```bash
ufw allow 22
ufw allow ssh
ufw allow 40400
ufw allow 8080
ufw enable
```

---

### 🔹 Step 9: Create a Screen Session for Node Running

```bash
screen -S aztec
```

---

### 🔹 Step 10: Run the Node (Replace the values)

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls YOUR_RPC_URL \
  --l1-consensus-host-urls YOUR_RPC_URL \
  --sequencer.validatorPrivateKey YOUR_PRIVATE_KEY \
  --sequencer.coinbase YOUR_EVM_ADDRESS \
  --p2p.p2pIp YOUR_SERVER_IP
```

💬 **Explanation:**

* `YOUR_RPC_URL` ➡️ RPC URL from **Step 6**.

  🔸 **Sepolia:** `YOUR_RPC_URL`

  🔸 **Sepolia BEACON:** `YOUR_RPC_URL`

* `YOUR_PRIVATE_KEY` ➡️ Private Key (should start with `0x`).

* `YOUR_EVM_ADDRESS` ➡️ Ethereum wallet address.

* `YOUR_SERVER_IP` ➡️ Server IP from **Step 7**.

🕐 **Node synchronization takes about 1 hour.**

🔸 **To detach from screen:**

* Press `Ctrl + A`, then `D`.

Tip: After an hour, return and check if the node is running.
If it's not, you can restart it by running the same run command again.

📟 To check or reconnect to the screen session **screen -r aztec**


---

### 🔹 Step 11: Get Block Number and Proof

```bash
BLOCK=$(curl -s -X POST -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' http://localhost:8080 | jq -r ".result.proven.number") && echo "Block: $BLOCK" && RESULT=$(curl -s -X POST -H 'Content-Type: application/json' -d "{\"jsonrpc\":\"2.0\",\"method\":\"node_getArchiveSiblingPath\",\"params\":[\"$BLOCK\",\"$BLOCK\"],\"id\":67}" http://localhost:8080 | jq -r ".result") && echo "Result:" && echo "$RESULT"
```

---

### 🔹 Step 12: Join Discord for Apprentice Role

* Join the Aztec Discord: [Aztec Discord](https://discord.gg/aztec)
* Navigate to **operators/faq** room.
* Type `/operator start`
* Provide the following:

  * Ethereum Wallet Address
  * Block Number
  * Proof Result

---

### 🔹 Step 13: Register Your Validator

```bash
aztec add-l1-validator \
  --l1-rpc-urls YOUR_RPC_URL \
  --private-key YOUR_PRIVATE_KEY \
  --attester YOUR_EVM_ADDRESS \
  --proposer-eoa YOUR_EVM_ADDRESS \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111
```
💬 **Explanation:**

* `YOUR_RPC_URL` ➡️ RPC URL from **Step 6**.

  🔸 **Sepolia:** `YOUR_RPC_URL`

  🔸 **Sepolia BEACON:** `YOUR_RPC_URL`

* `YOUR_PRIVATE_KEY` ➡️ Private Key (should start with `0x`).

* `YOUR_EVM_ADDRESS` ➡️ Ethereum wallet address.

* `YOUR_SERVER_IP` ➡️ Server IP from **Step 7**.

---

### 🔹 Step 14: Verify Node's Peer ID

```bash
sudo docker logs $(docker ps -q --filter ancestor=aztecprotocol/aztec:alpha-testnet | head -n 1) 2>&1 | grep -i "peerId" | grep -o '"peerId":"[^"]*"' | cut -d'"' -f4 | head -n 1
```

* Visit:

  * [Aztec Nethermind](https://aztec.nethermind.io/)
  * [AztecScan](https://aztecscan.xyz/validators)

---

---

📢 **Follow for Updates:**
- Twitter: [https://x.com/Reza_Cryptoo](https://x.com/Reza_Cryptoo)
- Telegram: [https://t.me/Rezaa_Cryptoo](https://t.me/Rezaa_Cryptoo)
