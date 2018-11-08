---
oip: "0005"
title: Incident Response
status: Draft
type: Operations
author: Jake Bunce <jake@omse.co>
created: 2018-11-
---

# OIP-0005: Incident Response 

## Summary

Outages are common place when running services. Managing these incidents with a defined process helps to restore services faster and creates trust for the users.

## What defines an incident?

An incident is an event that affects normal operation of an OmiseGO service. This can be a service that is used internally or externally, such as an automation service or a Plasma blockchain. Services are measured to determine normal operation through service level indicators (SLIs), from which service level objectives (SLOs) can be applied.

For example, we may give assurances to third party partners that an endpoint API on a Watcher instance is avaiable for 99.999% of the time. And we may give an assurance that the same API is returns a response within 300ms 99% of the time. If we're not meeting those service level objectives then an incident should be raised.

## Priority Levels

Priority levels are used to define the business impact to a technical service failure. Incidents can start at one level and be raised/lowered as the incident progresses.

### P0

A P0 event is the most severe incident that can happen. An example of this would be a failure of multiple related services, failure of the entire Plamsa chain, or a funds beind deposited into an incorrect wallet account. These events have the capability to impact revenue steams or create significant reputational damage with users and/or partners.

### P1

A P1 is an event that an entire service is unavailable. None of the features of the service are usable. 

### P2

In a P2 event a component of the service is unavailable. This may be an API endpoint, or a particular feature of the service.

### P3

A P3 is an event where a service is degraded and is not performing as expected. SLOs may be breached with API calls taking longer, creating a negative user experience. 
 
## Co-ordination during incidents

When an incident occurs the primary focus is to restore the affected service. For this to happen with complex incidents there needs to be efficient information exchange between the people managing the incident. Here are some guidelines: 

- All communication relevent to the incident should take place in #incidents
- One person from each affected team should take the role of representing the team to aid with troubleshooting
- Keep messages specific and on topic to the incident. "Value X was changed in foo.conf yesterday in PR 230" is prefered over "I think someone was going to change that last week".

## Incident follow up

All incidents created should be reviewed at a post mortum.
