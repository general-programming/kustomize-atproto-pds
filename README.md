# atproto-pds kustomization

Based on the Docker Compose files from https://github.com/bluesky-social/pds

I'm not too familiar with Kustomize best practices but I made a best effort to make the files not too opinionated to my infrastructure.

## Example kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: bsky-pds
resources:
  - github.com/general-programming/kustomize-atproto-pds?ref=v0.0.1 # main can be used if you are brave
  - ingress.yaml # bring your own ingress
  - secret.yaml # generate your own secret.yaml below or use your choice of secret store
patches:
  - path: patch-config.yaml
```

## Config
All ConfigMap values come from the upstream. Patch `pds-config` in your Kustomization as needed.

Common:
- PDS_HOSTNAME: "your-pds.example.com"

Other:
- PDS_DATA_DIRECTORY: "/data"
- PDS_BLOBSTORE_DISK_LOCATION: "/data/blocks"
- PDS_DID_PLC_URL: "https://plc.directory"
- PDS_BSKY_APP_VIEW_URL: "https://api.bsky.app"
- PDS_BSKY_APP_VIEW_DID: "did:web:api.bsky.app"
- PDS_REPORT_SERVICE_URL: "https://mod.bsky.app"
- PDS_REPORT_SERVICE_DID: "did:plc:ar7c4by46qjdydhdevvrndac"
- PDS_CRAWLERS: "https://bsky.network"
- LOG_ENABLED: "true"

## Secrets
Patch in secret `pds-secrets` with these values.

- PDS_JWT_SECRET: [can be generated with `openssl rand --hex 16`]
- PDS_ADMIN_PASSWORD: [can be generated with `openssl rand --hex 16`]
- PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX: (can be generated with `openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32`)

### Oneblocker
A patch can be generated with the following shell snippet.
```sh
cat << EOL
apiVersion: v1
kind: Secret
metadata:
  name: pds-secrets
data:
    PDS_JWT_SECRET: $(openssl rand --hex 16 | base64)
    PDS_ADMIN_PASSWORD: $(openssl rand --hex 16 | base64)
    PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX: $(openssl ecparam --name secp256k1 --genkey --noout --outform DER | tail --bytes=+8 | head --bytes=32 | xxd --plain --cols 32 | base64)
EOL
```
