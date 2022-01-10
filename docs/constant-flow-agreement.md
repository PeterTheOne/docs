---
description: Open and close a flow using @superfluid-finance/js-sdk
---

# Constant Flow Agreement

## Introduction

A **Constant Flow Agreement (CFA)** is a transfer of value from a `sender` to a `receiver` at a constant `flowRate` measured in _amount per second_.

Here is overview of the CFA contract. Did someone say _CRUD ?_

```javascript
contract IConstantFlowAgreementV1 is ISuperAgreement {
  function createFlow( ...
  function updateFlow( ...
  function deleteFlow( ...
}
```

Seem straightforward enough? Let's go!

## Prerequisites

Before starting this tutorial you should:&#x20;

* Complete the [@superfluid-finance/js-sdk](../resources/frontend-+-nodejs/) tutorial
* Have some goerli ETH and tokens in your wallet from the dashboard [https://app.superfluid.finance](https://app.superfluid.finance)

## Create a Constant Flow Agreement "CFA"

Bob has some DAIx, and he wants to send 100 per month to alice.

### Create a User object

First let's create a new `User` object for Bob.&#x20;

```javascript
const bobAddress = "0xbbb...." // address of the sender's wallet
const userBob = sf.user({
  address: bobAddress, 
  token: daix.address // address of the Super Token
});
```

### user.flow()

Now let's have Bob start a flow to Alice

```javascript
const aliceAddress = "0xaaa...", // address of the receiver's wallet

await userBob.flow({
  recipient: aliceAddress,
  flowRate: "385802469135802"
});
```

****

So what is this weird number `385802469135802`? This is the amount of DAIx to transfer per second, which is equivalent to 1000 DAIx per month.

But how can I see these flows? We can call `details()` to see a bunch of info about our user:

```javascript
await userBob.details(); // full object
(await userBob.details()).cfa.flows; // detailed flows view
```

If we call `flow()` again, we can edit this stream:

```javascript
await userBob.flow({
  recipient: aliceAddress,
  flowRate: "1000000000000000" // 2592 DAIx per month
});
```

To stop a stream, just pass a `0` value:

```javascript
userBob.flow({
  recipient: alice,
  flowRate: "0"
});
```

Awesome work. You're now ready for the next tutorial [💰 Perform an Instant Distribution](../resources/frontend-+-nodejs/perform-an-instant-distribution.md)

Or you can keep reading to take a deeper dive.

## Go lower level

If you want a bit more freedom to customize, you can go one level deeper into our SDK

#### createFlow()

To achieve this, we will create a **Constant Flow Agreement**. In this agreement, we define the _amount per second_ and `recipient` where DAIx should flow.

```javascript
await sf.cfa.createFlow({
  superToken: daix.address,
  sender: bob,
  receiver: alice,
  flowRate: "385802469135802" // 1000 per month
});
```

Here is the breakdown:

1. A flow is a type of agreement, called a _Constant Flow Agreement_, **CFA** in short
2. We want to send a flow of DAI, so we specify _superToken: daix.address_
3. Using the method `createFlow()`, we pass the arguments for the DAIx token, sender, receiver, and the amount "385802469135802"

So what is this weird number "385802469135802"? This is the amount of DAIx to transfer per second, which is equivalent to 1000 DAIx per month.

```python
>>> (385802469135802 * 3600 * 24 * 30) / 1e18
999.99999999999989 DAIx per month
```

> HUH?! How is bob able to send 1000 DAIx per month if he only has 50 Superfluid enabled DAI? The answer is that the sender isn't required to have the full amount to start a flow. The flow will continue to run as long as he has DAIx.

****

#### Inspect the Flow

The flow is now active, so let's check alice and bob's balances to see what changed. Their balances are updated every second, and reflected on-chain at every new block.

```javascript
(async () => wad4human(await daix.balanceOf(bob)))() >
  48.36226851851852(async () => wad4human(await daix.balanceOf(alice)))() >
  0.2546296296296293;
```

> Note: These amounts will not add up to 50 DAIx, due to a refundable deposit.

To get an idea of all Flow activity for bob, we can check his **net flow**. This will show us the sum of all inflow/outflows for his account.

We can use `getNetFlow()` to see the flow we just created.

```javascript
(await sf.cfa.getNetFlow({superToken: daix.address, account: bob})).toString()
> "-385802469135802" # units of wei
```

Since Bob only has one flow to alice, his net flow is negative. If bob had multiple flows, this would be an easy way to get an overall picture of bob's activity. Let's check it's the right amount:

```javascript
(-385802469135802 * 3600 * 24 * 30) / 1e18 > -999.9999999999989;
```

#### Stop the Flow

Now lets stop the flow by deleting it. Call `deleteFlow()` and select the flow between bob and alice.

```javascript
sf.cfa.deleteFlow({
  superToken: daix.address,
  sender: bob,
  receiver: alice,
  by: bob
});
```

Streams are identified by token, sender, and receiver.

The parameter "by" defines who is closing the stream. Streams can be closed by both sender and receiver.

If we check their balances, we'll see that they now add up to 50 since the refundable deposit has been returned.

```javascript
(await daix.balanceOf(bob)).toString() / 1e18 >
  "49.04360"(await daix.balanceOf(alice)).toString() / 1e18 >
  "0.95640";
```

Great job! You minted some Superfluid-enabled DAI, and created your first Flow.

Next we'll learn about another agreement, called **Instant Distribution**

## Liquidation and Solvency

Here's the general flow of solvency states for Super Tokens in a Constant Flow Agreement.&#x20;

#### **1.** **Solvent**

Everyone is in good standing. The sender's balance shows greater than 0 on the dashboard. The  stream flows to the receiver.&#x20;

#### **2. Critical**

The sender's balance shows 0 on the dashboard, and the stream is now able to be closed by a Licensed Agent. Until the stream is actually closed, funds are paid to the receiver's wallet using the sender's initial deposit. Licensed agents are incentivized to close the stream as soon as possible, since they earn the remaining balance of the sender's deposit.&#x20;

Technically, anyone can close stream during the Critical or Insolvent states, however only Licensed Agents can receive deposit rewards.

#### **3. Insolvent**

If the Licensed Agent has not closed the stream, and the deposit has been completely consumed,  then the Insolvent state begins. The stream will continue to the receiver, however since these tokens don't actually exist in the sender's wallet, we must keep track of this debt so that the Super Token itself can remain solvent within the Superfluid Protocol. We call this debt the "**Delay**".&#x20;

Once the Licensed Agent, or anyone closes the stream, the Delay is taken from their Bond as a slashing fee. This slashing fee is then burned, to offset the tokens created during insolvency.
