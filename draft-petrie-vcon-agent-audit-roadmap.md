---
title: "vCon for AI Agent Conversations: Requirements, Gaps and Extension Roadmap"
abbrev: "vCon Agent Roadmap"
category: info

docname: draft-petrie-vcon-agent-audit-roadmap-latest

submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Virtualized Conversations"
keyword:
 - vcon
 - conversation
 - AI agent
 - audit
 - AUDIT
 - agentproto
 - MCP
 - tool invocation
venue:
  group: "Virtualized Conversations"
  type: "Working Group"
  mail: "vcon@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/vcon/"
  github: "dgpetrie/draft-petrie-vcon-agent-audit-roadmap"
  latest: "https://dgpetrie.github.io/draft-petrie-vcon-agent-audit-roadmap/draft-petrie-vcon-agent-audit-roadmap.html"

author:
 -
    fullname: Daniel G Petrie
    organization: SIPez LLC
    email: dan.ietf@sipez.com

normative:

informative:

  VCON-CORE: I-D.draft-ietf-vcon-vcon-core

  VACR: I-D.draft-birkholz-verifiable-agent-conversations-01

  AUDIT-ARCH: I-D.draft-kuehlewind-audit-architecture-01

  AGENTIC-REQS: I-D.draft-agentic-ai-usecases-requirements

  AAT: I-D.draft-sharif-agent-audit-trail

  DMSC-PS: I-D.draft-song-dmsc-problem-statement

  DMSC-GW: I-D.draft-liu-dmsc-gw-requirements

  SCITT: I-D.draft-ietf-scitt-architecture

  RATS: RFC9334

  MCP:
    target: https://modelcontextprotocol.io/specification/
    title: "Model Context Protocol Specification"

  OTEL-GENAI:
    target: https://github.com/open-telemetry/semantic-conventions-genai
    title: "OpenTelemetry Semantic Conventions for Generative AI"

  ISO-24970:
    target: https://www.iso.org/standard/88723.html
    title: "Artificial intelligence - AI system logging"
    refcontent: ISO/IEC DIS 24970
    author:
      org: International Organization for Standardization


--- abstract

AI agents converse with people, other agents and tools.
There is a growing need to capture these conversations for audit, compliance and review.
This document sorts the requirements for capturing agent conversations into those that fit vCon, those better met by plain logging and those that need other solutions.
It then identifies the gaps in vCon and proposes a strawman set of vCon extensions to fill them.
It is intended as a discussion document.

--- middle

# Introduction

AI agents act on behalf of people and other agents.
They exchange prompts and responses with people, delegate to other agents and invoke tools.
Several drafts define requirements for recording these interactions (e.g. [VACR] and [AUDIT-ARCH]).

vCon [VCON-CORE] is a container for conversation data and the analysis of it.
A vCon captures the observation of a conversation from one or more points in the network.
An agent conversation is not that different from a contact center conversation in this respect.
We do not want to solve all of this with vCon.
vCon is a good fit for some of the requirements, but some are better met with plain logging and some need other solutions.

This document:

* sorts the requirements into vCon, logging and other solutions
* identifies the gaps between the vCon requirements and [VCON-CORE]
* proposes a strawman break down of vCon extensions to fill the gaps

This is a first cut and is surely incomplete.
We would like feedback on what is missing or should not be included.

~~~
 +------+   +-------+   +---------+   +----------+   +------+
 | User |---| Agent |---|  Agent  |---| Subagent |---| Tool |
 |      |   |       |   | Gateway |   |          |   |      |
 +------+   +-------+   +---------+   +----------+   +------+
    ^           ^            ^              ^            ^
   (1)         (2)          (3)            (4)          (5)

            (n) = example observation points
~~~
{: #fig-observation title="Example Observation Points"}

The observation points in [](#fig-observation) are examples, not an exhaustive list.
A conversation may be observed from any point in the network that it passes through, such as a gateway (e.g. [DMSC-PS] and [DMSC-GW]), a proxy or a platform.
In back end scenarios where communications inside a silo are less secure, ordinary network entities may also be observers.
Each observation may go in a single vCon or in separate vCons that are linked or merged later.
This is the same as a contact center call, which may be observed by the carrier, the contact center platform or both.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

See section 2.1 of [VCON-CORE] for terminology used in this document.

Agent: As defined in [AUDIT-ARCH], "a Workload whose behaviour is driven, in whole or in part, by a non-deterministic decision process (typically an LLM)".

Tool: As defined in [AGENTIC-REQS], "an external service invoked by an agent to retrieve data or perform operations. A tool is not necessarily an agent and may not participate in agent-to-agent communication."

Observation Point: A point in the network from which a conversation, or part of it, is observed and recorded (see [](#fig-observation)).

# Requirements and Split

The requirements considered here come from two sources:

* REQ-1 through REQ-11 in section 2.2 of [VACR]
* the four record types in section 5 of [AUDIT-ARCH]

Related work on AI logging includes [ISO-24970].
We only briefly summarize the requirements here, enough to jog the memory of someone who has read the source drafts.

Each requirement is put into one or more of the following categories:

* vCon - the conversation and the analysis of it
* Logging - system and operational events
* Other Solutions - existing or other IETF work

Some requirements are split across categories.

The agent protocols themselves (e.g. [MCP] and those in [AGENTIC-REQS]) are not candidates for capture in vCon.
The data exchanged over them, and the results of them that bear on the conversation (e.g. identity or the tools discovered), are candidates for observation and capture in vCon.

## vCon

From [VACR]:

* REQ-1, REQ-4 Event Logging and Type: conversation level events such as session start and end and permission requests and grants
* REQ-3 Actor Identification: people, agents and tools as parties
* REQ-5 Input/Output Recording: prompts, responses, agent to agent messages and tool invocations and results, with reasoning traces as analysis
* REQ-7 Tamper-Evidence and Integrity: integrity of the record itself, signing and encryption
* REQ-10 Human Oversight: a reviewable and redactable record of the conversation
* REQ-11 Traceability: one vCon with one or more observation points, or separate vCons linked or merged later

The same conversation observed from different points may result in different, duplicate or conflicting data.
vCon records observations, it does not reconcile them.
Conflicting observations are kept, each attributed to its observer.
Any reconciliation can be added as an Analysis.

Token usage is partially captured in vCon today, as vendor responses (e.g. from a LLM or transcription service) are often attached as an Analysis and include token counts.
A more formal definition of token usage would be useful to contact center as well as agent to agent and audit scenarios.

From [AUDIT-ARCH]:

* Interaction Records: prompts, responses, approvals, refusals and agent to agent dialog
* Action Records: the agent view of a tool invocation, the tool or service side record is another observation point which may go in the same vCon, a linked vCon or a log
* Delegation Records: only the structure of which agent delegated to which subagent (i.e. Session and Session_Reference)
* Authorization Transition Records: only the parts requested of a person or agent in the conversation

## Logging

From [VACR]:

* REQ-1, REQ-4 Event Logging and Type: system level events such as configuration and prompt changes and infrastructure events
* REQ-2 Timestamp: clock synchronization
* REQ-8 Incident Response: queries over logs and vCons
* REQ-9 Anomaly and Risk Detection: queries over logs and vCons

From [AUDIT-ARCH]:

* Action Records: the tool or service side record, when not put in a vCon

[AAT] and [OTEL-GENAI] are examples of logging formats for agents.

## Other Solutions

From [VACR]:

* REQ-6 Retention Period: storage and policy
* REQ-7 Tamper-Evidence and Integrity: non-repudiation and attestation (e.g. [SCITT] and [RATS])

From [AUDIT-ARCH]:

* Delegation Records: the delegated authority itself (e.g. OAuth tokens and WIMSE)
* Authorization Transition Records: token level state changes
* propagation of audit context (e.g. trace ID) across domains

## Out of Scope

* TODO: code provenance
* TODO: using vCon to make or enforce decisions during the conversation (e.g. allowing or blocking a tool invocation), as opposed to recording them
* TODO: the transport between agents

# Gap Analysis

The following compares the vCon requirements in [](#vcon) with what [VCON-CORE] already provides.
The gaps are filled by the strawman extensions in [](#strawman-vcon-extensions).

* REQ-1, REQ-4 Event Logging and Type
  * Covered: Session start and duration, join and drop Events
  * Gap: permission and directive requests and responses -> [](#permission-and-directive-events)

* REQ-3 Actor Identification
  * Covered: Party Object, including type "bot", uuid, did and validation
  * Gap: agent attributes such as model, provider, framework, version and workload identity -> [](#agent-party)
  * TODO: is the Party type "bot" sufficient, or are there kinds of agents that need to be distinguished beyond these attributes?
  * Gap: tools, which are named resources rather than Parties -> [](#tool-definitions)

* REQ-5 Input/Output Recording
  * Covered: text Dialog for prompts, responses and agent to agent messages
  * Gap: structured agent to agent content that is not just text, audio or video -> [](#agent-dialog-content)
  * Gap: tool invocations and results -> [](#tool-invocation-events)
  * Gap: the tools available to an agent and their definitions -> [](#tool-definitions)
  * Partial: reasoning traces can be an Analysis today, but there is no Analysis type for them -> [](#agent-analysis-types)
  * Partial: token usage is often in an attached vendor response, but is not formally defined -> [](#token-usage)

* REQ-7 Tamper-Evidence and Integrity
  * Covered: signed and encrypted forms and content_hash
  * Gap: none

* REQ-10 Human Oversight
  * Covered: redacted and amended
  * Gap: none beyond the other gaps listed here

* REQ-11 Traceability
  * Covered: Session and Session_Reference, including the AI Agent example in the sessions Objects Array section of [VCON-CORE]
  * Covered: an observer can be a Party
  * Gap: which observer captured which Dialog, Event or Analysis, and where it was in the network -> [](#observer)
  * Gap: grouping or linking separate vCons, as Session_Reference only refers to Sessions in the same vCon -> [](#group)
  * Gap: correlation with audit context outside of the vCon -> [](#correlation)

* General, also applies to contact centers
  * Covered: Analysis mediatype, vendor, product and schema
  * Gap: no standard media types for the data of common analysis (e.g. transcripts), making it hard to use one analysis as input to another -> [](#standard-analysis-media-types)

The [AUDIT-ARCH] record types map onto the same gaps.
Interaction Records are covered by Dialog and [](#permission-and-directive-events).
The agent view of an Action Record is covered by [](#tool-invocation-events), the tool or service side by [](#observer).
The structure of Delegation Records is covered by Session and Session_Reference.
The conversation parts of Authorization Transition Records are covered by [](#permission-and-directive-events).

# Strawman vCon Extensions

TODO: one subsection per extension: gap and a sketch.

Each of the following is listed as a separate extension.
Some could be bundled into a single draft, for example:

* AI Agent: [](#agent-party) and [](#agent-dialog-content)
* Tools: [](#tool-definitions) and [](#tool-invocation-events)
* Analysis Data: [](#agent-analysis-types), [](#token-usage) and [](#standard-analysis-media-types)
* Multiple vCons: [](#group) and [](#correlation), with [](#observer)

## Agent Party

TODO: model, provider, framework, version and workload identity.
TODO: the model may change mid conversation, so it may be a Session or Dialog parameter, with an Event to mark the change.
TODO: workload identity (e.g. WIMSE), possibly via the existing did parameter or a new one.

## Agent Dialog Content

TODO: a new Dialog type, or at least a new media type for Dialog content, for structured agent to agent exchanges (e.g. a JSON container).
TODO: tweak the text Dialog definition in [VCON-CORE], or justify agent messages as transcribable text.
TODO: the [VACR] media type covers a whole record, not a single one directional message, ask for a single entry media type or define one.

## Tool Definitions

TODO: revisit with more detail.

## Tool Invocation Events

TODO: [MCP] based, the first example extension, see draft-petrie-vcon-tool-invocation.

## Permission and Directive Events

Permission and directive requests occur within tool invocations (e.g. [MCP] elicitation), as well as directly in dialog with an AI agent.

There are two distinct things to record.

The first is a policy decision made by the agent or its host before acting (e.g. before invoking a tool).
The outcome may be that no permission is needed, that permission is required or that the action is disallowed.
No one is asked, so there is no conversation, only the decision and what it applies to.
In some cases it is likely desirable to also capture the reason for the decision (e.g. why an action was disallowed).

The second is a request for permission or input to the person or agent on whose behalf the agent is acting, and their response.
This is part of the conversation.
The response may be more than yes or no (e.g. allow once, allow for the Session, always allow or a choice from a menu).

Only a decision that permission is required is followed by a request.
The two are different and need distinct ways of being recorded in the vCon.
[AAT] and [AUDIT-ARCH] partially capture this distinction.
Before a tool is invoked, the agent or its host decides by policy whether the tool may be used.
The decision is one of no permission needed, permission required or tool use disallowed.
The decision is part of the conversation record, even when no person or agent is asked.
When permission is required, the request and the response are a separate conversation between the agent and the person or agent on whose behalf it is acting, not with the tool.
The decision refers to the tool invocation it applies to (see [](#tool-invocation-events)).
Recording the decision is in scope, making or enforcing it is not (see [](#out-of-scope)).

TODO: [MCP] elicitation (form and URL modes, accept, decline and cancel) as a model.
TODO: [MCP] does not define a message for tool invocation approval, that is left to the host.

## Agent Analysis Types

TODO: reasoning traces.

## Token Usage

TODO: supplemental data on the Analysis Object, separate from the analysis content, and on Dialog generated by a LLM.
TODO: applies to contact centers as well.

## Standard Analysis Media Types

TODO: standard media types for the data of common analysis, starting with transcripts.
TODO: applies to contact centers as well.

## Observer

TODO: applies to contact centers as well.
TODO: may be mostly a definition of how to use the existing schema (e.g. observer as a Party), rather than new parameters, plus observer location or entity.

## Group

TODO: a Group Object that refers to zero or more vCons, grouping vCons in a similar way that Session groups Dialog.
TODO: the "group" parameter is reserved in the vCon Object Parameter Names Registry in [VCON-CORE].
TODO: different from Observer perspectives, but may need to support them.

## Correlation

TODO: AUDIT trace ID, parent ID and delegation chain.

# Open Issues

* TODO: is the Party type "bot" sufficient for agents, or are there kinds of agents to distinguish beyond model, provider, framework, version and workload identity
* TODO: where the tool side record of an invocation goes
* TODO: Observer as an extension or later in core
* TODO: overlap between Agent Analysis Types and Standard Analysis Media Types
* TODO: linking, merging and de-duplicating vCons of the same conversation from different observation points, and handling conflicts

# Security Considerations

* TODO

# Privacy Considerations

* TODO

# IANA Considerations

TBD


--- back

# Acknowledgments
{:numbered="false"}

