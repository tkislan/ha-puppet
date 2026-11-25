# ha-puppet

Docker image of [Puppet](https://github.com/balloob/home-assistant-addons/tree/main/puppet), the
headless-Chromium server that renders Home Assistant dashboards to images, by Paulus Schoutsen.
[ha-mcp](https://github.com/homeassistant-ai/ha-mcp) uses it for dashboard screenshots. Upstream
ships Puppet only as a Home Assistant add-on that the Supervisor builds locally, so there is no
published image for Kubernetes or Docker installs.

The CI workflow checks out `balloob/home-assistant-addons` at `PUPPET_COMMIT` and builds
`puppet/` with upstream's own Dockerfile. It changes one line: Puppet listens on
`127.0.0.1:10000` instead of every interface, and the line is marked as changed. Puppet has no
authentication, so nothing outside its network namespace can reach it; run an authenticating
proxy in the same pod in front of it. The build fails if upstream's code has any other `.listen(`
call. Upstream's `LICENSE` is copied to `/app/LICENSE`.

## Tags

`tkislan/ha-puppet:<Puppet version>.<workflow run number>`, for example `2.6.3.12`, for
`linux/amd64`. There is no `latest`. Every push to `main` publishes a new tag, and so does a
rebuild on the 1st of each month, which picks up Debian's Chromium and base image updates.
Pull requests build the image without pushing it. GitHub disables scheduled workflows in a public
repository after 60 days without activity; re-enable the workflow under Actions when that happens.

## Running

Puppet reads its settings from `/data/options.json`:

```json
{
  "access_token": "<Home Assistant long-lived access token>",
  "home_assistant_url": "http://homeassistant:8123",
  "keep_browser_open": false
}
```

## Updating Puppet

Set `PUPPET_COMMIT` in `.github/workflows/ci.yml` to the commit of a `Release Puppet x.y.z` in
`balloob/home-assistant-addons`. The build fails if upstream no longer has exactly one
`server.listen(port);` line for the patch to replace.

## Publishing

Publishing needs the Actions secret `DOCKER_PASSWORD`, a Docker Hub access token for `tkislan`
with write access to `tkislan/ha-puppet`.

## License

Puppet is licensed under Apache-2.0 by its authors. This repository only builds and patches it.
