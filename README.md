# Wild Kernels for Android

## Your warranty is no longer valid!

I am **not responsible** for bricked devices, damaged hardware, or any issues that arise from using this kernel.

**Please** do thorough research and fully understand the features included in this kernel before flashing it!

By flashing this kernel, **YOU** are choosing to make these modifications. If something goes wrong, **do not blame me**!

---

### Proceed at your own risk!

<table>
  <tr>
    <th> :warning: </th>
    <th> Verify <a href="https://github.com/WildKernels/OnePlus_KernelSU_SUSFS/blob/main/compatibility.md">Compatibility</a> of kernels before flashing. </th>
  </tr>
</table>

---

# Kernels:
 
[GKI](https://github.com/WildKernels/GKI_KernelSU_SUSFS)  
[Sultan](https://github.com/WildKernels/Sultan_KernelSU_SUSFS)  
[OnePlus](https://github.com/WildKernels/OnePlus_KernelSU_SUSFS)  
[Legacy Pixels](https://github.com/WildKernels/Pixel_KernelSU_SUSFS)  

---

# Other Links:

[Kernel Patches](https://github.com/WildKernels/kernel_patches)  
[Old Build Scripts](https://github.com/TheWildJames/kernel_build_scripts)  
[Kernel Flasher - fatalcoder524 fork](https://github.com/fatalcoder524/KernelFlasher)  
[Horizon Kernel Flasher](https://github.com/libxzr/HorizonKernelFlasher)  

---

# Installation instructions: 

Follow the steps for GKI:  
[Installation](https://kernelsu.org/guide/installation.html)

To get boot.img format:  
[Get My Kernel Format](https://github.com/TheWildJames/Get_My_Kernel_Format)

---

# Features

- **KernelSU**: KernelSU is a root solution for Android GKI devices, it works in kernel mode and grants root permission to userspace applications directly in kernel space.
- **SUSFS**: An addon root hiding kernel patches and userspace module for KernelSU.

---

# Credits

- **KernelSU**: Developed by [tiann](https://github.com/tiann/KernelSU).
- **KernelSU-Next**: Developed by [rifsxd](https://github.com/KernelSU-Next/KernelSU-Next).
- **Magic-KSU**: Developed by [5ec1cff](https://github.com/5ec1cff/KernelSU).  
- **SUSFS**: Developed by [simonpunk](https://gitlab.com/simonpunk/susfs4ksu.git).
- **SUSFS Module**: Developed by [sidex15](https://github.com/sidex15).
- **Sultan Kernels**: Developed by [kerneltoast](https://github.com/kerneltoast).

Special thanks to the open-source community for their contributions!

---

# Support

If you encounter any issues or need help, feel free to open an issue in this repository or reach out to me.

---

# Disclaimer

Flashing this kernel will void your warranty, and there is always a risk of bricking your device. Please make sure to back up your data and ensure you understand the risks before proceeding.

**Proceed at your own risk!**

---

[Telegram](https://t.me/TheWildJames)  
[Telegram Group](https://t.me/WildKernels)  

# Special thanks to the following people for their contributions!
This helps me alot! <3

[simonpunk](https://gitlab.com/simonpunk/susfs4ksu.git) - Created SUSFS!  
[sidex15](https://github.com/sidex15) - Created module!  
[backslashxx](https://github.com/backslashxx) - Helped with patches!  
[Teemo](https://github.com/liqideqq) - Helped with patches!  
[幕落](https://github.com/MuLuo688) - Donation!

If you have contributed and are not here please remind me!

---

# 🖥️ Self-Hosted Runner Setup (Ubuntu 24.04 / WSL2)

This section covers the complete setup required to run the CI pipeline on self-hosted GitHub Actions runners.

> **Tested environment:** Ubuntu 24.04 on WSL2 (Windows 11), 4 concurrent runners, ext4 filesystem, 1 TB+ disk.

---

## 1. System Packages

```bash
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  git curl ca-certificates build-essential clang lld flex bison \
  libelf-dev libssl-dev libncurses-dev zlib1g-dev liblz4-tool \
  libxml2-utils rsync unzip dwarves file python3 ccache jq bc \
  dos2unix kmod libdw-dev elfutils gcc-aarch64-linux-gnu
```

---

## 2. Node.js >= 20

Required by `actions/checkout@v4` and other actions:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version   # must be v20.x or higher
```

---

## 3. GitHub CLI

Required by the `trigger-release` job:

```bash
(type -p wget >/dev/null || sudo apt-get install wget -y) && \
sudo mkdir -p -m 755 /etc/apt/keyrings && \
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null && \
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg && \
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] \
  https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null && \
sudo apt-get update && sudo apt-get install gh -y

gh auth login
```

---

## 4. sudo Without Password

The workflow uses `sudo` in non-interactive steps:

```bash
echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
```

---

## 5. bindgen (for Rust kernel builds)

Required for devices with `rust_build: true` (e.g. OP15, OP15r, ACE6T).

> The standard `curl ... | sh` method fails on WSL2 because the rustup installer writes to `/tmp` before the `chmod` step. Downloading directly to `$HOME` avoids this issue.

```bash
curl --proto '=https' --tlsv1.2 -sSf \
  "https://static.rust-lang.org/rustup/dist/x86_64-unknown-linux-gnu/rustup-init" \
  -o "$HOME/rustup-init"
chmod +x "$HOME/rustup-init"
"$HOME/rustup-init" -y --default-toolchain none --no-modify-path
rm -f "$HOME/rustup-init"

source "$HOME/.cargo/env"
cargo install bindgen-cli --version 0.69.5
bindgen --version   # must print: bindgen 0.69.5

# Add cargo to shell profile
echo 'source "$HOME/.cargo/env"' >> "$HOME/.bashrc"
```

---

## 6. ccache

> **Critical:** `hard_link = false` is required. With `hard_link = true`, ccache creates read-only hard links for `.o` files, causing `objcopy` to fail during kernel builds on ext4.

```bash
mkdir -p ~/.ccache
cat > ~/.ccache/ccache.conf << EOF
max_size = 40G
compression = true
compression_level = 1
direct_mode = true
hash_dir = false
file_clone = false
inode_cache = true
umask = 002
sloppiness = file_macro,time_macros,include_file_mtime,include_file_ctime,pch_defines,system_headers,locale
cache_dir = $HOME/.ccache
depend_mode = false
hard_link = false
stats_log = $HOME/.ccache/stats.log
compiler_check = content
EOF

ccache -p | grep -E "max_size|hard_link|compression"   # verify
```

---

## 7. git Configuration

```bash
git config --global user.name "your-github-username"
git config --global user.email "your@email.com"

# Required on WSL2 to prevent "dubious ownership" errors
# when the runner agent creates _work directories
git config --global safe.directory '*'
```

---

## 8. Runner Registration

Create the runner directory and register it with GitHub.

The example below sets up a **single runner**. To add more runners for parallel builds, repeat the steps in a new directory (e.g. `$HOME/self-host/runner2/`, `runner3/`, etc.) using a fresh registration token each time.

```bash
RUNNER_DIR="$HOME/self-host/runner"
REPO_URL="https://github.com/YOUR_USERNAME/YOUR_REPO"

mkdir -p "$RUNNER_DIR"
cd "$RUNNER_DIR"

# Check https://github.com/actions/runner/releases for the latest version
curl -o actions-runner-linux-x64.tar.gz -L \
  "https://github.com/actions/runner/releases/download/v2.331.0/actions-runner-linux-x64-2.331.0.tar.gz"
tar xzf ./actions-runner-linux-x64.tar.gz
rm actions-runner-linux-x64.tar.gz

# Get token from: Repository -> Settings -> Actions -> Runners -> New self-hosted runner
./config.sh --url "$REPO_URL" \
            --token YOUR_REGISTRATION_TOKEN \
            --name "runner" \
            --labels "self-hosted,ubuntu-24.04" \
            --work "_work" \
            --unattended
```

---

## 9. Runner PATH (`.env`)

The runner needs `bindgen` and `cargo` in its PATH. The `.env` file is read only at `run.sh` startup — restart the runner after any change.

```bash
echo "PATH=$HOME/.cargo/bin:$PATH" >> "$HOME/self-host/runner/.env"

# Verify
grep PATH "$HOME/self-host/runner/.env"
```

---

## 10. Starting the Runner

Start the runner in a terminal. The `.env` file is read only at startup.

```bash
wsl -d Ubuntu-24.04 bash -c "/home/YOUR_USERNAME/self-host/runner/run.sh"
```

The runner prints `Listening for Jobs` when ready.

---

## Directory Structure

```
$HOME/
├── .ccache/                        # ccache storage (up to 40 GB)
│   └── ccache.conf
├── .cargo/
│   └── bin/
│       └── bindgen                 # bindgen 0.69.5
└── self-host/
    └── runner/
        ├── run.sh
        ├── .env                    # PATH=$HOME/.cargo/bin:$PATH
        └── _work/
            └── REPO/REPO/          # Workspace (auto-managed by runner)
```

---

## Important Notes

- **Manual startup required:** The runner must be started manually with `run.sh`. If WSL2 restarts, restart it before triggering any workflow.
- **Multiple runners:** Each additional runner is an independent directory registered separately. All runners share `~/.ccache`, which is safe and beneficial — concurrent builds of different devices populate a shared cache with no risk of corruption.
- **Disk usage:** Kernel sources (~20–80 GB per device) remain on disk after each run. To free space: `rm -rf ~/self-host/runner/_work/REPO/REPO/DEVICE_NAME/`
- **`hard_link = false` is mandatory:** With `hard_link = true`, `.o` files become read-only and `objcopy` fails during kernel builds on ext4.
