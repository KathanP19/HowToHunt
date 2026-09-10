# Reverse Engineer an API

Many products are driven by private mobile/desktop or undocumented web APIs with no published specification — but security testing of those APIs (auth flaws, hidden endpoints, business-logic abuse) starts with rebuilding their interface. This technique captures live traffic through a proxy, converts the recorded flows into an OpenAPI/Swagger specification, and turns it into a structured, testable API collection. The output becomes your attack-surface inventory for IDOR, rate-limit, and mass-assignment hunting.

## Hunting Checklist

1. Set up a local proxy on a known port (FoxyProxy to flip browser/mobile traffic on and off).
2. Run `mitmweb`, then visit `mitm.it` from the proxied device to install and trust the CA certificate.
3. Browse the target application and exercise *every* functional flow so the proxy records a representative corpus of API calls.
4. Export the recorded traffic (`File → Save all`) to a flows file.
5. Convert the flows file into a spec with `mitmproxy2swagger` (run twice: clean pass, then `--examples` pass for documentation).
6. Remove the `ignore:` markers the tool inserts so all endpoints are kept in the spec.
7. Import the spec into Swagger Editor to visualize and audit every endpoint, verb, and parameter.
8. Import the same spec into Postman to get a ready-made, organized collection for replay and fuzzing.
9. Review the collection for endpoints not referenced in the UI (hidden functions), then test authentication and object-level authorization on each.

## Test Cases & Examples

### 1. Start the proxy

```bash
# FoxyProxy: create a proxy profile pointing at 127.0.0.1:8080 (label it anything you want) and enable it
sudo mitmweb --listen-port 8080
```

Then go to `http://mitm.it` through the proxy and install/import the certificate on the testing machine or device.

### 2. Capture and export

Explore the website/app and use its API-backed functionality — mitmweb records everything. Download the captures as a flow file: click `File → Save all`.

### 3. Generate the spec

```bash
# Pass 1 — baseline spec from the recorded flows file
sudo mitmproxy2swagger -i flows -o spec.yml -p '<website api>' -f flow

# Edit spec.yml: delete the `ignore:` lines so no endpoint is dropped

# Pass 2 — same command with --examples to embed request/response examples
# (enhances the documentation of the API endpoints)
sudo mitmproxy2swagger -i flows -o spec.yml -p '<website api>' -f flow --examples
```

### 4. Visualize and test

- Import the clean `spec.yml` into [Swagger Editor](https://editor.swagger.io/) to visualize the different endpoints.
- Import the same `spec.yml` in Postman to produce a well-organized collection — then replay with a second account's credentials and diff responses (auth/authz testing).

## Bypass Techniques

- **HTTPS inspection on mobile/emulators:** mitmweb works once its CA is installed; where certificate pinning blocks interception, standard practice is to bypass pinning on a rooted device/emulator with Frida/objection — only do this on apps you are authorized to test.
- **Endpoints mitmproxy2swagger lumps together:** regex-path-normalized variables (ids, UUIDs) can hide distinct routes — check the raw flow list in mitmweb and add narrower `-p` patterns if the spec looks under-populated.
- **Partial captures:** parameters seen only in one client platform (iOS) may be missing from a desktop-only capture; record flows across every client the API serves.

## Tools & Resources

1. [FoxyProxy](https://getfoxyproxy.org/) — browser proxy switcher (port 8080).
2. [mitmweb](https://docs.mitmproxy.org/stable/tools-mitmweb/) — interactive MITM proxy with web UI.
3. [mitmproxy2swagger](https://github.com/mitmproxy2swagger/mitmproxy2swagger) — flows file → OpenAPI spec converter.
4. [Swagger Editor](https://editor.swagger.io/) — visualize/import the generated `spec.yml`.
5. [Postman](https://www.postman.com/) — import the spec as a collection for replay/fuzzing.
6. Burp Suite can replace this pipeline once the endpoints are known — use it for the actual security testing.

## Report Tips

- Frame the impact around what the rebuilt spec revealed: "undocumented API surface of N endpoints, including admin/financial functions" is the finding, not the tooling.
- Attach (redacted) spec snippets and a short replay PoC showing an endpoint performing its function with an unauthorized/other-user session.
- A reverse-engineered API is the evidence base for stronger chained findings (BOLA, missing rate limits, mass assignment) — reference this inventory in those reports so triage can reproduce quickly.
