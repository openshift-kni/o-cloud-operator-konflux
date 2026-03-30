# o-cloud-operator-konflux

Konflux build definitions for the O-Cloud Manager operator: bundles, FBC catalogs, and Tekton pipelines.

## Repository structure

```
.
├── .tekton/                        # All Tekton pipelines (PAC discovery)
│   ├── build-pipeline.yaml         # Shared bundle build pipeline
│   ├── o-cloud-manager-bundle-0-1-{pull-request,push}.yaml
│   ├── fbc-pipeline-4-{20,21,22}.yaml
│   ├── images-mirror-set-4-{20,21,22}.yaml
│   └── o-cloud-manager-fbc-4-{20,21,22}-{pull-request,push}.yaml
│
├── bundles/                        # Operator bundle definitions
│   ├── catalog-idms.yaml           # Shared IDMS for quay.io ↔ registry.redhat.io mapping
│   └── v0.1/                       # Bundle version 0.1 (Tech Preview)
│       ├── oran-o2ims/             # Git submodule → openshift-kni/oran-o2ims (main)
│       ├── Dockerfile.bundle       # Bundle image build
│       ├── metadata/               # Konflux-specific annotation overrides
│       └── overlay/                # Image pinning, mapping, and release overlays
│
├── catalog/                        # FBC (File-Based Catalog) definitions
│   ├── fbc-images-resolvable-integration-test-idms.yaml
│   ├── 4.20/                       # Catalog for OpenShift 4.20
│   │   ├── Dockerfile.catalog
│   │   └── templates/              # Modular FBC template files
│   │       ├── openshift-4-20-template.in.yaml
│   │       ├── o-cloud-manager-fbc-base.yaml
│   │       ├── o-cloud-manager-channel-0-1.yaml
│   │       └── o-cloud-manager-deprecated-channels-4-20.yaml
│   ├── 4.21/                       # Catalog for OpenShift 4.21
│   └── 4.22/                       # Catalog for OpenShift 4.22
│
├── telco5g-konflux/                # Git submodule → shared Konflux build scripts
├── .gitmodules                     # Submodule definitions
├── OWNERS                          # Reviewer/approver groups
└── renovate.json                   # Automated dependency updates
```

## Submodules

| Submodule | Source | Branch | Purpose |
|---|---|---|---|
| `telco5g-konflux` | by2waysprojects/telco5g-konflux | CNF-22415-add-build-multi-catalog | Shared Konflux build scripts |
| `bundles/v0.1/oran-o2ims` | openshift-kni/oran-o2ims | main | Operator source and bundle manifests |

After cloning, initialize submodules:

```bash
git submodule update --init --recursive
```

## FBC channels

| OCP Version | Operator Version | FBC Channels |
|---|---|---|
| 4.20 | 4.20 (deprecated) | `stable`, `4.20` |
| 4.21 | 4.21 (deprecated) | `stable`, `4.21` |
| 4.20, 4.21, 4.22+ | 0.1 (TP) | `pre-ga-0.1` |

## How it works

1. **Bundle pipeline**: When the `oran-o2ims` submodule pointer is updated (via Renovate), the bundle Tekton pipeline builds a new bundle image applying the overlays from `bundles/v0.1/overlay/`.

2. **Catalog pipeline**: When catalog templates or `.tekton/` files change, the FBC pipeline runs `telco5g-konflux/scripts/catalog/konflux-build-catalog-from-resources-template.sh` to assemble the modular template files into a single `catalog-template.in.yaml`, then builds the catalog index image with `opm`.
