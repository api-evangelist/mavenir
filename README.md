# Mavenir (mavenir)

Mavenir is a US-headquartered (Richardson, Texas) cloud-native telecom network software vendor that builds the software operators run rather than a network or a developer platform of its own. Its portfolio spans Open RAN and vRAN (MAVair), converged packet core and cloud-native IMS (MAVcore), messaging, fraud and security applications, the Mavenir Digital Enablement (MDE) BSS and converged charging stack (MAVapps), private networks, MEC and an Intelligent IoT Platform (MAVedge). In the telecom value chain Mavenir sits upstream of the carrier — it supplies the Combo NEF/SCEF network-exposure function and the BSS that a mobile operator uses to expose and monetize CAMARA and GSMA Open Gateway network APIs, but it does not expose those APIs to developers itself.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/mavenir/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/mavenir/refs/heads/main/apis.yml)

## API Posture — Honest Summary

There is **no Mavenir developer portal**. `developer.mavenir.com`, `developers.mavenir.com`, `docs.mavenir.com`, `api.mavenir.com`, `apis.mavenir.com`, `opengateway.mavenir.com`, `nef.mavenir.com` and `exposure.mavenir.com` all fail to resolve. `/developer`, `/developers`, `/api` and `/open-gateway` on the corporate site all return 404. The full sitemap (86 pages, 360 press releases, 194 resources) contains zero developer or API-reference pages. The only live gated surface, `support.mavenir.com`, 307-redirects to an ADFS SSO login wall for existing customers.

Mavenir hosts **no downloadable specification**. `/openapi.json`, `/swagger.json` and `/api-docs` all 404, and no OpenID Connect or OAuth discovery document is served.

What *is* real and public is Mavenir's standards work.

## CAMARA Posture

**Active CAMARA contributor and co-maintainer, not a CAMARA API exposer.** This is not a press release — it is code and governance in the `camaraproject` GitHub organization:

- Eleven Mavenir individuals are listed in [camaraproject/Governance PARTICIPANTS.MD](https://github.com/camaraproject/Governance/blob/main/PARTICIPANTS.MD).
- Mavenir appears in the [CAMARA landscape](https://github.com/camaraproject/camara-landscape/blob/main/landscape.yml) as a **Solution Provider / Network Capability Solution Provider**.
- Two Mavenir engineers are listed in [camaraproject/WebRTC MAINTAINERS.MD](https://github.com/camaraproject/WebRTC/blob/main/MAINTAINERS.MD), and `@pradeepachar-mavenir` is a default **CODEOWNER** of the whole repository.
- The [CAMARA API Backlog](https://github.com/camaraproject/APIBacklog/blob/main/documentation/APIbacklog.md) lists Mavenir as a supporter of the WebRTC API alongside GSMA-OGW, Orange, Telefónica, T-Mobile US and Vodafone.

CAMARA APIs with real Mavenir evidence: **WebRTC Call Handling**, **WebRTC Events / Subscriptions**, **WebRTC Registration**. No evidence was found that Mavenir exposes any of the commercial CAMARA APIs (Number Verification, SIM Swap, Device Location, Device Status, Quality on Demand, Carrier Billing, KYC Match, Scam Signal, Device Swap, Population Density) — it supplies the operators who do.

**GSMA Open Gateway:** Mavenir is not an operator and is not on the GSMA operator list. Its 2024-12-18 Spry Fox Networks release states that the partner's QP Cloud MONET is "the first Channel Partner Solution to become certified with Open Gateway API Certification" and that "Mavenir's Combo NEF/SCEF ... provides exposure of standard defined network APIs to third-party applications via QPCM." The certification belongs to the partner. No Aduna relationship was found.

## TM Forum Open API Conformance

| Level | Product | APIs | Date |
| --- | --- | --- | --- |
| Platinum | Mavenir Digital Enablement (MDE) BSS | 20+ certified Open APIs (numbers not itemized in the announcement) | 2023-09-20 |
| Silver | Cloud-native BSS suite | TMF620 Product Catalog, TMF622 Product Ordering, TMF629 Customer Management, TMF632 Party Management, TMF667 Document Management | 2022-10-06 |

## 3GPP Exposure

Mavenir sells a **Combo NEF/SCEF** (Network Exposure Function / Service Capability Exposure Function) and publishes a resource, *Monetizing 5G SA with the Network Exposure Function*, describing exposure of 5G SA capability to third-party applications with usage-based charging and tiered QoS pricing. MEC, private networks/CBRS, IIoTP and network slicing are all productized. None of it has a public API surface.

## Auth

HTTP Bearer in Mavenir's own contributed definitions; OpenID Connect in the CAMARA WebRTC definitions Mavenir co-maintains. **CIBA does not appear anywhere** — not in the Mavenir BYON specs, not in the CAMARA WebRTC specs, not on mavenir.com.

## APIs

### Mavenir BYON Call Handling API (VVoIP Service)

A Mavenir-authored OpenAPI 3.0.0 definition for Bring Your Own Number one-to-one voice and video calling over WebRTC, contributed by Mavenir (`contact@mavenir.com`, Apache 2.0) to the CAMARA API Backlog. Four operations. A contributed reference definition in a public standards repository, not a Mavenir-hosted commercial endpoint.

- **Human URL:** [CAMARA APIBacklog — BYON-CallHandling-Service.yaml](https://github.com/camaraproject/APIBacklog/blob/main/documentation/SupportingDocuments/BYON-CallHandling-Service.yaml)
- **OpenAPI:** [openapi/mavenir-byon-call-handling-openapi.yml](openapi/mavenir-byon-call-handling-openapi.yml)

### Mavenir BYON Registration and Connectivity Management (RACM) API

A Mavenir-authored OpenAPI 3.0.0 definition letting a REST client register into and manage connectivity toward an operator's IMS network. Four operations, including a push-channel registration. Also a CAMARA API Backlog supporting document.

- **Human URL:** [CAMARA APIBacklog — BYON-RACM-Service.yaml](https://github.com/camaraproject/APIBacklog/blob/main/documentation/SupportingDocuments/BYON-RACM-Service.yaml)
- **OpenAPI:** [openapi/mavenir-byon-racm-openapi.yml](openapi/mavenir-byon-racm-openapi.yml)

### Mavenir Digital Enablement (MDE) TM Forum Open APIs

The TM Forum Open APIs implemented in Mavenir's cloud-native BSS, converged charging and digital marketplace platform, certified Platinum with 20+ Open APIs. The certification is real and verifiable; Mavenir publishes no API reference, no OpenAPI download and no endpoint for them. They ship inside a licensed operator deployment.

- **Human URL:** [Mavenir Digital Enablement BSS](https://www.mavenir.com/portfolio/mavapps/mavenir-digital-enablement-bss/)

## SDKs, Postman, GraphQL, Webhooks

None. `github.com/mavenir` exposes no public repositories, there is no PyPI or npm organization, no SwaggerHub owner, no public Postman workspace, no GraphQL surface and no published event or webhook catalog.

## Tags

- Telecommunications
- United States
- Network Vendor
- Network APIs
- CAMARA
- Open Gateway
- BSS
- OSS
- TM Forum
- Open RAN
- 5G
- IMS
- Messaging
- Network Exposure Function
- Standards

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## Links

- [Website](https://www.mavenir.com/)
- [About](https://www.mavenir.com/about/)
- [Portfolio](https://www.mavenir.com/portfolio/mavapps/)
- [Newsroom](https://www.mavenir.com/newsroom/)
- [Contact](https://www.mavenir.com/contact-us/)
- [Support Portal (ADFS login required)](https://support.mavenir.com/)
- [LinkedIn](https://www.linkedin.com/company/mavenir)
