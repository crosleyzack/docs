# Grype Components
- Vunnel - download security feeds into normalized local files
- Grype-DB - convert normalized local files into Sqlite DB
- Syft - convert target (image, files, purl, etc) into SBOM
- Grype - use SBOM and Sqlite DB to create vuln status
- Vulnerability Match Labels - integration test input-output data
```
 ┌────────────────────────────────────────────┐
 │ Anchore                                    │
 │                                            │
 │ ┌────────┐ ┌──────────┐ ┌──────┐ ┌───────┐ │
 │ │        │ │          │ │      │ │       │ │
 │ │ vunnel │ │ grype-db │ │ syft │ │ grype │ │ impl
 │ │        │ │          │ │      │ │       │ │
 │ └────────┘ └──────────┘ └──────┘ └───────┘ │
 │ ┌────────────────────────────┐             │
 │ │                            │             │
 │ │ vulnerability-match-labels │             │ testing
 │ │                            │             │
 │ └────────────────────────────┘             │
 └────────────────────────────────────────────┘
```
---

## Flow
```
                          Scan Targets                          
 Security Feeds            ┌──────┐                             
   ┌──────┐                │Image │                             
   │Cg_OSV│                │  ┌──────┐                          
   │  ┌──────┐             │  │Files │                          
   │  │Cg_VEX│             └──│ ┌──────┐                        
   └──│ ┌──────┐              │ │PURL  │                        
      │ │Github│              └─│ ┌──────┐                      
      └─│ ┌──────┐              │ │...   │                      
        │ │RHEL  │              └─│      │                      
        └─│  ... │                └──────┘                      
       │  └──────┘                 │                            
  ┌────▼───┐                    ┌──▼───┐                        
  │ Vunnel │                    │      │                        
  │ ------ │                    │ Syft │                        
  └────────┘                    │ ---- │                        
       │                        └──┬───┘                        
    Normalize                      │                            
       │                        ┌──▼───┐                        
┌──────▼─────┐                  │ SBOM │                        
│ Temp Files │                  │      ┼──┐                     
│ ┌───┐      │                  └──────┘  │                     
│ │VEX│      │                   ______   │                     
│ │ ┌───┐    │                  /      \  │   ┌─────────┐       
│ └─│OSV│    │   ┌─────────┐   │\______/│ └───►         │       
│   │ ┌───┐  │   │         │   │        │     │  Grype  ┼─►Vulns
│   └─│...│  ├──►│ Grype-db├──►│ SqlLite├─────►  -----  │       
│     │   │  │   │ ------- │   │        │     └─────────┘       
│     └───┘  │   └─────────┘   │        │                       
└────────────┘                  \______/                        
```
 1. Vunnel pulls in security feeds into normalized local store
 2. Grype-DB converts local store into sqlite database
 3. Syft converts input into SBOM
 4. Grype converts SBOM and sqlite database to vulns
---

## Download Daily Datastore
- Pull grype-db and checkout appropriate branch
- `make boostrap` in grype-db will setup your local env
- `make download-all-provider-cache` pulls cache from Anchore into local sqlite db
```
 Security Feeds               Scan Targets                              
  ┌──────┐                     ┌──────┐                                 
  │Cg_OSV│                     │Image │                                 
  │  ┌──────┐                  │  ┌──────┐                              
  │  │Cg_VEX│                  │  │Files │                              
  └──│ ┌──────┐                └──│ ┌──────┐                            
     │ │Github│                   │ │PURL  │                            
     └─│ ┌──────┐                 └─│ ┌──────┐                          
       │ │RHEL  │                   │ │...   │                          
       └─│  ... │                   └─│      ┼──────┐                   
         └──────┘                     └──────┘   ┌──▼───┐               
       │          ┌────────────────────────────┐ │      │               
  ┌────▼───┐      │    ______                  │ │ Syft │               
  │ Vunnel │      │   /      \                 │ │ ---- │               
  │ ------ │      │  │\______/│                │ └──┬───┘               
  └────┬───┘      │  │        │                │    │                   
       │          │  │ Cache  |                │ ┌──▼───┐               
    Normalize     │  │        │                │ │      │               
       │          │  │        │                │ │ SBOM │               
┌──────▼─────┐    │   \______/                 │ │      │               
│ Temp Files │    │      │                     │ └──┬───┘               
│ ┌───┐      │    │      │             ______  │    │                   
│ │VEX│      │    │      │            /      \ │    │ ┌─────────┐       
│ │ ┌───┐    │    │ ┌────▼────┐      │\______/││    └─►         │       
│ └─│OSV│    │    │ │         │      │        ││      │  Grype  ┼─►Vulns
│   │ ┌───┐  ├────┤►│ Grype-db│─────►│ SqlLite├│──────►  -----  │       
│   └─│...│  │    │ │ ------- │      │        ││      └─────────┘       
│     │   │  │    │ └─────────┘      │        ││                        
│     └───┘  │    │                   \______/ │                        
└────────────┘    └────────────────────────────┘                        
```
---

## Run Single Provider
- In vunnel, `make dev -- provider="<your provider name>"` runs specified provider and add results to `vunnel/build/vulnerability.db`.
- To store vunnel results in the grype-db:
    - clone vunnel, checkout proper branch, and run `uv sync --all-extras --dev`
    - run `uv pip install -e path/to/directory/vunnel` in grype-db to install vunnel
    - run `uv run vunnel run <your provider name>` to pull `<your provider name>` data
    - run `go run ./cmd/grype-db build -v -s 6` to update db with pulled data
```
                                          Scan Targets┌──────┐             
┌──────────────────────────────────────────────────┐  │Image │             
│   Security Feeds                                 │  │  ┌───┴──┐          
│   ┌──────────┐                                   │  │  │Files │          
│   │Chainguard│                                   │  └──│ ┌──────┐        
│   │  OpenVEX │                                   │     │ │PURL  │        
│   └────┬─────┘                                   │     └─│ ┌──────┐      
│        │                                         │       │ │...   │      
│    ┌───▼────┐                                    │       └─│      │      
│    │ Vunnel │                                    │         └──────┘      
│    │ ------ │                                    │        │              
│    └───┬────┘                                    │     ┌──▼───┐          
│        │                                ______   │     │      │          
│ ┌──────▼──────┐                        /      \  │     │ Syft │          
│ │ Temp Files  │      ┌─────────┐      │\______/│ │     │ ---- │          
│ │┌────┐       │      │         │      │        │ │     └──┬───┘          
│ ││Open│       │─────►│ Grype-db│─────►│ SqlLite├─┼─┐   ┌──▼───┐          
│ ││ ┌──┴─┐     │      │ ------- │      │        │ │ │   │ SBOM │          
│ │└─┤VEX │     │      └─────────┘      │        │ │ │   │      │          
│ │  │ ┌──┴──┐  │                        \______/  │ │   └──┬───┘          
│ │  └─┤State│  │                                  │ │  ┌───▼─────┐        
│ │    │ments│  │                                  │ │  │         │        
│ │    └─────┘  │                                  │ │  │  Grype  ┼──►Vulns
│ └─────────────┘                                  │ └──►  -----  │        
└──────────────────────────────────────────────────┘    └─────────┘        
```
---

## Use Grype With New Database
- Copy the built database to your grype cache using `grype db import path/to/grype-db/build/vulnerability.db`
- Run Grype against your target with `GRYPE_DB_AUTO_UPDATE="false" grype your-target`
    - NOTE: If you run grype _without_ `GRYPE_DB_AUTO_UPDATE`, it will reset the database to upstream version.
- To use a dev version of grype:
    - clone grype and checkout appropriate branch
    - run `GRYPE_DB_AUTO_UPDATE="false" go run cmd/grype/main.go your-target`
---

## Implementing a new provider
- Create new directory in `vunnel/src/vunnel/providers` for your new provider
- Create `__init__.py` file in directory to pull security feed and store files using `self.results_writer()` in format matching an unmashaller in `grype-db/pkg/provider/unmarshal`
    - `vunnel/src/vunnel/providers/chainguard-libraries` is a good example
- A processor in `grype-db/pkg/process/processors` reads this data to load the SQLite datastore.
```

 ┌──────────┐
 │  Vunnel  │
 │  ------  │
 │┌────────┐│   ┌────────┐
 ││  Your  ││   │YourFeed│
 ││Provider◄┼───┤        │
 │└───┬────┘│   └────────┘
 └────┼─────┘
      │
┌─────▼─────┐
│Local Files│
│ ┌───┐     │
│ │F1 │     │    ┌─────────────────────────┐     ______
│ │ ┌───┐   │    │         Grype-db        │    /      \
│ └─│F2 │   │    │         -------         │   │\______/│
│   │ ┌───┐ │    │┌─────────┐   ┌─────────┐│   │        │
│   └─│...├─┼────┼►Unmarshal┼───►Transform┼┼──►│ SqlLite│
│     │   │ │    │└─────────┘   └─────────┘│   │        │
│     └───┘ │    └─────────────────────────┘   │        │
└───────────┘                                   \______/
```
---

## Updating Integration Tests
- Vunnel integration tests compare the locally built DB and nightly cache for regressions
- Make sure the grype db is updated with `grype db update`
- Edit `tests/quality/.yardstick.yaml` to set `default_max_year: 2025`. This is required for some providers to work since our tests are for new dates.
- Setup the vulnerability match labels git submodule with `git submodule update --init --recursive` and `make update-labels`
- Edit `tests/quality/config.yaml` to add your provider and test images
- Run tests in `vunnel/tests/quality` directory with commands `make capture provider=<your provider name>` and `make validate provider=<your provider name>` This should produce output of the form (example using `chainguard-libraries`):
```
   Results used for image ghcr.io/chainguard-images/scanner-test@sha256:046b2c1b2df1e496eb3abf9e21224ffd879d92abc2bd57401456764d9aa8ddf1:
    ├── ebcf25fb-b8b6-4187-aaed-6ba9dd9079c8 : grype[custom-db]@v0.99.1-14-g8a04a093 (custom-db)  against ghcr.io/chainguard-images/scanner-test@sha256:046b2c1b2df1e496eb3abf9e21224ffd879d92abc2bd57401456764d9aa8ddf1
    └── 1b119bc3-df15-4237-a546-508ddd280608 : grype[reference]@v0.99.1-14-g8a04a093 (reference)  against ghcr.io/chainguard-images/scanner-test@sha256:046b2c1b2df1e496eb3abf9e21224ffd879d92abc2bd57401456764d9aa8ddf1
Deltas for ghcr.io/chainguard-images/scanner-test@sha256:046b2c1b2df1e496eb3abf9e21224ffd879d92abc2bd57401456764d9aa8ddf1:
Match differences between tooling (with labels):
   TOOL PARTITION                              PACKAGE              VULNERABILITY        LABEL      COMMENTARY
   grype[custom-db]@v0.99.1-14-g8a04a093 ONLY  aiohttp@3.9.1+cgr.2  GHSA-9548-qrrj-x5pj  (unknown)
   grype[reference]@v0.99.1-14-g8a04a093 ONLY  aiohttp@3.9.1+cgr.2  GHSA-5h86-8mv2-jq9f  (unknown)
   grype[reference]@v0.99.1-14-g8a04a093 ONLY  aiohttp@3.9.1+cgr.2  GHSA-5m98-qgg9-wh84  (unknown)
```
- Persist your test in the [vulnerability-match-labels repo](https://github.com/anchore/vunnel/blob/main/DEVELOPING.md#7-open-a-vulnerability-match-labels-repo-pr-to-persist-the-new-labels) by running `uv run yardstick label explore <uuid>` for each of the above UUIDs
    - yardstick may show vulnerabilities beyond the output by validate, as validate only shows the differences from the expected integration test output
- 

```
Scan Targets
 ┌──────┐
 │Image │
 │  ┌──────┐
 │  │Files │
 └──│ ┌──────┐     ┌──────┐
    │ │PURL  │     │      │
    └─│ ┌──────┐   │ Syft │
      │ │...   ┼───► ---- │
      └─│      │   └──┬───┘
  _____ └──────┘      │
 /     \           ┌──▼───┐
│\_____/│          │      │
│       │          │ SBOM │
│CacheDB├──┐       │      │
│       │  │       └──┬───┘
│       │  │     ┌────▼────┐   ┌────────┐
 \_____/   └─────►         ┼──►│Expected├─┐
  _____          │  Grype  │   └────────┘ │
 /     \         │  -----  │   ┌────────┐ ├─►Diff
│\_____/│  ┌─────►         ├──►│ Actual ├─┘
│       │  │     └─────────┘   └────────┘
│LocalDB├──┘
│       │
│       │
 \_____/
```
