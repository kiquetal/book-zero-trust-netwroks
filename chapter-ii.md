### Managing Trust

Trust delegation is important because it allows us to build automated systems that can grow to a large scale and operate in a secure and trusted way with minimal human intervention.
Concepts to be familiar with: trust chain and trust anchor.

- **Trust anchor**: the authoritative source of trust from which all trust in the system is derived (the root of the trust chain). In PKI, this is the root certificate authority.
- **Trust chain**: the delegated path from the trust anchor down to the entity being validated. Each link vouches for the next.

```
                 Trust Chain / Delegation
                 ------------------------

        [ Trust Anchor ]        <- Root CA (self-signed, offline)
               |
               | signs
               v
        [ Intermediate CA ]     <- delegated authority
               |
               | signs
               v
        [ Leaf / End-entity ]   <- server, service, workload cert
               |
               | presents cert
               v
        [ Verifier / Relying party ]
        walks the chain back up to a trusted anchor
```

Threat Models
A threat model enumerates the potential attackers, their capabilities, resources, and their intended targets.
Some common threat models are:
- STRIDE — Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege
- DREAD — Damage, Reproducibility, Exploitability, Affected users, Discoverability
- PASTA — Process for Attack Simulation and Threat Analysis
- TRIKE — a risk-based threat modeling methodology
- VAST — Visual, Agile, and Simple Threat modeling
- MITRE ATT&CK — a knowledge base of adversary tactics, techniques, and procedures (TTPs)

Zero Trust adopts a threat model that assumes the network is already compromised: it does not trust the local network, and it treats every actor as a potential attacker until proven otherwise.

Least privilege is the minimum set of permissions that an entity needs to accomplish an action.

### Public Key Infrastructure (PKI)

Regarding PKI (Public Key Infrastructure), it is preferred to use private PKI over public PKI, but public PKI is better than none.

Key PKI components:
- **Certificate Authority (CA)**: issues and signs certificates.
- **Registration Authority (RA)**: verifies identity before a certificate is issued.
- **Certificate Signing Request (CSR)**: request submitted to the CA to obtain a signed certificate.
- **X.509**: the standard defining the format of public key certificates.
- **Certificate revocation**: handled via CRL (Certificate Revocation List) or OCSP (Online Certificate Status Protocol).

```
              PKI Issuance Flow
              -----------------

   [ Entity ] --generate keypair--> (private key stays local)
        |
        | build CSR (public key + identity)
        v
   [ Registration Authority (RA) ] --verify identity--> OK
        |
        v
   [ Certificate Authority (CA) ] --sign--> [ X.509 Certificate ]
        |
        v
   Entity installs signed cert, presents it during TLS handshake
```

### Private PKI vs Public PKI

A **public PKI** chains up to trust anchors (root CAs) that ship pre-installed in operating systems and browsers (Let's Encrypt, DigiCert, etc.). A **private PKI** uses your own root/intermediate CA that only your systems are configured to trust.

Zero Trust prefers **private PKI** because it gives you full control over the trust anchor, issuance policy, naming, lifetimes, and revocation — none of which you control with a public CA.

Why private PKI is preferred:
- **Control of the trust anchor**: you decide who can issue, for what names, and for how long.
- **Automation friendly**: short-lived certs and rapid rotation without rate limits or ACME external dependencies.
- **Private namespaces**: internal service names (e.g. `svc.cluster.local`) can't get public certs anyway.
- **No third-party trust dependency**: you are not exposed to a public CA misissuing certs for your names.

Risks and trade-offs of private PKI:
- **Root key compromise is catastrophic**: whoever holds the root private key can mint trusted certs for anything. The root CA must be kept **offline / air-gapped** and issuance delegated to intermediates.
- **Operational burden**: you own rotation, revocation (CRL/OCSP), monitoring, and distribution of the root to every client.
- **Trust distribution problem**: every workload must be bootstrapped with the root CA bundle; a mistake here breaks TLS everywhere.
- **No external transparency**: public PKI has Certificate Transparency (CT) logs; private PKI relies on your own auditing.

Risks of relying on public PKI in a Zero Trust network:
- You depend on the CA's issuance and revocation practices.
- Rate limits and validation flows make very short-lived, high-volume automated issuance harder.
- You cannot issue certs for private/internal names.

Rule of thumb from the book: **private PKI > public PKI > no PKI**. The worst option is skipping PKI and falling back to network-location trust.

### Relation to Kubernetes (k8s)

Kubernetes is essentially a **private PKI in action** — it ships with its own CA and issues certs to every control-plane and node component. This makes it a concrete example of the concepts above.

```
                 Kubernetes Trust Model
                 ----------------------

        [ cluster Root CA ]  (/etc/kubernetes/pki/ca.crt)
                 |
     +-----------+------------------------+
     | signs                              | signs
     v                                    v
 [ API server cert ]              [ kubelet client certs ]
 [ etcd peer/client certs ]       [ controller-manager ]
 [ front-proxy CA ]               [ scheduler, admin.conf ]

  Nodes join via CSR:
  kubelet -> CertificateSigningRequest -> approved -> signed by cluster CA
```

How the PKI concepts map onto k8s:
- **Trust anchor** = the cluster CA (`ca.crt`). Every component trusts certs chaining to it.
- **CSR flow** = the `certificates.k8s.io` API. A kubelet submits a `CertificateSigningRequest`; it is approved (manually or by the CSR approver) and signed by the cluster CA. This is the RA + CA flow made native.
- **Least privilege** = RBAC. Certificates authenticate *who* you are (identity, via the cert's Common Name / Organization → user / group); RBAC decides *what* you may do.
- **Short-lived certs & rotation** = kubelet certificate rotation; the same "automate issuance, rotate often" principle Zero Trust wants.
- **mTLS everywhere** = a service mesh (Istio, Linkerd) or SPIFFE/SPIRE adds a workload-identity PKI on top, issuing short-lived SVID certs to pods so every pod-to-pod call is mutually authenticated. This is Zero Trust's "trust the workload, not the network" applied inside the cluster.

The main private-PKI risk shows up directly in k8s: **whoever can read the cluster CA key, or can approve arbitrary CSRs, can impersonate any component in the cluster** — which is why CA key material and CSR-approval permissions are among the most sensitive things to protect.

### Trust Score

Regarding the trust score: Instead of defining binary policy decisions assigned to specific actors in the network, a Zero Trust network will continuously monitor the actions of an actor on the network to update their trust score. This score can then be used to define policy in the network based on the severity of a breach of that trust.

The trust score is fed by signals such as historical behavior, device posture, and threat intelligence. Because scoring is continuous rather than binary, it enables adaptive, risk-based access decisions.

```
        Continuous Trust Scoring Loop
        -----------------------------

   [ Actor action ] --> [ Monitor / collect signals ]
                                |
                                v
                     [ Update trust score ]
                                |
                                v
                     [ Policy engine evaluates ]
                                |
              +-----------------+-----------------+
              | high trust                        | low trust
              v                                   v
        [ Allow / full access ]        [ Step-up auth / deny / quarantine ]
```
