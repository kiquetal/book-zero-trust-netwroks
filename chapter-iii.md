### Context-Aware Agents

This marriage of user and device is a new concept that Zero Trust introduces, which we are calling an agent.

An agent is a combination of data known about the actors in a request. This typically consists of a user (also known as the subject), a device (an asset used by the subject to make the request), and an application. These entities have been authorized separately, but Zero Trust networks recognize that policy is best captured as a combination of all participants in a request.

- **Agent**: an ephemeral, request-time combination of user + device + application context. It is not a stored record but a view assembled per request.
- **Subject**: the user identity making the request.
- **Asset**: the device used to make the request.
- **Application**: the software acting on the subject's behalf.

```
                 What Makes an Agent
                 -------------------

        [ User / Subject ]   authenticated (MFA, password + OTP...)
                 +
        [ Device / Asset ]   authenticated (X.509 device cert)
                 +
        [ Application ]      identified
                 =
        ============ AGENT ============
        (assembled per request, used for AUTHORIZATION only)
```

Trust score systems can evaluate each request in the network, using that activity feed to update the trust scores of users, applications, and devices.

### Agents Are for Authorization, Not Authentication

Agents are not for authentication; agents serve solely as authorization components and do not play any part in authentication. Devices can be authenticated with X.509, while users might be authenticated through a traditional multifactor approach.

- **Authentication (AuthN)**: proving *who* you are.
- **Authorization (AuthZ)**: deciding *what* you may do.

The agent is formed *after* the individual entities authenticate, and is consulted only to make authorization decisions.

```
        Authentication vs Authorization
        -------------------------------

   User  --MFA----------------+
   Device --X.509 cert--------+--> [ each entity AUTHENTICATED separately ]
   App   --identified---------+
                                        |
                                        v
                              [ assemble AGENT ]
                                        |
                                        v
                              [ AUTHORIZATION decision ]
                              (per request, not cached)
```

### Session vs Request Orientation

Authentication is session-oriented, but in the case of authorization, it is best to be request-oriented. For that reason, caching for authorization is not recommended.

- **Session-oriented (authentication)**: prove identity once, reuse for the session's lifetime.
- **Request-oriented (authorization)**: re-evaluate on every request so a change in trust score or policy takes effect immediately.
- **Why not cache authZ**: a cached "allow" would keep granting access after trust has dropped or access was revoked. Fresh evaluation is what makes revocation fast.

```
   Authentication (session-oriented)      Authorization (request-oriented)
   ---------------------------------      --------------------------------
   login once ---> [ session ]            req1 -> evaluate -> allow/deny
                     |  |  |               req2 -> evaluate -> allow/deny
                   reused across           req3 -> evaluate -> allow/deny
                   many requests           (no caching; every request judged)
```

### Revocation

Successful authentication is the act of proving one's identity to a remote system. That verified identity is then used to determine if the user actually has the rights to access the resource in question. If access must be revoked, updating authorization is more effective than changing authentication credentials.

Why authorization is the better revocation lever:
- Rotating credentials (rekeying, re-issuing certs) is slow and disruptive, and the old credential may stay valid until it propagates/expires.
- Flipping an **authorization** policy takes effect on the very next request (because authorization is request-oriented and uncached).
- This is the practical payoff of not caching authZ: revocation is near-instant.

### Relation to Kubernetes (k8s)

The AuthN/AuthZ split and the "revoke via authorization" principle map cleanly onto Kubernetes:

```
        Kubernetes AuthN -> AuthZ Pipeline
        ----------------------------------

   Request to API server
        |
        v
   [ Authentication ]   client cert (X.509) / OIDC token / service-account token
        |   identity = user + groups (or service account)
        v
   [ Authorization ]    RBAC / ABAC / Webhook  -> allow or deny
        |
        v
   [ Admission control ] -> mutate / validate -> persist
```

- **Device auth with X.509** maps to k8s client-certificate authentication; **user MFA** maps to an OIDC identity provider fronting the cluster.
- **Authorization is request-oriented**: the API server runs RBAC checks on *every* API call — it does not cache an "allow" decision, exactly matching the book's guidance.
- **Revoke via authorization**: to cut off access fast, you remove a RoleBinding/ClusterRoleBinding (an authorization change) rather than trying to rotate the user's certificate — which cannot be revoked easily and stays valid until expiry. This is the concrete reason k8s security guidance favors short-lived certs plus RBAC removal for revocation.
- **Agent concept**: a service mesh with SPIFFE identities effectively builds an "agent" per workload (identity + pod/node context) that the mesh authorization policy evaluates per request.
