### GitHub Actions workflow to demonstrate how to run a user systemd service

The tested echo server container [__ghcr.io/eriksjolund/socket-activate-echo__](https://github.com/eriksjolund/socket-activate-echo/pkgs/container/socket-activate-echo 
) supports _socket activation_.

Using [socket activation](https://github.com/containers/podman/blob/main/docs/tutorials/socket_activation.md) has
the advantage that there is no need for the client to use _/usr/bin/sleep_ combined with a while-loop to check if the server is online.

The file [.github/workflows/demo.yml](.github/workflows/demo.yml) contains:

```
name: Echo server demo with podman and systemd
on: 
  push:
    branch:
      - main
jobs:
  systemd_podman_echo_server_demo:
    runs-on: ubuntu-22.04
    permissions:
      contents: read
    steps:
      - name: Run actions/checkout for this repo
        uses: actions/checkout@v3
        with:
          path: checkout
          persist-credentials: false
      - name: Run an echo server with podman in a user systemd service
        run: |
          sudo apt-get update
          sudo apt-get install -y socat
          podman pull ghcr.io/eriksjolund/socket-activate-echo:latest
          mkdir -p ~/.config/systemd/user
          podman create --rm --name echo --network none ghcr.io/eriksjolund/socket-activate-echo
          podman generate systemd --name --new echo > ~/.config/systemd/user/echo.service
          cp checkout/echo.socket ~/.config/systemd/user/
          systemctl --user daemon-reload
          systemctl --user start echo.socket
          echo hello | socat - tcp4:127.0.0.1:3000
          echo hello | socat - tcp6:[::1]:3000
          echo hello | socat - udp4:127.0.0.1:3000
          echo hello | socat - udp6:[::1]:3000
          echo hello | socat - unix:$HOME/echo_stream_sock
          echo hello | socat - VSOCK-CONNECT:1:3000
```
