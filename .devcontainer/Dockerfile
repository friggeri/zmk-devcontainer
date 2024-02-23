FROM docker.io/zmkfirmware/zmk-dev-arm:3.5

RUN curl -sS https://starship.rs/install.sh | sh -s -- -y

COPY .bashrc starship.toml /tmp
RUN mv /tmp/.bashrc ~/.bashrc && \
   mkdir -p ~/.config/ && \
   mv /tmp/starship.toml ~/.config/starship.toml