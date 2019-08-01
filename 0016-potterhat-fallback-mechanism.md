---
oip: "0016"
title: Potterhat's fallback mechanism when relying on multiple Ethereum clients
status: Final
type: Potterhat
author: Unnawut Leepaisalsuwanna <unnawut@omise.co>
created: 2019-04-24
---

# OIP-0015: Potterhat's fallback mechanism when relying on multiple Ethereum clients

# Simple Summary

Consuming multiple Ethereum clients is pretty hard. Nodes become out of sync, competing chains, etc. At the same time, the ability to consume multiple Ethereum clients is important for resiliency.

This OIP attempts to lay down all common scenarios and suggest ways to deal with them.

# Motivation

Designing the fallback mechanism for Ethereum clients can be quite an effort as you will see below. This OIP hopes to draw a guideline on typical failures and discrepancies that we can expect from consuming multiple clients, regardless of being the same or of different implementations. So any future design/implementation for resiliency could make use of this guideline, not limited to Potterhat as defined in OIP-15 (#15).

# Specification

## Scenario 1 - Sync Delayed
A client stops reporting new blocks, but resume back after a while.

- Continue broadcasting blocks from the active node
- Alert of a `:node_delayed` after a pre-configured `:block_delay_tolerance` threshold
- Alert of a `:node_caughtup` when the delayed node is up to the same block height

![Illustration of Scenario 1: Sync Delay](./0016-potterhat-fallback-mechanism/case1-sync-delay.jpeg)

## Scenario 2 - Sync Delayed & Never Recovered
A client stops reporting new blocks, never recovered.

- Continue broadcasting blocks from the active node
- Alert of a `:node_delayed` after a pre-configured `:block_delay_tolerance` threshold
- Restart the unresponsive node

![Illustration of Scenario 2: Sync Delayed & Never Recovered](./0016-potterhat-fallback-mechanism/case2-delayed-never-recovered.jpeg)

## Scenario 3 - Diverged & Converged
For the same block height, one client reports a block, another client reports another block. But eventually a chain reorg occurs and both clients are in sync again.

- Pick one node (based on pre-configured priority) to broadcast from
- Alert of `:node_diverged`immediately
- Alert of `:chain_reorg` when the nodes converge
- Broadcast a chain reorg event

![Illustration of Scenario 3 - Diverged & Converged](./0016-potterhat-fallback-mechanism/case3-delayed-converged.jpeg)

## Scenario 4 - Diverged & Never Converged
2 or more clients report different blocks for the same block height, never reconciled. Probably a hard fork.

- Pick one node (based on pre-configured priority) to broadcast from
- Alert of `:node_diverged` immediately`
- Stops broadcasting completely after a pre-configured `:max_conflicting_blocks_tolerance`

![Illustration of Scenario 4 - Diverged & Never Converged](./0016-potterhat-fallback-mechanism/case4-diverged-never-converge.jpeg)

## Scenario 5 - Diverged & Delayed & Never Recovered
2 clients reporting differing blocks for the same height. 1 client stopped reporting new blocks, and never recovered.

- Pick one node (based on pre-configured priority) to broadcast from
- Alert of `:node_diverged` immediately
- Alert of a `:node_delayed` after a pre-configured `:block_delay_tolerance` threshold
- Restart the unresponsive node
- Alert `:node_converged` if the node converged after restart

![Illustration of Scenario 5 - Diverged & Delayed & Never Recovered](./0016-potterhat-fallback-mechanism/case5-diverged-delayed-never-recovered.jpeg)

# Implementation

Work in progress at [omisego/potterhat](https://github.com/omisego/potterhat).
