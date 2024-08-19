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

The MIMI protocol has a baseline level of metadata privacy, which can be
made more private through the optional use of per-room pseudonyms. This
document describes three of many possible flows that use pseudonyms for
enhanced privacy. It also discusses some ways that spam and abuse prevention
mechanisms can work in conjunction with pseudonyms.


--- middle

# Introduction

The More Instant Messaging Interoperability (MIMI) protocol
{{!I-D.ietf-mimi-protocol}} defines a baseline mechanism of metadata privacy
with the following properties. Each local provider can know which of its
users are a participant in any given room, and the domain of the hub. The
hub provider knows the list of participants for each room that it manages.
Local/follower providers do not learn the identity (or even the domain of)
participants from other providers as those users send handshakes and
application messages, unless the provider happens to also be the hub for the
room.

There is also consensus that user can join rooms using a unique pseudonym
per room. The MIMI endpoints provided by the hub operate equally well on
pseudonyms as on "real" identities. However, there are operational
implications related to authorization, consent, KeyPackage availability,
credentials, spam/abuse mitigation, and disclosure of the user's "real"
identity. As a result there are several possible ways of using pseudonyms
that are compatible with MIMI. This document describes three specific flows.
Other flows and other metadata privacy mechanisms are possible, some of
which also use pseudonyms, for example
{{?I-D.kohbrok-mimi-metadata-minimalization}}.

The flows described here include a connection-oriented flow, an out-of-band
join link flow, and a knock flow. A very high level summary of each flow
follows.

Connection flow:

- Each party obtains (typically single-use) pseudonyms
- Alice finds Bob, and connects with him from one of her pseudonyms
- Alice reveals her actual identity to Bob inside an end-to-end encrypted channel, and provides a second pseudonym
- Bob connects to Alice from one of his pseudonyms to Alice's second pseudonym.

Since the last step is based on a human delay which could vary from seconds
to years, the timing would be difficult to correlate between a pair of
providers with a large volume of traffic. If either provider has a very
small number of users, either provider could use traffic analysis to
associate the second room with Bob.

Out-of-band link flow:

- create a room link
- distribute the link out-of-band
- get the GroupInfo using the link
- join the room
- optionally reveal the "real" identity inside the room

Knock flow:

- a new users Cathy wants to join an established room. she uses a pseudonym
to externally join the associated "knock" room.
- Cathy provides a second pseudonym and KeyPackage inside the "knock room",
and immediately leaves the room
- later, an administrator of the room decides to add Cathy to the room using
the KeyPackage provided by Cathy.

This flow is substantially similar to the connection flow, except that Cathy
immediately leaves the "knock room", and the administrator adds Cathy to an
existing room (vs. Bob creating a new room).


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses MIMI terms defined in {{?I-D.ietf-mimi-arch}} and
{{!I-D.ietf-mimi-protocol}}. MIMI uses the Messaging Layer Security (MLS)
protocol extensively; the document uses several MLS terms defined in
{{!RFC9420}}.


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

Alice eventually destroys the bootstrap room, for example after a random delay (not shown).

Alice and Bob can add each other to additional rooms by sending an
application message with a join link, as shown in the next flow.

The connection flow may be useful when the sender wants to initially
establish connectivity but does not have an out-of-band or third-party
channel to the receiver. On providers with a low volume of flows or when
relatively few flows use pseudonyms, this flow is vulnerable to timing
analysis. Between a pair of providers with large volumes of new rooms using
pseudonyms, this approach can be very effective. This flow has the advantage
that the initiator can validate the real identity of the receiver before
establishing any type of communication. This may be useful when contacting
a known journalist, law-enforcement agent, rights-advocacy group, or ombudsperson.


## Join link flow

The join link flow is useful when two parties meet in person, have another
communications channel (possibly a different MIMI room), or each is
introduced via a trusted third-party over separate (typically secure)
communications channels.

This flow begins with Alice creating a new room from one of her pseudonyms,
adding her own clients to it, and the creating a join link. Alice needs to
have permissions to create a join link for this step. Alice then sends the
join link out-of-band to Bob. This could include showing a QR code to Bob,
or sending it to a trusted third-party to give to Bob.

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

Once Bob receives the join link, he fetches the GroupInfo for the room
and validates it, then joins the room.

As in the previous flow, Bob needs to collect KeyPackages from his other
clients and add them to the room (not shown).

Bob can also then reveal his actual identity to the other participants of
the room in an application message.

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
  |               |               |  Bob real ID  |
  | Message       |       /notify |<~~~~~~~~~~~~~~+
  | Bob real ID   |<--------------+               |
  |<~~~~~~~~~~~~~~+               |               |
  |               |               |               |
~~~

This powerful flow is only possibly when Alice and Bob have an out-of-band
channel, and when Alice has permissions to create a join link in the target
room. Note that if a malicious third-party is used, the parties can still authenticate each other if they disclose their actual identities inside the
target room, but they would lose any metadata privacy properties they had.


## Knock flow

A new user Cathy is aware of a room that she wants to join. Likely there is
web page for the room with instructions for joining. The page includes a
related "knock room" which contains the moderators or administrators of the
target room. The "knock room" can be joined by anyone, but by default each
joiner can send one message to the admins while regular users never receive application messages sent to the group (although the client has the keying material needed to decrypt the ciphertext if they receive it.)

Cathy sends a message with her KeyPackages for one of her pseudonyms. She
then leaves the "knock room".

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
  |               |               | "Knock"       |
  | App Message:  |               | Real ID, KPs  |
  | "Knock"       |       /notify |<~~~~~~~~~~~~~~+
  | Real ID, KPs  |<--------------+               |
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

This flow is ideal for joining an established affinity group. For example,
this method could be used by members of marginalized communities. The
perspective joiner needs to believe that the "knock room" contains only
moderators who will treat their join request in confidence and not "out"
them.


# Disclosing additional identity properties in an application message

The concept of pseudonyms is of limited utility if the subject of the
pseudonym cannot disclose additional "real" identity properties as it
wishes. In the connection flow, Alice reveals her "real" identity only to
Bob; in the join link flow, Bob reveals aspects of his "real" identity to
the room; in the knock flow, Cathy reveals aspects of her "real" identity
to the moderators of her target room, and depending on the policy of the
target room she might reveal a different aspect of her identity inside the
target room.

The properties needed for safe and appropriate identity disclosure include:

- The disclosure MUST be consistent with the room policy, which may require
disclosure of some elements, allow some elements to be optionally disclosed,
and may even forbid disclosure of other elements. For example, a sexually
explicit room might require participants to disclose that they are at least a certain age, and forbid the disclosure of postal addresses and family names.
- The issuer / authority MUST be known to every participant of the room and
trusted for the purpose of making identity assertions for the domain(s) of
the provider of the subject user.
- The signature key of the subject client in its MLS LeafNode MUST be the
same as the public key in the identity assertion.
- Whatever identity construction is used MUST be valid according the rules for that construction.
- The presentation of the identity assertion MUST bind the presentation to
possession of the identity's private key (key binding).
- The key binding SHOULD be scoped to a narrow and relevant audience (for
example the target room), and include other mechanisms to prevent replay/
copy and paste attacks.

This document assumes that these disclosures happen to participants of a
room inside an application message, directly using an appropriate media type
(not necessarily inside a MIMI content message).  The next three subsections
describe three such mechanisms.


## Generic MLS Credential

MIMI describes its use with the MLS protocol {{!RFC9420}}.
If a new media type `application/mls-credential` was defined, clients could
send the MLS Credential struct (possibly with the credential name as a media type parameter.) The MLS struct is reproduced here:

~~~ tls
struct {
    CredentialType credential_type;
    select (Credential.credential_type) {
        case basic:
            opaque identity<V>;

        case x509:
            Certificate certificates<V>;
    };
} Credential;
~~~

For example, the client's MLS LeafNode could contain an MLS Credential with
an X.509 certificate with a `subjectAltName` URI that corresponds to the
pseudonym address of the client. The client could then disclose to the room
a different X.509 certificate with the same issuer and public key which also
reveals the "real" display name, and email address of the user.

OpenID Connect UserInfo Verifiable Credentials (VCs) are another proposed
MLS Credential type {{?I-D.barnes-mls-addl-creds}} with more natural claims
semantics.

## Selective Disclosure JSON Web Tokens

While the use of JSON Web Tokens (JWT) {{?RFC7519}} is widespread, most
uses do not require the holder/presenter to prove possession of a private
key as in {{?RFC9449}}. Selective disclosure JWT (SD-JWT)
{{!I-D.ietf-oauth-selective-disclosure-jwt}} describes both a way to
selectively disclose claims, it also include an optional presenter key
binding mechanism. The format uses the media type `application/sd+jwt`.

This format could be used directly in MIMI with an appropriate profile.

## Selective Disclosure CBOR Web Tokens

Likewise, there are Selective Disclosure CBOR Web Tokens (SD-CWT)
{{!I-D.prorock-spice-cose-sd-cwt}}. SD-CWT uses the media type
`application/sd+cwt`, and requires the use of its key binding
mechanism in presentations.


# Spam and Abuse prevention

MIMI has a requirement to be able to prevent spam and other forms of abuse.
When using pseudonymous identities, there is naturally concern that an
account with many pseudonyms could be used to violate room policy in the
same room repeatedly or in multiple rooms with impunity. This section
explores some implementation options to prevent this.

In a multi-provider messaging system, the hub provider is the only provider that knows the room policy and therefore the only provider that can decide if the policy is being violated. The local provider will need to eventually cooperate with the hub provider in order to prevent the same bad actor from
violating policy repeatedly with different pseudonyms.


## Detection signals

There are several signals used for detection of spam or abuse.
In theory, an explicit abuse report should be a very strong signal of abuse.
However, an abuse report could be sent accidentally; it could be the result
of a misunderstanding about the policy in a room; or it could have been sent
maliciously. A report might be incorrectly processed by an algorithm, or it
might need to wait for a human moderator to process it.

A ban or kick by a moderator or administrator of a group could also be a strong signal, but an administrator might maliciously act against someone
they disagree with rather than someone who actually violated policy. Since
the motivation for the ban or kick is not shared with the hub, the claim is
also impossible to validate.

The hub may also use the rate or pattern of joining groups or sending
messages for one of its own users as an indicator of how spammy that user
is, but for users based on other providers it has no way to correlate that
information across pseudonyms.


## Remedies

Once the hub suspects that a user has violated its policies, it has a number
of possible remedies. Some of these remedies require cooperation with the
local provider of that user.

Actions the hub can take independently:

- prevent the user from sending messages in a room
- remove the user from a room (kick)
- ban the user from a room (temporarily)
- ban the user from a room (permanently)

Actions requiring cooperation of the user's local provider

- suspend the account of the user
- suspend the account of the user and remove the user (including all pseudonyms) from all rooms
- completely delete the account of the user
- suspend the account and any accounts deemed "related" (ex: used the same email address or phone number)
- delete the account and any "related" accounts

Specifying an interface for reporting suspected abusive identities from
one provider to another is currently out-of-scope of the MIMI charter and
likely to be defined between providers or by a more focussed standards
developing organization, such as the Messaging Anti-Abuse Working Group
(M3AAWG).


## Correlating anti-abuse actions across multiple pseudonyms

It is possible that the local provider for a user can lookup the
account associated with a specific user identity. This may be
straightforward, or the provider might implement a scheme where this lookup
is possible but requires extra technical or operational steps and leaves
evidence of the inquiry. This would be the moral equivalent of breaking the
pane of glass in some manual fire alarms.

In other implementations, perhaps no direct lookup is possible during normal
operation of the system, but an API could suspend of delete the account
associated with a particular pseudonym.

For some providers it may be operationally acceptable that only a small
number of pseudonyms can be created for a particular account.

There are also mechanisms such as searchable encryption and homomorphic
encryption, which allow searching for all the pseudonyms with the same
account id with a combined spam factor over a certain threshold.
Alternatively, a provider may use similar encryption mechanisms but require
a positive reputation threshold in order to allow new pseudonyms to be
created.


# Implications with explicit consent mechanism

In many of the flows in this document, one user provides another with a
KeyPackage. In many cases the KeyPackages will be valid for several weeks or
months and most new rendezvous will be realized during that time. However if
a KeyPackage expires, the other party may need explicit consent to fetch a
new KeyPackage to replace the expired one.


# Other mechanism needed

This document describes some flows for effective, selectively
pseudonymous privacy. Use in a MIMI system requires a small amount
of additional specification and implementation. The author has attempted to
list this items according to whether they are in-scope according to the MIMI
charter.

Out of scope:

- A way for clients to obtain pseudonyms from their own providers
- Rate limiting pseudonym creation on a local provider
- Correlating pseudonyms with an account on a local provider (may or may not be possible or necessary)
- A way for a client to obtain a join link from its provider

Assumed in scope:

- A way to indicate a KeyPackage is only valid for initial connections
- A way for MIMI entities to recognize pseudonyms
- A way to examine the room policy about pseudonyms (required, optional,
forbidden), and if certain claims/elements are required, optional, or
forbidden.
- A format or convention to wrap additional credentials, or disclosures of
selective disclosure credentials inside a room, along with KeyPackages, etc.
- Conventions for use of existing authorization and consent primitives
- Conventions to make KeyPackages available appropriately


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
