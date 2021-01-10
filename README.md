### GitHub Actions workflow to demonstrate how to run a user systemd service

Note that `loginctl enable-linger runner` needs to be followed by a `sleep`.
One second seems to be enough (`sleep 1`).

The file [.github/workflows/demo.yml](.github/workflows/demo.yml) contains:

```
name: Web server demo with podman and systemd
on: 
  push:
    branch:
      - main
jobs:
  systemd_podman_web_server_demo:
    runs-on: ubuntu-20.04
    steps:
      - name: Run actions/checkout for this repo
        uses: actions/checkout@v2
        with:
          path: checkout_testing
          persist-credentials: false
      - name: Run a web server with podman in a user systemd service
        run: |
          loginctl enable-linger runner
          sleep 1
          ls /run/user/$UID 
          podman pull docker.io/library/nginx:latest
          mkdir -p $HOME/.config/systemd/user
          cp checkout_testing/webserver.service $HOME/.config/systemd/user
          XDG_RUNTIME_DIR=/run/user/$UID systemctl --user enable --now webserver.service
          sleep 1
          curl http://localhost:8080
```
