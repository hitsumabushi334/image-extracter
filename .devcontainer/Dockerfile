FROM python:3.11-slim

ARG TZ
ENV TZ="$TZ"

# Install basic development tools and iptables/ipset
RUN apt update && apt install -y less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  curl \
  wget \
  build-essential

# Create python user
RUN useradd -m -s /bin/zsh python && \
  usermod -aG sudo python && \
  echo 'python ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

ARG USERNAME=python

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/python/.claude && \
  chown -R python:python /workspace /home/python/.claude

WORKDIR /workspace

RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/0.18.2/git-delta_0.18.2_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_0.18.2_${ARCH}.deb" && \
  rm "git-delta_0.18.2_${ARCH}.deb"

# Install Python development tools
RUN pip install --upgrade pip && \
  pip install poetry black flake8 pylint pytest ipython uv

# Set up non-root user
USER python

# ---- START: Added for human-in-the-loop ----
# Install Rust (as python user)
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
# Add Rust to the PATH for subsequent commands
ENV PATH="/home/python/.cargo/bin:${PATH}"
# ---- END: Added for human-in-the-loop ----

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Default powerline10k theme
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.0/zsh-in-docker.sh)" -- \
  -p git \
  -a "source /usr/share/fzf/shell/key-bindings.zsh" \
  -a "source /usr/share/fzf/shell/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x
# Install Node.js for Claude Code
USER root
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
  apt-get install -y nodejs
# Configure npm for the non-root user
USER python
# 1. pythonユーザーのホームディレクトリに、npmパッケージ用のディレクトリを作成
RUN mkdir -p /home/python/.npm-global
# 2. npmに対して、グローバルインストールの場所を上記ディレクトリに設定
RUN npm config set prefix '/home/python/.npm-global'
# 3. 新しく設定したパスを環境変数PATHに追加し、コマンドが見つかるようにする
ENV PATH="/home/python/.npm-global/bin:${PATH}"
# ---- END: npm/npx 修正箇所 ----
# Install Claude Code via npm
RUN npm install -g @anthropic-ai/claude-code@1.0.81 ccusage@latest @google/gemini-cli@latest

# ---- START: Added for human-in-the-loop ----
# Install human-in-the-loop MCP server using cargo
RUN cargo install --git https://github.com/KOBA789/human-in-the-loop.git
# ---- END: Added for human-in-the-loop ----
    
USER root

COPY .devcontainer/init-firewall.sh /usr/local/bin/init-firewall.sh
COPY .devcontainer/scripts /home/python/scripts/

RUN chmod +x /usr/local/bin/init-firewall.sh && \
  chown python:python /usr/local/bin/init-firewall.sh && \
  chown -R python:python /home/python/scripts && \
  chmod +x /home/python/scripts/deny-check.sh && \
  chmod +x /home/python/scripts/discord_notification.sh && \
  # Ensure all scripts have correct line endings
  sed -i 's/\r$//' /home/python/scripts/deny-check.sh && \
  sed -i 's/\r$//' /home/python/scripts/discord_notification.sh && \
  echo "python ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/python-firewall && \
  echo "python ALL=(root) NOPASSWD: /usr/bin/chmod +x /home/python/scripts/deny-check.sh" >> /etc/sudoers.d/python-firewall && \
  chmod 0440 /etc/sudoers.d/python-firewall
USER python