---
stand_alone: true
ipr: trust200902
docname: draft-keranen-t2trg-rest-iot-latest
cat: info
pi:
  toc: 'yes'
  symrefs: 'yes'
  iprnotified: 'yes'
  strict: 'yes'
  compact: 'yes'
  sortrefs: 'yes'
  colonspace: 'yes'
  rfcedstyle: 'no'
  tocdepth: '4'
title: RESTful Design for Internet of Things Systems
abbrev: RESTful Design for IoT Systems
area: T2TRG
author:
 
- ins: A. Keranen
  name: Ari Keranen
  org: Ericsson
  street: ''
  city: Jorvas
  code: '02420'
  country: Finland
  email: ari.keranen@ericsson.com

- ins: M. Kovatsch
  name: Matthias Kovatsch
  org: ETH Zurich
  street: 'Universitaetstrasse 6'
  city: Zurich
  code: 'CH-8092'
  country: Switzerland
  email: kovatsch@inf.ethz.ch

normative:
  RFC3986:
  RFC7230:
  RFC7231:
  REST:
    title: Architectural Styles and the Design of Network-based Software Architectures
    author:
    - ins: R. Fielding
    date: 2000
    seriesinfo: Ph.D. Dissertation, University of California, Irvine
  RFC5246:
  RFC6347:
  I-D.ietf-core-resource-directory:
  RFC7049:
  W3C.REC-exi-20110310:
  RFC5590:
  RFC5246:
  RFC5280:
  RFC6347:
  I-D.ietf-core-object-security:
informative:
  RFC7228:
  RFC7252:
  RFC7159:
  RFC7925:
  I-D.ietf-core-senml:
  IANA-media-types:
    title: Media Types
    target: http://www.iana.org/assignments/media-types/media-types.xhtml
  IANA-CoAP-media:
    title: CoAP Content-Formats
    target: http://www.iana.org/assignments/core-parameters/core-parameters.xhtml#content-formats

--- abstract

This document gives guidance for designing Internet of Things (IoT)
systems that follow the principles of the Representational State
Transfer (REST) architectural style.

--- middle

# Introduction

The Representational State Transfer (REST) architectural style {{REST}} is a set of guidelines and best practices for building distributed hypermedia systems.
At its core is a set of constraints, which when fulfilled enable desirable properties for distributed software systems such as scalability and modifiability.
When REST principles are applied to the design of a system, the result is often called RESTful and in particular an API following these principles is called a RESTful API.

Different protocols can be used with RESTful systems, but at the time of writing the most common protocols are HTTP {{RFC7230}} and CoAP {{RFC7252}}.
Since RESTful APIs are often simple and lightweight, they are a good fit for various IoT applications.
The goal of this document is to give basic guidance for designing RESTful systems and APIs for IoT applications and give pointers for more information. 
Design of a good RESTful IoT system has naturally many commonalities with other Web systems.
Compared to other systems, the key characteristics of many IoT systems include:

* data formats, interaction patterns, and other mechanisms that minimize, or preferably avoid, the need for human interaction
* preference for compact and simple data formats to facilitate efficient transfer over (often) constrained networks and lightweight processing in constrained nodes

# Terminology {#sec-terms}

This section explains some of the common terminology that is used in the context of RESTful design for IoT systems. For terminology of constrained nodes and networks, see {{RFC7228}}.

Cache:
: A local store of response messages and the subsystem that controls storage, retrieval, and deletion of messages in it.

Client:
: A node that sends requests to servers and receives responses.

Client State:
: The state kept by a client between requests. This typically includes the "current" resource, the set of active requests, the history of requests, bookmarks (URIs stored for later retrieval) and application-specific state. (Note that this is called "Application State" in {{REST}}, which has some ambiguity in modern (IoT) systems where the overall state of the distributed application (i.e., application state) is reflected in the union of all Client States and Resource States of all clients and servers involved.)

Content Negotiation:
: The practice of determining the "best" representation for a client when examining the current state of a resource. The most common forms of content negotiation are Proactive Content Negotiation and Reactive Content Negotiation.

Form:
: A hypermedia control that enables a client to change the state of a resource or to construct a query locally.

Forward Proxy:
: An intermediary that is selected by a client, usually via local configuration rules, and that can be tasked to make requests on behalf of the client. This may be useful, for example, when the client lacks the capability to make the request itself or to service the response from a cache in order to reduce response time, network bandwidth and energy consumption.

Gateway:
: A reverse proxy that provides an interface to a non-RESTful system such as legacy systems or alternative technologies such as Bluetooth ATT/GATT. See also "Reverse Proxy".

Hypermedia Control:
: A component embedded in a representation that identifies a resource for future hypermedia interactions, such as a link or a form. If the client engages in an interaction with the identified resource, the result may be a change to resource state and/or client state.

Idempotent Method:
: A method where multiple identical requests with that method lead to the same visible resource state as a single such request. For example, the PUT method replaces the state of a resource with a new state; replacing the state multiple times with the same new state still results in the same state for the resource. However, the response from the server can be different when the same idempotent method is used multiple times. For example when DELETE is used twice on an existing resource, the first request would remove the association and return success acknowledgement whereas the second request would likely result in error response due to non-existing resource.
<!-- Too much text for terminology section? Should be separate section after 1st sentence? -->

Link:
: A hypermedia control that enables a client to navigate between resources and thereby change the client state.

Link Relation Type:
: An identifier that describes how the link target resource relates to the current resource  (see {{RFC5988}}).

Media Type:
: A string such as "text/html" or "application/json" that is used to label representations so that it is known how the representation should be interpreted and how it is encoded.

Method:
: An operation associated with a resource. Common methods include GET, PUT, POST, and DELETE (see {{sec-methods}} for details).

Origin Server:
: A server that is the definitive source for representations of its resources and the ultimate recipient of any request that intends to modify its resources. In contrast, intermediaries (such as proxies caching a representation) can assume the role of a server, but are not the source for representations as these are acquired from the origin server.

Proactive Content Negotiation:
: A content negotiation mechanism where the server selects a representation based on the expressed preference of the client. For example, in an IoT application, a client could send a request with preferred media type "application/senml+json".

Reactive Content Negotiation:
: A content negotiation mechanism where the client selects a representation from a list of available representations. The list may, for example, be included by a server in an initial response. If the user agent is not satisfied by the initial response representation, it can request one or more of the alternative representations, selected based on metadata (e.g., available media types) included in the response.

Representation:
: A serialization that represents the current or intended state of a resource and that can be transferred between clients and servers. REST requires representations to be self-describing, meaning that there must be metadata that allows peers to understand which representation format is used. Depending on the protocol needs and capabilities, there can be additional metadata that is transmitted along with the representation.

Representation Format:
: A set of rules for serializing resource state. On the Web, the most prevalent representation format is HTML. Other common formats include plain text and formats based on JSON {{RFC7159}}, XML, or RDF. Within IoT systems, often compact formats based on JSON, CBOR {{RFC7049}}, and EXI {{W3C.REC-exi-20110310}} are used.

Representational State Transfer (REST):
: An architectural style for Internet-scale distributed hypermedia systems.

Resource:
: An item of interest identified by a URI. Anything that can be named can be a resource. A resource often encapsulates a piece of state in a system. Typical resources in an IoT system can be, e.g., a sensor, the current value of a sensor, the location of a device, or the current state of an actuator.

Resource State:
: A model of a resource's possible states that is represented in a supported representation type, typically a media type. Resources can change state because of REST interactions with them, or they can change state for reasons outside of the REST model.

Resource Type:
: An identifier that annotates the application-semantics of a resource (see Section 3.1 of {{RFC6690}}).

Reverse Proxy:
: An intermediary that appears as a server towards the client but satisfies the requests by forwarding them to the actual server (possibly via one or more other intermediaries). A reverse proxy is often used to encapsulate legacy services, to improve server performance through caching, and to enable load balancing across multiple machines.

Safe Method:
: A method that does not result in any state change on the origin server when applied to a resource. For example, the GET method only returns a representation of the resource state but does not change the resource. Thus, it is always safe for a client to retrieve a representation without affecting server-side state.

Server:
: A node that listens for requests, performs the requested operation and sends responses back to the clients.

Uniform Resource Identifier (URI):
: A global identifier for resources. See {{sec-uris}} for more details.

# Basics

## Architecture

The components of a RESTful system are assigned one or both of two roles: client or server.
Note that the terms "client" and "server" refer only to the roles that the nodes assume for a particular message exchange. The same node might act as a client in some communications and a server in others.
Classic user agents (e.g., Web browsers) are always in the client role and have the initiative to issue requests.
Origin servers always have the server role and govern over the resources they host.

~~~~~~~~~~~~~~~~~~~
 ________                       _________
|        |                     |         |
| User  (C)-------------------(S) Origin |
| Agent  |                     |  Server |
|________|                     |_________|
(Browser)                      (Web Server)
~~~~~~~~~~~~~~~~~~~
{: artwork-align="center" #basic-arch-x title="Client-Server Communication"}

Intermediaries (such as forward proxies, reverse proxies, and gateways) implement both roles, but only forward requests to other intermediaries or origin servers.
They can also translate requests to different protocols, for instance, as CoAP-HTTP cross-proxies.

~~~~~~~~~~~~~~~~~~~
 ________       __________                        _________
|        |     |          |                      |         |
| User  (C)---(S) Inter- (C)--------------------(S) Origin |
| Agent  |     |  mediary |                      |  Server |
|________|     |__________|                      |_________|
(Browser)     (Forward Proxy)                    (Web Server)
~~~~~~~~~~~~~~~~~~~
{: artwork-align="center" #basic-arch-a title="Communication with Forward Proxy"}

Reverse proxies are usually imposed by the origin server.
In addition to the features of a forward proxy, they can also provide an interface for non-RESTful services such as legacy systems or alternative technologies such as Bluetooth ATT/GATT.
In this case, reverse proxies are usually called gateways.
This property is enabled by the Layered System constraint of REST, which says that a client cannot see beyond the server it is connected to (i.e., it is left unaware of the protocol/paradigm change).

~~~~~~~~~~~~~~~~~~~
 ________                        __________       _________
|        |                      |          |     |         |
| User  (C)--------------------(S) Inter- (x)---(x) Origin |
| Agent  |                      |  mediary |     |  Server |
|________|                      |__________|     |_________|
(Browser)                        (Gateway)     (Legacy System)
~~~~~~~~~~~~~~~~~~~
{: artwork-align="center" #basic-arch-b title="Communication with Reverse Proxy"}

Nodes in IoT systems often implement both roles.
Unlike intermediaries, however, they can take the initiative as a client (e.g., to register with a directory, such as CoRE Resource Directory {{I-D.ietf-core-resource-directory}}, or to interact with another thing) and act as origin server at the same time (e.g., to serve sensor values or provide an actuator interface).

~~~~~~~~~~~~~~~~~~~
 ________                                         _________
|        |                                       |         |
| Thing (C)-------------------------------------(S) Origin |
|       (S)                                      |  Server |
|________| \                                     |_________|
 (Sensor)   \   ________                     (Resource Directory)
             \ |        |
              (C) Thing |
               |________|
              (Controller)
~~~~~~~~~~~~~~~~~~~
{: artwork-align="center" #basic-arch-c title="Constrained RESTful environments"}

## System design

When designing a RESTful system, the state of the distributed application must be assigned to the different components.
Here, it is important to distinguish between "client state" and "resource state".

Client state encompasses the control flow and the interactions between the components (see {{sec-terms}}).
Following the Stateless constraint, the client state must be kept only on clients.
That is, there is no establishment of shared information about future interactions between client and server (usually called a session).
On the one hand, this makes requests a bit more verbose since every request must contain all the information necessary to process it.
On the other hand, this makes servers efficient and scalable, since they do not have to keep any state about their clients.
Requests can easily be distributed over multiple worker threads or server instances.
For the IoT systems, it lowers the memory requirements for server implementations, which is particularly important for constrained servers (e.g., sensor nodes) and servers serving large amount of clients (e.g., Resource Directory).

Resource state includes the more persistent data of an application (i.e., independent of the client control flow and lifetime).
This can be static data such as device descriptions, persistent data such as system configuration, but also dynamic data such as the current value of a sensor on a thing.

## Uniform Resource Identifiers (URIs) {#sec-uris}

An important part of RESTful API design is to model the system as a set of resources whose state can be retrieved and/or modified and where resources can be potentially also created and/or deleted.

Uniform Resource Identifiers (URIs) are used to indicate a resource for interaction, to reference a resource from another resource, to advertise or bookmark a resource, or to index a resource by search engines.

      foo://example.com:8042/over/there?name=ferret#nose
      \_/   \______________/\_________/ \_________/ \__/
       |           |            |            |        |
    scheme     authority       path        query   fragment

A URI is a sequence of characters that matches the syntax defined in {{RFC3986}}. It consists of a hierarchical sequence of five components: scheme, authority, path, query, and fragment (from most significant to least significant). A scheme creates a namespace for resources and defines how the following components identify a resource within that namespace. The authority identifies an entity that governs part of the namespace, such as the server "www.example.org" in the "http" scheme. A host name (e.g., a fully qualified domain name) or an IP address, potentially followed by a transport layer port number, are usually used in the authority component for the "http" and "coap" schemes. The path and query contain data to identify a resource within the scope of the URI's scheme and naming authority. The fragment allows to refer to some portion of the resource, such as a section in an HTML document. However, fragments are processed only at client side and not sent on the wire. {{?RFC7320}} provides more details on URI design and ownership with best current practices for establishing URI structures, conventions, and formats.

For RESTful IoT applications, typical schemes include "https", "coaps", "http", and "coap". These refer to HTTP and CoAP, with and without Transport Layer Security (TLS) {{RFC5246}}. (CoAP uses Datagram TLS (DTLS) {{RFC6347}}, the variant of TLS for UDP.) These four schemes also provide means for locating the resource; using the HTTP protocol for "http" and "https", and with the CoAP protocol for "coap" and "coaps". If the scheme is different for two URIs (e.g., "coap" vs. "coaps"), it is important to note that even if the rest of the URI is identical, these are two different resources, in two distinct namespaces.

The query parameters can be used to parametrize the resource. For example, a GET request may use query parameters to request the server to send only certain kind data of the resource (i.e., filtering the response). Query parameters in PUT and POST requests do not have such established semantics and are not commonly used.
Whether the order of the query parameters matters in URIs is unspecified and they can be re-ordered e.g., by proxies. Therefore applications should not rely on their order; see Section 3.3 of {{?RFC6943}} for more details.

## Representations

Clients can retrieve the resource state from an origin server or manipulate resource state on the origin server by transferring resource representations.
Resource representations have a media type that tells how the representation should be interpreted by identifying the representation format used.
Typical media types for IoT systems include "text/plain" for simple UTF-8 text, "application/octet-stream" for arbitrary binary data, "application/json" for the JSON format {{RFC7159}}, "application/senml+json" {{I-D.ietf-core-senml}} for Sensor Markup Language (SenML) formatted data, "application/cbor" for CBOR {{RFC7049}}, and "application/exi" for EXI {{W3C.REC-exi-20110310}}.
A full list of registered Internet Media Types is available at the IANA registry {{IANA-media-types}} and numerical media types registered for use with CoAP are listed at CoAP Content-Formats IANA registry {{IANA-CoAP-media}}.

## HTTP/CoAP Methods {#sec-methods}

Section 4.3 of {{RFC7231}} defines the set of methods in HTTP; 
Section 5.8 of {{RFC7252}} defines the set of methods in CoAP.
As part of the Uniform Interface constraint, each method can have certain properties that give guarantees to clients:
Safe methods do not cause any state change on the origin server when applied to a resource.
Idempotent methods can be applied multiple times to the same resource while causing the same visible resource state as a single such request.
The following lists the most relevant methods and gives a short explanation of their semantics.

### GET

The GET method requests a current representation for the target resource.
Only the origin server needs to know how each of its resource identifiers corresponds to an implementation and how each implementation manages to select and send a current representation of the target resource in a response to GET.

A payload within a GET request message has no defined semantics.

The GET method is safe and idempotent.

### POST

The POST method requests that the target resource process the representation enclosed in the request according to the resource's own specific semantics.

If one or more resources has been created on the origin server as a result of successfully processing a POST request, the origin server sends a 201 (Created) response containing a Location header field that provides an identifier for the resource created and a representation that describes the status of the request while referring to the new resource(s).

The POST method is not safe nor idempotent.

### PUT

The PUT method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message payload.
A successful PUT of a given representation would suggest that a subsequent GET on that same target resource will result in an equivalent representation being sent.

The fundamental difference between the POST and PUT methods is highlighted by the different intent for the enclosed representation.
The target resource in a POST request is intended to handle the enclosed representation according to the resource's own semantics, whereas the enclosed representation in a PUT request is defined as replacing the state of the target resource.
Hence, the intent of PUT is idempotent and visible to intermediaries, even though the exact effect is only known by the origin server.

The PUT method is not safe, but is idempotent.

### DELETE

The DELETE method requests that the origin server remove the association between the target resource and its current functionality. 

If the target resource has one or more current representations, they might or might not be destroyed by the origin server, and the associated storage might or might not be reclaimed, depending entirely on the nature of the resource and its implementation by the origin server. 

The DELETE method is not safe, but is idempotent.

## HTTP/CoAP Status/Response Codes

Section 6 of {{RFC7231}} defines a set of Status Codes in HTTP that are used by application to indicate whether a request was understood and satisfied, and how to interpret the answer. 
Similarly, Section 5.9 of {{RFC7252}} defines the set of Response Codes in CoAP.

The status codes consist of three digits (e.g., "404" with HTTP or "4.04" with CoAP) where the first digit expresses the class of the code.
Implementations do not need to understand all status codes, but the class of the code must be understood.
Codes starting with 1 are informational; the request was received and being processed.
Codes starting with 2 indicate a successful request.
Codes starting with 3 indicate redirection; further action is needed to complete the request.
Codes stating with 4 and 5 indicate errors.
The codes starting with 4 mean client error (e.g., bad syntax in the request) whereas codes starting with 5 mean server error; there was no apparent problem with the request, but server was not able to fulfill the request.

Responses may be stored in a cache to satisfy future, equivalent requests.
HTTP and CoAP use two different patterns to decide what responses are cacheable.
In HTTP, the cacheability of a response depends on the request method (e.g., responses returned in reply to a GET request are cacheable).
In CoAP, the cacheability of a response depends on the response code (e.g., responses with code 2.04 are cacheable).
This difference also leads to slightly different semantics for the codes starting with 2; for example, CoAP does not have a 2.00 response code whereas 200 ("OK") is commonly used with HTTP.

# REST Constraints

The REST architectural style defines a set of constraints for the system design.
When all constraints are applied correctly, REST enables architectural properties of key interest {{REST}}:

* Performance
* Scalability
* Reliability
* Simplicity
* Modifiability
* Visibility
* Portability 

The following sub-sections briefly summarize the REST constraints and explain how they enable the listed properties.

## Client-Server

As explained in the Architecture section, RESTful system components have clear roles in every interaction.
Clients have the initiative to issue requests, intermediaries can only forward requests, and servers respond requests, while origin servers are the ultimate recipient of requests that intent to modify resource state.

This improves simplicity and visibility, as it is clear which component started an interaction.
Furthermore, it improves modifiability through a clear separation of concerns.

## Stateless

The Stateless constraint requires messages to be self-contained.
They must contain all the information to process it, independent from previous messages.
This allows to strictly separate the client state from the resource state.

This improves scalability and reliability, since servers or worker threads can be replicated.
It also improves visibility because message traces contain all the information to understand the logged interactions.

Furthermore, the Stateless constraint enables caching.

## Cache

This constraint requires responses to have implicit or explicit cache-control metadata.
This enables clients and intermediary to store responses and re-use them to locally answer future requests.
The cache-control metadata is necessary to decide whether the information in the cached response is still fresh or stale and needs to be discarded.

Cache improves performance, as less data needs to be transferred and response times can be reduced significantly.
Less transfers also improves scalability, as origin servers can be protected from too many requests.
Local caches furthermore improve reliability, since requests can be answered even if the origin server is temporarily not available.

## Uniform Interface

RESTful APIs all use the same interface independent of the application.
It is defined by:

* URIs to identify resources
* representation formats to retrieve and manipulate resource state
* self-descriptive messages with a standard set of methods (e.g., GET, POST, PUT, DELETE with their guaranteed properties)
* hypermedia controls within representations

The concept of hypermedia controls is also known as HATEOAS: hypermedia as the engine of application state.
The origin server embeds controls for the interface into its representations and thereby informs the client about possible requests.
The mostly used control for RESTful systems is Web Linking {{RFC5590}}.
Hypermedia forms are more powerful controls that describe how to construct more complex requests, including representations to modify resource state.

While this is the most complex constraints (in particular the hypermedia controls), it improves many different key properties.
It improves simplicity, as uniform interfaces are easier to understand.
The self-descriptive messages improve visibility.
The limitation to a known set of representation formats fosters portability.
Most of all, however, this constraint is the key to modifiability, as hypermedia-driven, uniform interfaces allow clients and servers to evolve independently, and hence enable a system to evolve.

## Layered System

This constraint enforces that a client cannot see beyond the server with which it is interacting.

A layered system is easier to modify, as topology changes become transparent.
Furthermore, this helps scalability, as intermediaries such as load balancers can be introduced without changing the client side.
The clean separation of concerns helps with simplicity.

## Code-on-Demand

This principle enables origin servers to ship code to clients.

Code-on-Demand improves modifiability, since new features can be deployed during runtime (e.g., support for a new representation format).
It also improves performance, as the server can provide code for local pre-processing before transferring the data.

# Hypermedia-driven Applications

## Motivation
Extensibility at runtime

* The server implements a newer version of the application. Older clients ignore the new links and forms, while newer clients are able to take advantage of the new features by following the new links and submitting the new forms.
* The server offers links and forms depending on the current state. The server can tell the client which operations are currently valid and thus help the client navigate the application state machine. The client does not have to have knowledge which operations are allowed in the current state or make a request just to find out that the operation is not valid.
* The server offers links and forms depending on the client's access control rights. If the client is unauthorized to perform a certain operation, then the server can simply omit the links and forms for that operation.

## A Priori

Knowledge that needs to be shared a priori among all participants of a REST system.

* URI schemes that identify communication protocols,
* Internet Media Types that identify representation formats,
* link relation types or resource types that identify link semantics,
* form relation types that identify form semantics,
* variable names that identify the semantics of variables in templated links,
* form field names that identify the semantics of form fields in forms, and
* optionally, well-known locations.

## At Runtime

Explain how it works during runtime: server knows application and offers possible choices to client, client chooses by following links or submitting forms.

# Design Patterns

## Server Push
Observing State (asynchronous updates) of a resource

## Executing a Function

## Long-running Functions

## Conversion

### Text-to-Speech

### Speech-to-Text

## Collections

## Events as State

# Security Considerations {#sec-sec}

This document does not define new functionality and therefore does not introduce new security concerns.
We assume that system designers apply classic Web security on top of the basic RESTful guidance given in this document.
Thus, security protocols and considerations from related specifications apply to RESTful IoT design.
These include:

* Transport Layer Security (TLS): {{RFC5246}} and {{RFC6347}}
* Internet X.509 Public Key Infrastructure: {{RFC5280}}
* HTTP security: Section 9 of {{RFC7230}}, Section 9 of {{RFC7231}}, etc.
* CoAP security: Section 11 of {{RFC7252}}
* URI security: Section 7 of {{RFC3986}}

IoT-specific security is mainly work in progress at the time of writing.
First specifications include:

* (D)TLS Profiles for the Internet of Things: {{RFC7925}}

# Acknowledgement

The authors would like to thank Mert Ocak, Heidi-Maria Back, Tero Kauppinen, Michael Koster, Robby Simpson, Ravi Subramaniam, Dave Thaler, and Erik Wilde for the reviews and feedback.

--- back

# Future Work

* Interface semantics: shared knowledge among system components (URI schemes, media types, relation types, well-known locations; see core-apps)

* Unreliable (best effort) communication, robust communication in network with high packet loss, 3-way commit

* Discuss directories, such as CoAP Resource Directory

* More information on how to design resources; choosing what is modeled as a resource, etc.
