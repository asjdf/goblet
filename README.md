# Goblet: Git caching proxy

Goblet is a Git proxy server that caches repositories for read access. Git
clients can configure their repositories to use this as an HTTP proxy server,
and this proxy server serves git-fetch requests if it can be served from the
local cache.

In the Git protocol, the server creates a pack-file dynamically based on the
objects that the clients have. Because of this, caching Git protocol response
is hard as different client needs a different response. Goblet parses the
content of the HTTP POST requests and tells if the request can be served from
the local cache.

This was developed by Google to reduce the automation traffic to googlesource.com. 
Goblet would be useful if you need to run a Git read-only mirroring server to offload
the traffic.

We took the initial implementation from 
[`github.com/google/goblet`](https://github.com/google/goblet) at commit 
`140dd10abcdde487161f1e3c2c6e9b4868f7326c` and added the following modifications:
- Removed all code specific to `googlesource.com`.
- Removed all the GCP-related code and dependencies.
- Added support for caching GitHub repositories, and automatically fetch every 15 minutes.
- Added support for GitHub Apps as authentication mechanism.
- Added DataDog for exporting metrics.
- Remove Bazel. Use pure Go tooling to build and release.

## Usage
1. Build Goblet
    ```bash
    bazel build //goblet-server
    ```
2. Start the Goblet server
    ```bash
    export GH_APP_ID="<APP_ID>"
    export GH_APP_INSTALLATION_ID="<APP_INSTALLATION_ID>"
    export GH_APP_PRIVATE_KEY="<APP_PRIVATE_KEY_PEM_TEXT>"
    bazel-bin/goblet-server/goblet-server_/goblet-server -config "<PATH_TO_CONFIG_FILE>"
    ```

    See `example_config.json` for how the minimum config file should look like.

3. Configure your `Git` client to use `Goblet` as a read-only proxy:
    ```bash
    git config http.proxy "http://localhost:8080"
    git remote set-url origin "http://github.com/<owner>/<repo-name>.git"
    git remote set-url --push origin "git@github.com:<owner>/<repo-name>.git"
    ```
4. Try a `git fetch` command and watch `Goblet`'s outputs to see if it's working 
   as expected.
    ```bash
   git fetch origin master
    ```

## Limitations

Note that Goblet forwards the ls-refs traffic to the upstream server. If the
upstream server is down, Goblet is effectively down. Technically, we can modify
Goblet to serve even if the upstream is down, but the current implementation
doesn't do such thing.
