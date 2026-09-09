### Context-Aware Agents

This marriage of user and device is a new concept that Zero Trust introduces, which we are calling an agent.

An agent is a combination of data known about the actors in a request. This typically consists of a user (also known as the subject), a device (an asset used by the subject to make the request), and an application. These entities have been authorized separately, but Zero Trust networks recognize that policy is best captured as a combination of all participants in a request.

Trust score systems can evaluate each request in the network, using that activity feed to update the trust scores of users, applications, and devices.

Agents are not for authentication; agents serve solely as authorization components and do not play any part in authentication. Devices can be authenticated with X.509, while users might be authenticated through a traditional multifactor approach.

Authentication is session-oriented, but in the case of authorization, it is best to be request-oriented. For that reason, caching for authorization is not recommended.

Successful authentication is the act of proving one's identity to a remote system. That verified identity is then used to determine if the user actually has the rights to access the resource in question. If access must be revoked, updating authorization is more effective than changing authentication credentials.