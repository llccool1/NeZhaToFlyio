# NeZhaToFlyio
部署在FlyIo的哪吒监控面板
#### Apps

```
root@VM-0-15-debian:~/grafana# /root/.fly/bin/flyctl launch
Creating app in /root/grafana
Scanning source code
Could not find a Dockerfile, nor detect a runtime or framework from source code. Continuing with a blank app.
? App Name (leave blank to use an auto-generated name): mi
Automatically selected personal organization: literature
? Select region: hkg (Hong Kong, Hong Kong)
Created app mi in organization personal
Wrote config file fly.toml
```

#### Volumes

```
root@VM-0-15-debian:~/grafana# /root/.fly/bin/flyctl volumes create grafana_storage  --region hkg --size 3
        ID: vol_g2yxp4mgjgd463qd
      Name: grafana_storage
       App: mi
    Region: hkg
      Zone: c8cf
   Size GB: 3
 Encrypted: true
Created at: 03 Sep 22 04:49 UTC
```

Edit `fly.toml` to add mount information. Grafana defaults to `/var/lib/grafana` for persistence, so we can just mount it directly.
```toml
[mount]
source = "grafana_storage"
destination = "/var/lib/grafana"
```

#### Deploy

This is all you need, just run `flyctl deploy` and watch what happens.

Once that's done, run `flyctl open` to launch the Grafana UI in your browser. Grafana defaults to `admin` for both username and password.
```
root@VM-0-15-debian:~/grafana# /root/.fly/bin/flyctl deploy==> Verifying app config
--> Verified app config
==> Building image
Searching for image 'grafana/grafana' remotely...
image found: img_n37qpoxd2lepmz69
==> Creating release
--> release v2 created

--> You can detach the terminal anytime without stopping the deployment
==> Monitoring deployment

 1 desired, 1 placed, 0 healthy, 0 unhealthy [health checks: 1 total, 1 pass 1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 pass 1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 passing]
--> v0 deployed successfully
```

#### Plugins

Grafana has a number of interesting plugins, like the [Worldmap Panel](https://grafana.com/grafana/plugins/grafana-worldmap-panel). The Grafana Docker image will install plugins from an environment variable. You can configure environment variables in `fly.toml` like so:

```toml
[env]
GF_INSTALL_PLUGINS = "grafana-worldmap-panel,grafana-clock-panel"
```

Run a quick `deploy`, check the logs, and you'll see messages like this:

```
2020-11-04T22:41:08.458Z ecb06e0a ord [info] installing grafana-worldmap-panel @ 0.3.2
2020-11-04T22:41:08.460Z ecb06e0a ord [info] from: https://grafana.com/api/plugins/grafana-worldmap-panel/versions/0.3.2/download
2020-11-04T22:41:08.461Z ecb06e0a ord [info] into: /var/lib/grafana/plugins
2020-11-04T22:41:08.928Z ecb06e0a ord [info] ✔ Installed grafana-worldmap-panel successfully
2020-11-04T22:41:08.930Z ecb06e0a ord [info] Restart grafana after installing plugins . <service grafana-server restart>
2020-11-04T22:41:09.011Z ecb06e0a ord [info] installing grafana-clock-panel @ 1.1.1
2020-11-04T22:41:09.012Z ecb06e0a ord [info] from: https://grafana.com/api/plugins/grafana-clock-panel/versions/1.1.1/download
2020-11-04T22:41:09.013Z ecb06e0a ord [info] into: /var/lib/grafana/plugins
2020-11-04T22:41:09.302Z ecb06e0a ord [info] ✔ Installed grafana-clock-panel successfully
2020-11-04T22:41:09.303Z ecb06e0a ord [info] Restart grafana after installing plugins . <service grafana-server restart>
```
