# Explainer for ActivityPub Server-to-Service Content Delivery

## Introduction

This proposal aims to solve an underspecified area of ActivityPub: the delivery of public content to 3rd party ActivityPub services. When users post “public” content in ActivityPub, it does not get broadcast publicly, or automatically made available to other ActivityPub services. To make users' content more discoverable or searchable, server admins need a standardised mechanism to deliver that content more broadly.

## Background

Content discoverability and search services, while not critical to all ActivityPub users, can be helpful to those who want to increase visibility and audience reach. ActivityPub operates fundamentally on a push model, so ActivityPub discovery/search services often require content to be delivered to them. While ActivityPub users, or servers, can address their content to “public” and set flags on their actor objects (e.g, indexable and discoverable), this doesn’t lead to presence on these discovery services.

Users can make use of ActivityPub addressing to deliver their content to specific services they choose. This is often implemented by service actors that users can follow, who will automatically follow them back (e.g, Bridgy Fed, tags.pub) By being added to the user’s followers list, the services will receive future content and updates. This flow is well established in the ecosystem, and uses delivery mechanisms outlined in the ActivityPub spec. However, this approach faces significant scalability and maintenance challenges, as it requires individual users to manually manage connections across a growing and changing ecosystem of services.

Server admins, as part of their responsibilities, control server federation. This includes the selection of which ActivityPub services to integrate with, and deliver content to, a practice already adopted by many server administrators. Where it makes sense for their users, server admins may deliver content to ActivityPub services for things like discoverability and search. Admins sometimes also choose to integrate with 3rd party ActivityPub services to assist with server management. Whilst not every service integration should be managed at the server level, this does help to scale the work of selecting and maintaining which ActivityPub services to integrate with.

However, while the user directed content delivery is well standardised, there is no standardised mechanism in ActivityPub for this server-to-service content delivery. This has led to the development of multiple diverging mechanisms, with user control and privacy limitations.

## Proposal

We propose to standardise delivery of server-wide public content to ActivityPub services, using the existing defined Activities and delivering those to a service actor’s inbox. Server admins can maintain lists of ActivityPub service actors (or their inboxes), that all content created on their server and addressed to the special “public” collections will be delivered to. The ActivityPub specification for server to server delivery ([7.1 Delivery](https://www.w3.org/TR/activitypub/#delivery)) currently limits delivery to only specified recipients:

> “If the object is just sent to the "public" collection the object is not delivered to any actors but is publicly viewable in the actor's outbox.”

This should be expanded for servers to also deliver content to some other ActivityPub servers/services, at their discretion, when content is addressed to the “public” collection.

Many ActivityPub servers already support this as part of their implementations of relay protocols ([FEP-ae0c: Fediverse Relay Protocols: Mastodon and LitePub](https://helge.codeberg.page/fep/fep/ae0c/)), including Mastodon, Misskey, Pleroma, and many of their forks. When these servers deliver content as per the audience targeting on the object, they also deliver it to additional ActivityPub service inboxes if the content is addressed to the “public” collection. This proposal is only to standardise the initial server-to-service delivery used within these relay protocols, that can apply to any general ActivityPub service, and is not limited to those providing relay and redistribution. Use of this delivery component beyond typical relay services can already be seen in the ecosystem (e.g, [tags.pub](https://tags.pub/), which can be connected to via servers relay settings, but won’t redistribute content directly).

While this server-to-service content delivery approach doesn’t mandate how to enforce user controls, it does allow for standard delivery controls to be used. For example servers could choose not to deliver content to a service when the user has blocked that service actor.

## Out of scope

This proposal does not:

- call for the standardisation of the full relay protocols (as outlined in [FEP-ae0c](https://helge.codeberg.page/fep/fep/ae0c/)). Those protocols target specific ActivityPub services designed for content redistribution, and by doing so the relay protocols cover a much larger scope than the server-to-service delivery focus of this proposal.  
- dictate how servers manage the lists of service actors/inboxes to deliver content to. Existing implementations already manage this in multiple different ways.  
- intend to change user directed delivery. Server-to-service content delivery does not conflict with nor replace user directed flow (e.g, users adding services to their follower lists).  
- define what user controls should be available and respected for server-to-service content delivery. While this is also important to get right, most existing implementations don’t provide these controls to users at all, so there is further work to be done before standardising this component.

## Alternatives Considered

### FASPs

[Federated Auxiliary Service Providers](https://github.com/mastodon/fediverse_auxiliary_service_provider_specifications) is a more recently developed approach for integration between an ActivityPub server and service. One of the existing FASP “capabilities” (discovery data-sharing) provides servers a mechanism for this server-to-service delivery of content. With this enabled, both existing (backfill) and new/altered content can be shared with an ActivityPub service. This is done over a bespoke channel that provides ids of content. This includes both public content that has been authored on this server, as well as other public content this server receives from 3rd parties. The service is then expected to perform a separate fetch using those id’s in order to receive the full content.

The FASP data-sharing protocols allow for the server to limit outgoing content based on discoverable and indexable flags, depending on the services indicated usage. However, just as with the relay protocol implementations, once content is addressed to the special “public” collections, there are no user controls to opt-out of delivery to ActivityPub services (either altogether, or individually).

The enforcement of user and host server controls (e.g, blocks and defederation) is handled on the subsequent fetch, done from the ActivityPub service to the host server. While ActivityPub spec contains authorization controls, which are available on many implementations, some servers don’t have this activated due to conflicting priorities (e.g, caching). If this isn’t enabled, then even if the ActivityPub service signs its request, it has no way of knowing if the returned content has respected the user controls. Even for servers that verify fetch signatures, as the FASP protocol shares content from 3rd parties, ActivityPub servers may not realise their content is being shared with a new service, and may not know to restrict that new service’s actor/domain in advance.

FASP protocols also offer other capabilities, for services that need integration beyond server-to-service content delivery.

### Platform APIs

Some ActivityPub platforms offer custom APIs to retrieve public content-feeds from their servers. The implementations of these are often using bespoke API endpoints, data formats and subscription controls. This means that ActivityPub services wishing to support multiple platforms may require multiple custom integrations. There are often additional API’s for services that need integration beyond server-to-service content delivery.

### Centralised redistribution

Relays (like those outlined in [FEP-ae0c](https://helge.codeberg.page/fep/fep/ae0c/)) have explored redistribution of content to 3rd party ActivityPub servers and services. ActivityPub services could participate in new or existing relays to receive public content. Relying on relays can make it opaque to users and server admins, where their content will be delivered to. There are also no standardised mechanisms, that are compatible with relay redistribution, to control (or signal) which ActivityPub services may, or may not, use this content.

### Public collections

Special “public” collection ids are already used extensively within ActivityPub. Services that wish to receive public content could be listed within a public collection. The existing [https://www.w3.org/ns/activitystreams\\\#Public](https://www.w3.org/ns/activitystreams\\#Public) collection id has significant established social norms, so altering it would be unwise. However, other special “public” collections (either centralised or per server) could be created and used by ActivityPub servers when users post publicly. While this would require minor changes to the addressing set when users post publicly, it could then use standardised outgoing delivery mechanisms and existing controls upon those. This also provides simple ways for post level controls by including/excluding these additional “public” collections.