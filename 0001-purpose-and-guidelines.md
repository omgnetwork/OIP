---
oip: "0001"
title: OIP Purpose and Guidelines
status: Active
type: Meta
author: Unnawut Leepaisalsuwanna <unnawut@omise.co>
created: 2018-10-05
---

# OIP-0001: OIP Purpose and Guidelines

## What is an OIP?

OIP stands for OmiseGO Improvement Proposal. An OIP is a design document providing information to the OmiseGO community, or describing a new feature for all OmiseGO applications or its process or environment. The OIP should provide a concise technical specification of the feature and a rationale for the feature.

The OIP author is responsible for building consensus within the community and documenting dissenting opinions.

## OIP Rationale

The OIPs are intended to be the primary mechanisms for proposing major new features, for collecting community input on an issue, and for documenting the design decisions that have gone into OmiseGO applications. Because the OIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

For OmiseGO implementers, OIPs are a convenient way to track the progress of their implementation. Ideally each implementation maintainer would list the OIPs that they have implemented. This will give end users a convenient way to know the current status of a given implementation or library.

## OIP Types

There are three types of OIP:

- **A Standard Track OIP** describes any changes that affects one or more OmiseGO implementations, such as a change to the network protocol, a change in business rules, proposed application standards/conventions, or any change or addition that affects the interoperability with the OmiseGO applications. Furthermore, Standard OIPs can be broken down into the following categories.
  - **Core** - improvements requiring a consensus fork, as well as changes that are not necessarily consensus critical but may be relevant to core dev discussions.
  - **Interface** - improvements around the client API/RPC specifications and standards.
  - **eWallet** - improvements around the eWallet application.

- **An Informational OIP** describes an OmiseGO design issue, or provides general guidelines or information to the OmiseGO community, but does not propose a new feature. Informational OIPs do not necessarily represent OmiseGO community consensus or a recommendation, so users and implementers are free to ignore Informational OIPs or follow their advice.

- **A Meta OIP** describes a process surrounding OmiseGO or proposes a change to (or an event in) a process. Process OIPs are like Standards Track OIPs but apply to areas other than the OmiseGO applications itself. They may propose an implementation, but not to OmiseGO's codebase; they often require community consensus; unlike Informational OIPs, they are more than recommendations, and users are typically not free to ignore them. Examples include procedures, guidelines, changes to the decision-making process, and changes to the tools or environment used in OmiseGO development. Any meta-OIP is also considered a Process OIP.

It is highly recommended that a single OIP contain a single key proposal or new idea. The more focused the OIP, the more successful it tends to be. A change to one client doesn't require an OIP; a change that affects multiple clients, or defines a standard for multiple apps to use, does.

An OIP must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement. The proposed implementation, if applicable, must be solid and must not complicate the protocol unduly.

## OIP Work Flow

Parties involved in the process are you, the champion or *OIP author*, and the [*OIP editors*](#oip-editors).

:warning: Before you begin, vet your idea, this will save you time. Ask the OmiseGO community first if an idea is original to avoid wasting time on something that will be be rejected based on prior research (searching the Internet does not always do the trick). It also helps to make sure the idea is applicable to the entire community and not just the author. Just because an idea sounds good to the author does not mean it will work for most people in most areas where OmiseGO is used.

Examples of appropriate public forums to gauge interest around your OIP include [the Issues section of this repository](https://github.com/omisego/OIP/issues/), [one of the OmiseGO Gitter chat rooms](https://gitter.im/omisego/), and [the OmiseGO subreddit](https://www.reddit.com/r/omise_go/). In particular, [the Issues section of this repository](https://github.com/omisego/OIP/issues/) is an excellent place to discuss your proposal with the community and start creating more formalized language around your OIP.

Your role as the champion is to write the OIP using the style and format described below, shepherd the discussions in the appropriate forums, and build community consensus around the idea. Following is the process that a successful OIP will move along:

```
[ WIP ] -> [ DRAFT ] -> [ LAST CALL ] -> [ ACCEPTED ] -> [ FINAL ]
```

Each status change is requested by the OIP author and reviewed by the OIP editors. Use a pull request to update the status. Please include a link to where people should continue discussing your OIP. The OIP editors will process these requests as per the conditions below.

* **Active** -- Some Informational and Process OIPs may also have a status of “Active” if they are never meant to be completed. E.g. OIP 1 (this OIP).
* **Work in progress (WIP)** -- Once the champion has asked the OmiseGO community whether an idea has any chance of support, they will write a draft OIP as a [pull request]. Consider including an implementation if this will aid people in studying the OIP.
  * :arrow_right: Draft -- If agreeable, OIP editor will assign the OIP a number (generally the issue or PR number related to the OIP) and merge your pull request. The OIP editor will not unreasonably deny an OIP.
  * :x: Draft -- Reasons for denying draft status include being too unfocused, too broad, duplication of effort, being technically unsound, or not providing proper motivation or addressing backwards compatibility.
* **Draft** -- Once the first draft has been merged, you may submit follow-up pull requests with further changes to your draft until such point as you believe the OIP to be mature and ready to proceed to the next status. An OIP in draft status must be implemented to be considered for promotion to the next status (ignore this requirement for core OIPs).
  * :arrow_right: Last Call -- If agreeable, the OIP editor will assign Last Call status and set a review end date, normally 14 days later.
  * :x: Last Call -- A request for Last Call status will be denied if material changes are still expected to be made to the draft. We hope that OIPs only enter Last Call once, so as to avoid unnecessary noise on the RSS feed.
* **Last Call** -- This indicates that the OIP has matured and is ready to be accepted or finalized unless material changes or substantial unaddressed technical complaints arise.
  * :arrow_right: Accepted (Standard Track OIPs only) -- A successful Last Call without material changes or unaddressed technical complaints will become Accepted.
  * :arrow_right: Final (Non-Standard Track OIPs) -- A successful Last Call without material changes or unaddressed technical complaints will become Final.
  * :x: -- A Last Call which results in material changes or substantial unaddressed technical complaints will cause the OIP to revert to Draft.
* **Accepted (Core OIPs only)** -- This OIP is in the hands of the OmiseGO client developers.
  * :arrow_right: Final -- Standard Track Core OIPs must be implemented in at least two-thirds of viable OmiseGO clients before it can be considered Final. When the implementation is complete and adopted by the community, the status will be changed to “Final”.
* **Final** -- This OIP represents the current state-of-the-art. A Final OIP should only be updated to correct errata.

Other exceptional statuses include:

* **Deferred** -- This is for core OIPs that have been put off for a future hard fork.
* **Rejected** -- An OIP that is fundamentally broken or a Core OIP that was rejected by the Core Devs and will not be implemented.
* **Active** -- This is similar to Final, but denotes an OIP which which may be updated without changing its OIP number.
* **Superseded** -- An OIP which was previously final but is no longer considered state-of-the-art. Another OIP will be in Final status and reference the Superseded OIP.

## What belongs in a successful OIP?

Each OIP should have the following parts:

- **Preamble** - RFC 822 style headers containing metadata about the OIP, including the OIP number, a short descriptive title (limited to a maximum of 44 characters), and the author details. See [OIP Header Preamble](#oip-header-preamble) for details.
- **Simple Summary** - “If you can’t explain it simply, you don’t understand it well enough.” Provide a simplified and layman-accessible explanation of the OIP.
- **Abstract** - a short (~200 word) description of the technical issue being addressed.
- **Motivation (optional)** - The motivation is critical for OIPs that want to change the OmiseGO protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the OIP solves. OIP submissions without sufficient motivation may be rejected outright.
- **Specification** - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current OmiseGO platforms.
- **Rationale** - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.
- **Backwards Compatibility** - All OIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The OIP must explain how the author proposes to deal with these incompatibilities. OIP submissions without a sufficient backwards compatibility treatise may be rejected outright.
- **Test Cases** - Test cases for an implementation are mandatory for OIPs that are affecting consensus changes. Other OIPs can choose to include links to test cases if applicable.
- **Implementations** - The implementations must be completed before any OIP is given status “Final”, but it need not be completed before the OIP is merged as draft. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of “rough consensus and running code” is still useful when it comes to resolving many discussions of API details.
- **Copyright Waiver** - All OIPs must be in the public domain. See the bottom of this OIP for an example copyright waiver.

## OIP Formats and Templates

OIPs should be written in [Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) format.
Image files should be included in a subdirectory of the `assets` folder for that OIP as follow: `assets/oip-X` (for oip X). When linking to an image in the OIP, use relative links such as `../assets/oip-X/image.png`.

### OIP Header Preamble

Each OIP must begin with an RFC 822 style header preamble, preceded and followed by three hyphens (`---`). The headers must appear in the following order. All headers, except ones marked with `(optional)`, are required.

- `oip:` \<The OIP number (this is determined by the OIP editor)\>.
- `title:` \<The OIP title\>
- `author:` \<A list of the author's or authors' name(s) and/or username(s), or name(s) and email(s). Details are below.\>
- `discussions-to:` (optional) \<a url pointing to the official discussion thread\>
  - `discussions-to` can not be a Github `Pull-Request`.
- `status:` \<Draft | Last Call | Accepted | Final | Active | Deferred | Rejected | Superseded\>
- `review-period-end:` (optional) \<YYYY-MM-DD\>
- `type:` \<Standard Track (Core, Blockchain, eWallet)  | Informational | Meta\>
- `category:` (optional) \<Core | Blockchain | eWallet\>
- `created:` \<Date created on, in ISO 8601 (yyyy-mm-dd) format\>
- `requires:` (optional) \<OIP number(s)\>
- `replaces:` (optional) \<OIP number(s)\>
- `superseded-by:` (optional) \<OIP number(s)\>
- `resolution:` (optional) \<A url pointing to the resolution of this OIP\>

#### OIP Header Preamble Usage

The `author` header optionally lists the names, email addresses or usernames of the authors/owners of the OIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the author header value must be:

```
Random J. User &lt;address@dom.ain&gt;
```

or

```
Random J. User (@username)
```

if the email address or GitHub username is included, and

```
Random J. User
```

if the email address is not given.

Note: The `resolution` header is required for Standard Track OIPs only. It contains a URL that should point to an email message or other web resource where the pronouncement about the OIP is made.

While an OIP is a draft, a `discussions-to` header will indicate the mailing list or URL where the OIP is being discussed. As mentioned above, examples for places to discuss your OIP include [OmiseGO topics on Gitter](https://gitter.im/omisego/topics), an issue in this repo or in a fork of this repo, and [Reddit r/omise_go](https://www.reddit.com/r/omise_go/). No discussions-to header is necessary if the OIP is being discussed privately with the author.

The `type` header specifies the type of OIP: Standards Track, Meta, or Informational. If the track is Standards please include the subcategory (core, blockchain, eWallet).

The `category` header specifies the OIP's category. This is required for Standard Track OIPs only.

The `created` header records the date that the OIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

OIPs may have a `requires` header, indicating the OIP numbers that this OIP depends on.

OIPs may also have a `superseded-by` header indicating that an OIP has been rendered obsolete by a later document; the value is the number of the OIP that replaces the current document. The newer OIP must have a Replaces header containing the number of the OIP that it rendered obsolete.

Headers that permit lists must separate elements with commas.

### Auxiliary Files

OIPs may include auxiliary files such as diagrams. Such files must be named OIP-XXXX-Y.ext, where “XXXX” is the OIP number, “Y” is a serial number (starting at 1), and “ext” is replaced by the actual file extension (e.g. “png”).

## Transferring OIP Ownership

It occasionally becomes necessary to transfer ownership of OIPs to a new champion. In general, we'd like to retain the original author as a co-author of the transferred OIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the OIP process, or has fallen off the face of the 'net (i.e. is unreachable or isn't responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the OIP. We try to build consensus around an OIP, but if that's not possible, you can always submit a competing OIP.

If you are interested in assuming ownership of an OIP, send a message asking to take over, addressed to both the original author and the OIP editor. If the original author doesn't respond to email in a timely manner, the OIP editor will make a unilateral decision (it's not like such decisions can't be reversed :)).

## OIP Editors

The current OIP editors are

- Thibault Denizet (@T-Dnzt)
- Jeremy Lam (@whoisjeremylam)
- Unnawut Leepaisalsuwanna (@unnawut)

## OIP Editor Responsibilities

For each new OIP that comes in, an editor does the following:

- Read the OIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to get to final status.
- The title should accurately describe the content.
- Check the OIP for language (spelling, grammar, sentence structure, etc.), markup (Github flavored Markdown), code style

If the OIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the OIP is ready for the repository, the OIP editor will:

- Assign an OIP number (generally the PR number or, if preferred by the author, the Issue # if there was discussion in the Issues section of this repository about this OIP)

- Merge the corresponding pull request

- Send a message back to the OIP author with the next step.

Many OIPs are written and maintained by developers with write access to the OmiseGO codebase. The OIP editors monitor OIP changes, and correct any structure, grammar, spelling, or markup mistakes we see.

The editors don't pass judgment on OIPs. We merely do the administrative & editorial part.

## History

This document was derived heavily from [Ethereum's EIP-1](https://github.com/ethereum/OIPs/) written by Martin Becze, Hudson Jameson and others, and [Bitcoin's BIP-0001](https://github.com/bitcoin/bips/) written by Amir Taaki, which in turn were derived from [Python's PEP-0001](https://www.python.org/dev/peps/). In many places text was simply copied and modified.

Although the PEP-0001 text was written by Barry Warsaw, Jeremy Hylton, and David Goodger, they are not responsible for its use in the OmiseGO Improvement Proposal, and should not be bothered with technical questions specific to OmiseGO or the OIP. Please direct all comments to the OIP editors.

## License

The OmiseGO Improvement Proposal is released under the [Apache License](LICENSE).
