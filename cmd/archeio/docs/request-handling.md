# Request Handling

Requests to archeio follows the following flow:

1. If it's a request for `/`: redirect to our wiki page about the project
1. If it's a request for `/privacy`: redirect to linux foundation privacy policy page
1. If it's not a request for `/` or `/privacy` and does not start with `/v2/`: 404 error
1. For registry API requests, all of which start with `/v2/`:
    - If it's a manifest request: redirect to Upstream Registry
    - If it's from a known GCP IP: redirect to Upstream Registry
    -  If it's a known AWS IP AND HEAD request for the layer succeeeds in S3: redirect to S3
    -  If it's a known AWS IP AND HEAD fails: redirect to Upstream Registry
1. OCI Distribution [Specification](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)

Currently the `Upstream Registry` is a region specific Artifact Registry backend.

Or in chart form:
```mermaid
flowchart TD

A(Does the request path start with /v2/?) -->|No, it is not a registry API call| B(Is the request for /?)
B -->|No| D[Is the request for /privacy?]
D -->|No, it is an unknown path| C[Serve 404 error]
D -->|Yes| K[Serve redirect to linux foundation privacy policy page]
B -->|Yes| E[Serve redirect to registry wiki page]
A -->|Yes, it is a registry API call| F(Is it a blob request?)
F -->|No| G[Serve redirect to Source Registry on GCP]
F -->|Yes, it matches known blob request format| H(Is the client IP known to be from GCP?)
H -->|Yes| G
H -->|No| I(Does the blob exist in S3?<br/>Check by way of cached HEAD on the bucket we've selected based on client IP.)
I -->|No| G
I -->|Yes| J[Redirect to blob copy in S3]
```

This allows us to efficiently serve traffic in the most local copy available
based on the cloud resource funding the Kubernetes project receives.
