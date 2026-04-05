## Espresso Making Guide

Espresso extracts first the tasty acids, then the sugars, then the bitter flavors.
Goal is to extract as much of the first two and as little of the latter.
Espresso has a very tightly defined spec by the [SCA](https://sca.coffee/sca-news/25-magazine/issue-3/defining-ever-changing-espresso-25-magazine-issue-3)
To make a good beverage in this spec, we can adjust the grind size, the extraction time, and the input bean size
We should always adjust variables one at a time in the following order:
- Adjust grind size first to get an extraction rate that results in the appropriate amount (27-36g) in the appropriate time (20-30s)
- Adjust extraction time to adjust output amount or flavoring
- Only adjust the bean input amount once the other two variables have been exhausted

```mermaid
---
config:
      theme: redux
---
flowchart TD
    A(["measure 18g of coffee beans (16–20g range)"])
    B(["Grind at appropriate size"])
    C(["Use WDT, level, and tamp"])
    D(["Extract for 25 seconds (20–30s range)"])
    F(["– Increase Grind Size<br>– Decrease Extraction Time<br>– Increase mass bean input"])
    G(["– Decrease Grind Size<br>– Increase Extraction Time<br>– Decrease mass bean input"])
    I(["– Decrease Grind Size<br>– Increase Extraction Time<br>– Decrease mass bean input"])
    J(["– Increase Grind Size<br>– Decrease Extraction Time<br>– Increase mass bean input"])
    K(["Good Work!"])
    A --> B
    B --> C
    C --> D
    D --> E{"Is output within 1:1.5<br>or 1:2 rato (27–36g)"}
    E -->|"Too Much Output (>36g)"| F
    E -->|"Too Little Output (<27g)"| G
    E -->|In Range| H{"How does it taste"}
    H -->|Bland| I
    H -->|Bitter| J
    H -->|Spot On| K
```