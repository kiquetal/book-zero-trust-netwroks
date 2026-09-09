### Context Aware Agents

This marriage of user and device is a new concept that zero trust introduces, which we are calling an agent. 

An agent is a combination of data known about the actors in a request. This typically consist of a user(also known as the subject), a device(an asset used by the subject to make the request) and an application,these entities have been authorized separtely but zero trust networks recognize that policy is best captured as a combination of all paritcipants in a request.
 
Trust score systems can evaluate each request in the network, using that activity feed to update the trust scores of users, applications, and devices. 

Agents are not for authentication , agents serve solely as authorization components and do not play any part in authentication, devices can be authenticated with X509 while users might be authetnicated through a traditional multifactor approach.

Authentication is session oriented, but in the case of authorization it is best to be request oriented. For that reason the caching for authorization is not recommended

Successful authentication is the acct of proving one's identity to a remote system. That verified identity is then used to determine if the user actually has the rights to access the resource in question. If access must be revoked, updating authorization is more effective than changing authentication credentials. 
