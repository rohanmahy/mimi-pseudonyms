---
title: "Some pseudonymous privacy flows for More Instant Messaging Interoperability (MIMI)"
abbrev: "MIMI pseudonym privacy flows"
category: info

docname: draft-mahy-mimi-pseudonyms-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "More Instant Messaging Interoperability"
keyword:
 - metadata privacy
 - pseudonyms
 - pseudonymous
 - selective disclosure
venue:
  group: "More Instant Messaging Interoperability"
  type: "Working Group"
  mail: "mimi@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mimi/"
  github: "rohanmahy/mimi-pseudonyms"
  latest: "https://rohanmahy.github.io/mimi-pseudonyms/draft-mahy-mimi-pseudonyms.html"

author:
 -
  fullname: Rohan Mahy
  organization: Rohan Mahy Consulting Services
  email: rohan.ietf@gmail.com

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

There are several possible ways of using pseudonyms that are compatible with
the MIMI protocol. This document includes three specific flows. Other flows
and other metadata privacy mechanisms are possible, some of which also use
pseudonyms.

The flows described here include the a connection flow, an out-of-band join
link flow, and a knock flow. A very high level summary of each flow follows.

Connection flow:

- Each party obtains (typically single-use) pseudonyms
- Alice finds Bob, and connects with him from one of her pseudonyms
- Alice reveals her actual identity to Bob inside an end-to-end encrypted channel, and provides a second pseudonym
- Bob connects to Alice from one of his pseudonyms to Alice's second pseudonym.

Since the last step is based on a human delay which could vary from seconds to years, the timing would be difficult to correlate between a pair of providers with a large volume of traffic. If either provider has a very small number of users, either provider could use traffic analysis to associate the second room with Bob.

Out-of-band link flow:

- create a room link
- distribute the link out-of-band
- get the groupinfo using the link
- join the room
- optionally reveal the "real" identity inside the room

Knock flow:

- a new users Cathy wants to join an established room. she uses a pseudonym
to externally join the associated "knock" room.
- Cathy provides a second pseudonym and KeyPackage inside the "knock room",
and immediately leaves the room
- later, an administrator of the room decides to add Cathy to the room using
the KeyPackage provided by Cathy.

This flow is substantially like the connection flow, except that Cathy immediately leaves the "knock room", and the administrator adds Cathy to an existing room (vs. Bob creating a new room).

Note: move down.  knock room allows external joiners write only and maybe autoremoves after a few seconds.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# What is needed

Out of scope:

- A way for clients to obtain pseudonyms from their own providers
- Rate limiting pseudonym creation on a local provider
- Correlating pseudonyms with an account on a local provider (may of may not be possible)

In scope:

- A way to indicate a KeyPackage is only valid for initial connections
- A way for MIMI entities to recognize pseudonyms
- A way to examine the room policy about pseudonyms (required, optional, forbidden)
- A way to present additional credentials or disclosures of selective disclosure credentials inside a room


## Spam and Abuse prevention

Detection



Remedy






# Example flows

## Connection flow

Initially Alice and Bob both request a number of pseudonyms (and
associated MLS credentials). They may upload KeyPackages for these
pseudonyms.

~~~ aasvg
ClientA1       ServerA         ServerB         ClientB*
  |               |               |               |
  | Request       |               | Request       |
  | pseudonyms    |               | pseudonyms    |
  +~~~~~~~~~~~~~~>|               |<~~~~~~~~~~~~~~+
  |<~~~~~~~~~~~~~~+               +~~~~~~~~~~~~~~>|
  |               |               |               |
  |     Store KPs |               |     Store KPs |
  +~~~~~~~~~~~~~~>|               |<~~~~~~~~~~~~~~+
  |               |               |               |
  |      ...      | time passes   |      ...      |
  |               |               |               |
~~~

Alice decides to connect to Bob. She creates a room in which to
bootstrap her private connection with Bob. She uses one of her pseudonyms
as her identifier in the new room. Alice searches for Bob using his handle
identifier. She requests KeyPackages for Bob's real identity. Note that she
may need consent to fetch his KeyPackages (not shown), or Bob may grant
blanket consent for connection rooms. Alice adds Bob and Welcomes him to the
new temporary room.

Alice now sends Bob an application message revealing her actual identity,
another of her pseudonyms, and optionally a list of her KeyPackages from
that pseudonym. In all likelihood, the user Bob would not be presented
with any indication that Alice wants to connect until this point.

~~~ aasvg
ClientA1       ServerA         ServerB         ClientB*
  |               |               |               |
  | Create room   |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  | Add Alice's   |               |               |
  | other clients |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  |               | /idQuery      |               |
  |               +-------------->|               |
  |               |        200 OK |               |
  |               |<--------------+               |
  | Request KPs   |               |               |
  +~~~~~~~~~~~~~~>| /keyMaterial  |               |
  |               +-------------->|               |
  |               |        200 OK |               |
  |   connect KPs |<--------------+               |
  |<~~~~~~~~~~~~~~+               |               |
  |               |               |               |
  | Commit, etc.  |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  |      Accepted | /notify       |               |
  |<~~~~~~~~~~~~~~+-------------->|               |
  |               |        200 OK | Welcome, Tree |
  |      Message  |<--------------+~~~~~~~~~~~~~~>|
  +~~~~~~~~~~~~~~>|               |               |
  |               | /notify       |               |
  |               +-------------->|               |
  |               |        200 OK |               |
  |               |<--------------+               |
  |               |               | AliceID, KPs  |
  |               |               +~~~~~~~~~~~~~~>|
  |               |               |               |
  |      ...      | time passes   |      ...      |
  |               |               |               |
~~~

At some point, Bob accepts Alice's connection request. Bob creates
a new room using one of his pseudonyms. Bob adds Alice's clients to the room
using the provided KeyPackages. Bob can also add the rest of his clients at
the same time, assuming that Bob has a way to get KeyPackages securely among
his clients. Bob then sends a message to Alice. The hub sees a room between
two pseudonyms.

~~~ aasvg
ClientA1       ServerA         ServerB         ClientB*
  |               |               |               |
  |               |               |               | Bob
  |               |               |   Create room | accepts
  |               |               |<~~~~~~~~~~~~~~+
  |               |               |               |
  |               |               | Commit, etc.  |
  |               |               |<~~~~~~~~~~~~~~+
  |               |       /notify |               |
  | Welcome, Tree |<--------------+               |
  |<~~~~~~~~~~~~~~+               |  Message      |
  |               |       /notify |<~~~~~~~~~~~~~~+
  | Message       |<--------------+               |
  |<~~~~~~~~~~~~~~+               |               |
  |               |               |               |
~~~

Alice eventually destroys the bootstrap room after a random delay. (Not shown.)

Alice and Bob can add each other to additional rooms by sending an
"invite" application message.



## Join link flow

~~~ aasvg
ClientA1       ServerA         ServerB         ClientB*
  |               |               |               |
  | Create room   |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  | Add Alice's   |               |               |
  | other clients |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  | Create join   |               |               |
  | link          |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  |               | Send join     |               |
  |               | link OOB      |               |
  +==============================================>|
  |               |               |               |
  |      ...      | time passes   |      ...      |
  |               |               |               |
~~~


As in the previous flow, Bob needs to collect KeyPackages from his other
clients and add them to the room.

~~~ aasvg
ClientA1       ServerA         ServerB         ClientB*
  |               |               |               |
  |               |               |               | Bob uses
  |               |               | Use join link | join link
  |               |               |<~~~~~~~~~~~~~~+
  |               |    /groupInfo |               |
  |               |<--------------+               |
  |               | OK  GroupInfo |               |
  |               |-------------->+ GroupInfo     |
  |               |               +~~~~~~~~~~~~~~>|
  |               |               | Commit, etc.  |
  |               |               |<~~~~~~~~~~~~~~+
  |               |       /notify |               |
  | Commit        |<--------------+               |
  |<~~~~~~~~~~~~~~+               |  Message      |
  |               |       /notify |<~~~~~~~~~~~~~~+
  | Message       |<--------------+               |
  |<~~~~~~~~~~~~~~+               |               |
  |               |               |               |
~~~


## Knock flow

A new user Cathy is aware of a room that she wants to join. Likely there is
web page for the room with instructions for joining. The page includes a
related "knock room" which contains the moderators or administrators of the
target room. The "knock room" can be joined by anyone, but by default each
joiner can send one message to the admins while regular users never receive application messages sent to the group (although the client has the keying material needed to decrypt the ciphertext if they receive it.)

Cathy sends a message with her KeyPackages for one of her pseudonyms.

~~~ aasvg
ClientA1       ServerA         ServerC         ClientC*
  |               |               |               |
  |               |               |               | Cathy finds
  |               |               |               | "knock room"
  |               |               | get GroupInfo |
  |               |               |<~~~~~~~~~~~~~~+
  |               |    /groupInfo |               |
  |               |<--------------+               |
  |               | OK  GroupInfo |               |
  |               |-------------->+ GroupInfo     |
  |               |               +~~~~~~~~~~~~~~>|
  |               |               | Commit, etc.  |
  |               |               |<~~~~~~~~~~~~~~+
  |               |       /notify |               |
  | Commit        |<--------------+               |
  |<~~~~~~~~~~~~~~+               | App Message:  |
  |               |               | Knock, KPs    |
  |               |       /notify |<~~~~~~~~~~~~~~+
  | Knock, KPs    |<--------------+               |
  |<~~~~~~~~~~~~~~+               | Remove        |
  |               |               | Proposals     | Cathy removes
  |               |               |<~~~~~~~~~~~~~~+ herself
  | Remove        |<--------------+               |
  |<~~~~~~~~~~~~~~+               |               |
  |               |               |               |
  |      ...      | time passes   |      ...      |
  |               |               |               |
~~~

An administrator of the target room decides to add Cathy using the
KeyPackages she provided in the (related) "knock room".

~~~ aasvg
ClientA1       ServerA         ServerC         ClientC*
  |               |               |               |
  | Commit, etc.  |               |               |
  +~~~~~~~~~~~~~~>|               |               |
  |      Accepted | /notify       |               |
  |<~~~~~~~~~~~~~~+-------------->|               |
  |               |        200 OK | Welcome, Tree |
  |               |<--------------+~~~~~~~~~~~~~~>|
  |               |               |               |
~~~


# Connection request message format

TODO

Invite format

# Implications with explicit consent mechanism

In many of the flows in this document, one user provides another with a
KeyPackage. In many cases the KeyPackages will be valid for several weeks or
months and most new rendezvous will be realized during that time. However if
a KeyPackage expires, the other party may need explicit consent to fetch a
new KeyPackage to replace the expired one.



# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
