#!/bin/bash

exists()
{
    command -v "$1" >/dev/null 2>&1
}

if exists curl; then
    echo ''
else
    sudo apt update && sudo apt install curl -y < "/dev/null"
fi

# Determine the user's shell and set the profile file accordingly
if [ -n "$ZSH_VERSION" ]; then
    PROFILE_FILE="$HOME/.zshrc"
else
    PROFILE_FILE="$HOME/.bash_profile"
fi

# Source the profile file if it exists
if [ -f "$PROFILE_FILE" ]; then
    source "$PROFILE_FILE"
fi

sleep 1

if ss -tulpen | awk '{print $5}' | grep -q ":26656$"; then
    echo -e "\e[31mInstallation is not possible, port 26656 already in use.\e[39m"
    exit
else
    echo ""
fi

# Ask the user for input
read -p "Enter User: " USER
read -p "Enter Operating System (e.g. Linux, Darwin): " OPERATING_SYSTEM
read -p "Enter NAMADA_TAG (e.g., v0.30.3): " NAMADA_TAG
read -p "Enter NAMADA_CHAIN_ID: " NAMADA_CHAIN_ID

# Update the profile file with user input
echo 'export USER='\"${USER}\" >> "$PROFILE_FILE"
echo 'export NAMADA_TAG='\"${NAMADA_TAG}\" >> "$PROFILE_FILE"
echo 'export NAMADA_CHAIN_ID='\"${NAMADA_CHAIN_ID}\" >> "$PROFILE_FILE"

sleep 1

cd $HOME
sudo apt update
sudo apt install make unzip clang pkg-config git-core libudev-dev libssl-dev build-essential libclang-12-dev git jq ncdu bsdmainutils htop -y < "/dev/null"

echo -e '\n\e[42mInstall Rust\e[0m\n' && sleep 1
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env

echo -e '\n\e[42mInstall libprotoc 3.12.0\e[0m\n' && sleep 1
if ! command -v protoc; then
    curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v3.12.0/protoc-3.12.0-linux-x86_64.zip
    unzip -o protoc-3.12.0-linux-x86_64.zip -d $HOME/.local
    echo 'export PATH=$PATH:'"$HOME/.local/bin" >> "$PROFILE_FILE"
    source "$PROFILE_FILE"
elif [ "$(protoc --version | awk '{print $NF}')" != "3.12.0" ]; then
    sudo apt remove libprotoc-dev -y
    sudo rm -f $(which protoc)
    curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v3.12.0/protoc-3.12.0-linux-x86_64.zip
    unzip -o protoc-3.12.0-linux-x86_64.zip -d $HOME/.local
    echo 'export PATH=$PATH:'"$HOME/.local/bin" >> "$PROFILE_FILE"
    source "$PROFILE_FILE"
else
    echo "Protocol Buffers already installed"
fi

echo -e '\n\e[42mInstall Go\e[0m\n' && sleep 1
VERSION=1.20.5
wget -O go.tar.gz https://go.dev/dl/go$VERSION.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go.tar.gz && rm go.tar.gz
echo 'export GOROOT=/usr/local/go' >> "$PROFILE_FILE"
echo 'export GOPATH=$HOME/go' >> "$PROFILE_FILE"
echo 'export GO111MODULE=on' >> "$PROFILE_FILE"
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> "$PROFILE_FILE"
source "$PROFILE_FILE"
go version

echo -e '\n\e[42mInstall software\e[0m\n' && sleep 1
mkdir -p $HOME/cometbft_bin
cd $HOME/cometbft_bin
wget -O cometbft.tar.gz https://github.com/cometbft/cometbft/releases/download/v0.37.2/cometbft_0.37.2_linux_amd64.tar.gz
tar xvf cometbft.tar.gz
sudo chmod +x cometbft
sudo mv ./cometbft /usr/local/bin/

cd $HOME
latest_release_url=$(curl -s "https://api.github.com/repos/anoma/namada/releases/latest" | grep "browser_download_url" | cut -d '"' -f 4 | grep "$OPERATING_SYSTEM")
wget "$latest_release_url"
tar -xzvf namada-v0.31.0-Linux-x86_64.tar.gz
cd namada-v0.31.0-Linux-x86_64
sudo cp namada* /home/$USER/.cargo/bin/
