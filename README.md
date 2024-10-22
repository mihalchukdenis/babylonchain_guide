# Babylon chain node installation guide

### Step 1: System Requirements
You’ll need the following hardware:
- **CPU**: Quad Core AMD or Intel (amd64)
- **RAM**: 32 GB
- **Storage**: 1 TB NVMe
- **Network**: 100 Mbps bidirectional internet connection.

### Step 2: Install Prerequisites
- Install **Golang** (version 1.21+):

  ```bash
  sudo apt update
  sudo apt install golang-go
  go version  # Should return go1.21+
  ```

- Install build dependencies:

  ```bash
  sudo apt install git build-essential curl jq --yes
  ```

### Step 3: Build and Install Babylon
1. Clone the Babylon GitHub repository:

   ```bash
   git clone https://github.com/babylonlabs-io/networks.git
   cd networks
   ```

2. Checkout the correct version:

   ```bash
   git checkout <version_to_install>
   ```

3. Build and install `babylond`:

   ```bash
   make install
   ```

### Step 4: Initialize the Node
1. Create and initialize the node directory:

   ```bash
   babylond init <node_name> --chain-id bbn-test-3
   ```

2. Download the genesis file:

   ```bash
   wget https://github.com/babylonlabs-io/networks/raw/main/bbn-test-3/genesis.tar.bz2
   tar -xjf genesis.tar.bz2 && mv genesis.json ~/.babylond/config/genesis.json
   ```

### Step 5: Configure Peers
1. Open the configuration file `~/.babylond/config/config.toml`.
2. Set the `seeds` and `persistent_peers` with the approved nodes (found [here](https://github.com/babylonlabs-io/networks)).

3. In `~/.babylond/config/app.toml`, adjust:

   ```toml
   iavl-cache-size = 0
   [btc-config]
   network = "signet"
   ```

### Step 6: Set Up Cosmovisor
1. Install **Cosmovisor**:

   ```bash
   go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
   ```

2. Create the required directories:

   ```bash
   mkdir -p ~/.babylond/cosmovisor/genesis/bin
   ```

3. Move the `babylond` binary to the `genesis/bin` folder:

   ```bash
   cp $GOPATH/bin/babylond ~/.babylond/cosmovisor/genesis/bin/
   ```

4. Create a systemd service for Cosmovisor:

   ```bash
   sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
   [Unit]
   Description=Babylon daemon
   After=network-online.target

   [Service]
   User=$USER
   ExecStart=$(which cosmovisor) run start
   Restart=always
   RestartSec=3
   LimitNOFILE=infinity

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

### Step 7: Start the Node
1. Enable and start the service:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable babylond
   sudo systemctl start babylond
   ```

2. Check the status:

   ```bash
   systemctl status babylond
   ```

### Step 8: Create the Validator
After the node syncs, create the validator with:

```bash
babylond tx staking create-validator \
  --amount 1000000ubbn \
  --from <your_wallet_name> \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --pubkey $(babylond tendermint show-validator) \
  --moniker "<validator_name>" \
  --chain-id bbn-test-3
```

This guide covers the full setup process of a Babylon validator node, from system configuration to staking. For additional details, refer to the official [Babylon documentation](https://docs.babylonlabs.io).
