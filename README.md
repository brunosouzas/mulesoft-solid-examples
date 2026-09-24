# SOLID in MuleSoft — runnable examples

Before-and-after examples of the five SOLID principles applied to Mule 4 flows. Each principle has a **before** endpoint that breaks the principle and an **after** endpoint that follows it, and MUnit tests show where the two behave the same and where the "before" version causes problems.

This repository accompanies the article series [SOLID in MuleSoft](https://brunosouzas.com/blog/solid-in-mulesoft-the-art-of-designing-evolutionary-integrations) (also available [in Portuguese](https://brunosouzas.com/pt/blog/solid-em-mulesoft)).

The app needs no external systems: ERPs, databases and payment providers are simulated with synthetic data, so it runs anywhere.

## The examples

| Principle | Scenario | Before | After | Article |
|---|---|---|---|---|
| **S**ingle Responsibility | Process an order | One flow validates, prices and calls the ERP | The flow orchestrates; each step is a focused sub-flow | [SRP](https://brunosouzas.com/blog/single-responsibility-principle-srp-in-mulesoft-structuring-flows-with-focused-responsibilities) |
| **O**pen/Closed | Take a payment | A choice router that must be edited for every new method | The router looks up the implementation in config; `pix` was added without touching it | [OCP](https://brunosouzas.com/blog/openclosed-principle-ocp-in-mulesoft-apis-and-flows-that-evolve-without-breaking) |
| **L**iskov Substitution | Read an order from the legacy or the new ERP | The new ERP adapter looks compatible but changes values and error behaviour, so the consumer silently takes wrong decisions | Both adapters honour the whole contract (shape, values, errors) and can replace each other | [LSP](https://brunosouzas.com/blog/liskov-substitution-principle-lsp-in-mulesoft-ensuring-consistency-and-substitutability) |
| **I**nterface Segregation | Read an order | One endpoint returns everything to everyone, including internal cost data | Mobile and finance each get an interface shaped for their need | [ISP](https://brunosouzas.com/blog/interface-segregation-principle-isp-in-mulesoft-specific-interfaces-for-specific-needs) |
| **D**ependency Inversion | Read an order and decide if it can be cancelled | The business rule knows the legacy ERP's field names and codes | The rule depends on an "order repository"; in-memory and ERP adapters are swapped in config | [DIP](https://brunosouzas.com/blog/dependency-inversion-principle-dip-in-mulesoft-rely-on-abstractions-not-implementations) |

## Layout

```
src/main/mule/
  global.xml              HTTP listener, properties, error handler (400/404 as JSON)
  srp/ ocp/ lsp/ isp/ dip/
    <principle>-before.xml
    <principle>-after.xml
src/main/resources/config/app.yaml   port, OCP payment routing, DIP repository choice
src/test/munit/<principle>-test-suite.xml
docs/contracts/          RAML types for the LSP and ISP contracts
requests.http            sample requests for every endpoint
```

## Run

Requirements: Java 17, Maven 3.9+, Mule runtime 4.9 (Anypoint Studio or a standalone runtime).

- **Anypoint Studio:** import the project (File → Import → Anypoint Studio project from file system) and run it. The app listens on port `8081`.
- **Try it:** open `requests.http` in VS Code (REST Client) or IntelliJ and send the requests, or use curl:

```bash
curl -s -X POST localhost:8081/lsp/before/quotes -H 'Content-Type: application/json' \
  -d '{"orderId":"ORD-1","orderType":"express","price":50,"quantity":2}'
```

## Test

```bash
mvn clean test
```

MUnit runs on the Mule Enterprise runtime, so Maven needs access to the MuleSoft Enterprise repository (credentials in `~/.m2/settings.xml`, as in any MuleSoft project). The build fails below 80% application coverage; the current suites cover 36 cases at about 95%.

## Notes

- The OCP router uses a dynamic `flow-ref`. MUnit only loads flows that are referenced statically, so the OCP suite references the payment implementations in a `before-suite`. Real applications do not need this.
- Everything here is intentionally small. The point is the shape of the flows, not the business logic.

## Licence

[MIT](LICENSE) © Bruno Pinto de Souza
