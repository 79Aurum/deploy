apt update && \
apt install make clang pkg-config libssl-dev build-essential gcc xz-utils git curl vim tmux ntp jq llvm ufw -y && \
tmux new -s deploy  
echo Enter your Private Key: && read PK && \
echo Enter your View Key: && read VK && \
echo Enter your Address: && read ADDRESS
echo Enter your Private Key: && read PK && \
echo Enter your View Key: && read VK && \
echo Enter your Address: && read ADDRESS
echo "$ADDRESS"
cd $HOME
git clone https://github.com/AleoHQ/snarkOS.git --depth 1
cd snarkOS
bash ./build_ubuntu.sh
source $HOME/.bashrc
source $HOME/.cargo/env
cd $HOME
git clone https://github.com/AleoHQ/leo.git
cd leo
cargo install --path .
echo Enter the Name of your contract "(any)": && read NAME
cd $HOME && mkdir leo_deploy && cd leo_deploy
leo new $NAME
echo Paste the link: && read QUOTE_LINK && \
CIPHERTEXT=$(curl -s $QUOTE_LINK | jq -r '.execution.transitions[0].outputs[0].value')
RECORD=$(snarkos developer decrypt --ciphertext $CIPHERTEXT --view-key $VK)
snarkos developer deploy "$NAME.aleo" \
--private-key "$PK" \
--query "https://vm.aleo.org/api" \
--path "$HOME/leo_deploy/$NAME/build/" \
--broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" \
--fee 600000 \
--record "$RECORD"
