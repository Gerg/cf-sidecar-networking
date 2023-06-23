# Cloud Foundry Sidecar TLS Networking

This is a proof of concept for communicating with Cloud Foundry application
sidecars over TLS.

## Procedure 

1. `cf push --var app-domain=<APP DOMAIN GOES HERE>`
1. Find the server route guid: `cf curl '/v3/routes?hosts=server' | jq '.resources[].guid'`
1. Find the app guid: `cf curl '/v3/apps?names=cf-sidecar-networking' | jq '.resources[].guid'`
1. Update the server route to point to app port 8081, instead of 8080: `cf curl -X PATCH /v3/routes/<ROUTE GUID GOES HERE>/destinations -d '{"destinations": [{"app": { "guid": "<APP GUID GOES HERE>", "process": { "type": "web" } }, "weight": null, "port": 8081, "protocol": "http1" }]}'`
1. Authorize c2c networking between the app and itself: `cf add-network-policy cf-sidecar-networking cf-sidecar-networking`
1. Restart the app to update port configuration for the container: `cf restart cf-sidecar-networking`
1. Curl the server: `curl server.cherry-trader.capi.land`

## How This Works

The C2C TLS external container port is only available for the default app port
(8080). We can make the sidecar listen on this port, so it can do C2C over TLS,
but then we need to move the webserver to another port (8081, but this could be
anything).

Relevant documentation:
- https://docs.cloudfoundry.org/devguide/deploy-apps/cf-networking.html
- https://v3-apidocs.cloudfoundry.org/version/release-candidate/#replace-all-destinations-for-a-route
- https://docs.cloudfoundry.org/devguide/deploy-apps/manifest-attributes.html
- https://docs.cloudfoundry.org/devguide/custom-ports.html
- https://docs.cloudfoundry.org/devguide/sidecars.html
