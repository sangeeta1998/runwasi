# Install required dependencies
sudo apt-get install -y libseccomp-dev build-essential curl git

# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

source $HOME/.cargo/env

# Clone the runwasi repository
git clone https://github.com/containerd/runwasi.git
cd runwasi

# Build the wasmtime shim for aarch64
cargo build --target=aarch64-unknown-linux-gnu --target-dir=./target/ -p containerd-shim-wasmtime

# Copy the built Wasmtime shim to a location where containerd can find it:

```sudo cp ~/runwasi/target/aarch64-unknown-linux-gnu/release/containerd-shim-wasmtime-v1 /usr/local/bin/containerd-shim-wasmtime```

# If containerd is not already installed, install it:


```wget https://github.com/containerd/containerd/releases/download/v1.7.15/containerd-1.7.15-linux-arm64.tar.gz```
```sudo tar Cxzvf /usr/local containerd-1.7.15-linux-arm64.tar.gz```

# Set up systemd service for containerd
sudo tee /etc/systemd/system/containerd.service <<EOF
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStart=/usr/local/bin/containerd
Delegate=yes
KillMode=process
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd and start containerd

sudo systemctl daemon-reload

sudo systemctl enable --now containerd

# configure containerd to use the wasmtime shim by modifying the config.toml file:

```sudo nano /etc/containerd/config.toml```

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.wasmtime]
  runtime_type = "io.containerd.wasmtime.v1"
  runtime_engine = "/usr/local/bin/containerd-shim-wasmtime"
  runtime_root = ""



sudo systemctl restart containerd

sudo systemctl status containerd

ctr runtime list
(should have wasmtime)

# Verify built output

```ls ./target/aarch64-unknown-linux-gnu/debug/```

Copying the binary to /usr/local/bin

```sudo cp ./target/aarch64-unknown-linux-gnu/debug/containerd-shim-wasmtime /usr/local/bin/```

Confirm that the binary is correctly placed

```ls -l /usr/local/bin/containerd-shim-wasmtime```

sudo systemctl restart containerd

Assuming we already extracted and placed the containerd binary in /usr/local/bin, the command to create a systemd service file would be:


# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

# Add  user to the Docker group
sudo usermod -aG docker $USER

newgrp docker

# Install ctr
sudo apt-get install containerd-tools


