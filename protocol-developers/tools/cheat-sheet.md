---
description: One-liners to keep you "in the flow".
---

# Cheat Sheet

![](<../../.gitbook/assets/image (1).png>)

#### Quick-start

Setup the SDK in Truffle console and mint DAIx. See the [tutorial](https://docs.superfluid.finance/superfluid/protocol-tutorials/getting-started) for a full walk-through.

## Tokens

**`balanceOf()`**

```
(await daix.balanceOf(bob)).toString() / 1e18
(await dai.balanceOf(bob)).toString() / 1e18

or

(async () => wad4human(await dai.balanceOf(bob))()
(async () => wad4human(await daix.balanceOf(bob))()
```

**`upgrade()`**

```javascript
dai.approve(daix.address, "1" + "0".repeat(42), { from: bob });
daix.upgrade(web3.utils.toWei("50", "ether"), { from: bob });
```

## User Class

**`user()`**

```javascript
const bob = sf.user({ address: bobAddress, token: daix.address });
const alice = sf.user({ address: aliceAddress, token: daix.address });
```

**`bob.details()`**

```javascript
const details = await bob.details();

// returns an object
details: {
    cfa: {
        flows: {
            inFlows: [
                { sender, receiver, flowRate }
            ],
            outFlows: [
                { sender, receiver, flowRate }
            ]
        },
        netFlow: ""
    }
    ida: {
        subscriptions: []
    }
```

## Agreements

### Constant Flow Agreement (CFA)

A **CFA** is a transfer of value from a `sender` to a `receiver` at a constant `flowRate` measured in _amount per second_.

#### User Class (easy mode)

**`bob.flow()`**

```javascript
// create new flow
bob.flow({ recipient: alice, flowRate: "-38580246913580" }); //recipient can be user object or address

// edit existing flow
bob.flow({ recipient: alice.address, flowRate: 0.1 * 1e18 }); //0.1 DAI per second with 18 decimals.

// close flow
bob.flow({ recipient: alice, flowRate: "0" });

// add custom userData parameters
const userData = { message: "here's a flow Alice", flowId: "007" };
bob.flow({ recipient: alice, flowRate: 1 * 1e18, options: { userData } });
```

#### Agreement Helper Class (hard mode)

**`createFlow()`**

```javascript
sf.cfa.createFlow({
  superToken: daix.address,
  sender: bob,
  receiver: alice,
  flowRate: "385802469135802"
});
```

**`updateFlow()`**

```javascript
sf.cfa.updateFlow({
  superToken: daix.address,
  sender: bob,
  receiver: alice,
  flowRate: "632802469135333"
});
```

**`deleteFlow()`**

```
sf.cfa.deleteFlow({superToken: daix.address, sender: bob, receiver: alice, by: bob})
```

**`getNetFlow()`**

```
(await sf.cfa.getNetFlow({superToken: daix.address, account: bob})).toString()
```

**`getFlow()`**

```
(await sf.cfa.getFlow({superToken: daix.address, sender: bob, receiver: alice})).toString()
```

**`listFlows()`**

```
await sf.cfa.listFlows({superToken: daix.address, account: bob})
```

### Instant Distribution Agreement (IDA)

An **IDA** is used to send funds as one-time-payments. It consists of a **Publishing Index** with `indexId`, an `indexValue` which is updated every time `distribute()` is called, and one or more **subscribers**.

Remember there is one _distributor_, and many _subscribers_.

#### User Class (easy mode)

**`bob.createPool()`**

```javascript
await bob.createPool({ poolId: 1 });
```

**`bob.giveShares()`**

```javascript
await bob.giveShares({ poolId: 1, recipient: alice, shares: 90 });
await bob.giveShares({ poolId: 1, recipient: carol, shares: 10 });
```

**`bob.distributeToPool()`**

```javascript
await bob.distributeToPool({ poolId: 1, amount: 1000 });
```

#### Agreement Helper Class (hard mode)

**`createIndex()`** (sent from _publisher)_

```javascript
sf.ida.createIndex({
  superToken: daix.address,
  indexId: 1,
  sender: bob.address
});
```

**`updateSubscription()`** (sent from _publisher)_

```javascript
sf.ida.updateSubscription({
  superToken: daix.address,
  indexId: 1,
  subscriber: carol.address, //who is receiving the units
  sender: bob.address, //the publisher
  units: "100"
});
```

**`approveSubscription()`** (sent from _subscriber)_

```javascript
sf.ida.approveSubscription({
  superToken: daix.address,
  indexId: 1,
  publisher: bob.address, // the publisher
  subscriber: carol.address // who is receiving the units and sending this tx
});
```

**`distribute()`** (sent from _publisher)_

```javascript
sf.ida.distribute({
  superToken: daix.address,
  indexId: 1,
  amount: 100 * 1e18, // amount to distribute
  sender: bob.address // the Publisher
});
```

**`claim()`** (sent from _subscriber)_

```javascript
sf.ida.claim({
  superToken: daix.address,
  publisher: bob.address,
  indexId: 1,
  subscriber: carol.address,
  sender: carol.address // because ANYONE can send this tx
});
```
