## OCI Diagram

#### As Ascii Art
```ascii
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                      OCI Diagram                                            │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                             │
│  ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────┐ │
│  │  OCI Registry   │ 1:N   │    OCI Repo     │ 1:N   │  OCI Artifact   │ 1:N   │   OCI   │ │
│  │                 │──────>│                 │──────>│                 │──────>│  Layer  │ │
│  │ - Container for │       │ - Collection of │       │ - Has layers,   │       │         │ │
│  │   0 to N repos  │       │   0 to N        │       │   manifest,     │       │ - Single│ │
│  │ - Examples:     │       │   artifacts     │       │   and tags      │       │   piece │ │
│  │   docker.io,    │       │ - Identified    │       │ - Has mediaType │       │   of    │ │
│  │   ghcr.io,      │       │   by URI        │       │ - Identified by │       │   data  │ │
│  │   cgr.dev       │       │ - Examples:     │       │   digest        │       │ - Has   │ │
│  │                 │       │   docker.io/    │       │ - Examples:     │       │   media │ │
│  │                 │       │   python,       │       │   python:latest │       │   Type  │ │
│  │                 │       │   ghcr.io/      │       │                 │       │ - Has   │ │
│  │                 │       │   homebrew/     │       │                 │       │   digest│ │
│  │                 │       │   core/crane    │       │                 │       │         │ │
│  └─────────────────┘       └─────────────────┘       └─────────────────┘       └─────────┘ │
│                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                             Chainguard Specific Example                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                             │
│  ┌──────────────┐         ┌─────────────────────────┐                                      │
│  │   cgr.dev    │────────>│  cgr.dev/chainguard/ko  │                                      │
│  │              │         │  (Chainguard repo)      │                                      │
│  │ Chainguard   │         └──────────┬──────────────┘                                      │
│  │  registry    │                    │                                                     │
│  └──────────────┘                    │                                                     │
│                                      ├──────────────────────┬────────────────────────┐     │
│                                      │                      │                        │     │
│                                      ▼                      ▼                        ▼     │
│             ┌────────────────────────────────┐  ┌──────────────────────┐  ┌───────────────┐│
│             │ cgr.dev/chainguard/ko@         │  │ cgr.dev/chainguard/  │  │ cgr.dev/...   ││
│             │ sha256:9d36fe9...              │  │ ko:sha256-9d36fe9... │  │ :sha256-...   ││
│             │ (image artifact)               │  │ .att (attestations)  │  │ .sig (cosign) ││
│             └───────┬────────┬───────────────┘  └──────┬───────┬───────┘  └───────┬───────┘│
│                     │        │                         │       │                  │        │
│                     ▼        ▼                         ▼       ▼                  ▼        │
│              ┌─────────┐ ┌─────────┐           ┌─────────┐ ┌─────────┐      ┌─────────┐   │
│              │L1       │ │L2       │           │L1       │ │LN       │      │L1       │   │
│              │(tar.gz) │ │(tar.gz) │           │(envelope)│(envelope)│      │(cosign) │   │
│              └─────────┘ └─────────┘           └─────────┘ └─────────┘      └─────────┘   │
│                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

#### As Mermaid Diagram
```mermaid
graph LR
    subgraph "OCI Diagram"
        Registry["OCI Registry<br/>Container for 0 to N OCI repos<br/>Examples: docker.io, ghcr.io, cgr.dev"]
        Repo["OCI Repo<br/>Collection for 0 to N OCI Artifacts<br/>with the same name differentiated<br/>by tag or digest<br/>Examples: docker.io/python,<br/>ghcr.io/homebrew/core/crane"]
        Artifact["OCI Artifact<br/>A thing in the registry with one or<br/>more layers, a manifest, and tags<br/>Has a mediaType defining what it encodes<br/>Identifiable by digest (hash of manifest)<br/>Examples: docker.io/python:latest"]
        Layer["OCI Layer<br/>A single piece of data of an artifact<br/>Each layer has a mediaType<br/>Identified by its own digest"]

        Registry -->|"1:N"| Repo
        Repo -->|"1:N"| Artifact
        Artifact -->|"1:N"| Layer
    end

    subgraph "Chainguard Specific Example"
        CG_Registry["cgr.dev<br/>Chainguard registry"]
        CG_Repo["cgr.dev/chainguard/ko<br/>Chainguard repo for ko"]

        CG_Image["cgr.dev/chainguard/ko@sha256:9d36fe9...<br/>image artifact in repo"]
        CG_Att["cgr.dev/chainguard/ko:sha256-9d36fe9....att<br/>envelope artifact defining<br/>cosign attestations"]
        CG_Sig["cgr.dev/chainguard/ko:sha256-9d36fe9....sig<br/>cosign artifact defining<br/>the providence"]

        L1_Image["L1 (tar.gz)"]
        L2_Image["L2 (tar.gz)"]
        L1_Att["L1 (envelope)"]
        LN_Att["LN (envelope)"]
        L1_Sig["L1 (cosign)"]

        CG_Registry --> CG_Repo
        CG_Repo --> CG_Image
        CG_Repo --> CG_Att
        CG_Repo --> CG_Sig

        CG_Image --> L1_Image
        CG_Image --> L2_Image
        CG_Att --> L1_Att
        CG_Att --> LN_Att
        CG_Sig --> L1_Sig
    end
```
