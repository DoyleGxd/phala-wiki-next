---
title: "3. Phala App to Manage Mining"
weight: 6024
menu:
  docs:
    parent: "khala-mining"
---

> We highly recommend reading about our [staking mechanism]({{< relref "docs/tokenomic//1-mining-staking" >}}) before using the Phala App.

[The Phala App](https://app.phala.network/en/mining/)

Miners and pool owners can use teh Phala App to manage Workers/ Miners and StakePools. It also provides an overview of the status of all the managed Workers/ Miners and StakePools.

## Phala App Manual

### Prerequisites

1. [Create a Khala account]({{< relref "docs/khala-user/polkadot-extension-guide" >}}) as the pool Owner and Worker operator;
2. Get the WorkerPublicKey from Worker and has bound the Worker to the Khala account above.

### Phala App Operations

1. [Connect your Khala account]({{< relref "docs/khala-user/polkadot-extension-guide" >}}).
2. Create StakePool
   - In the left menu, make sure you select 'Mining.'
   - Click "Create Stakepool" in the top right corner;
     ![](/images/docs/khala-mining/create-pool.png)
   - Click "Confirm" in the pop-up window;
   - Sign the transaction in the Polkadot{.js} Extension and wait;
   - The created pool will be listed in Stakepool;
3. Configure StakePool (Optional) 
   - Set Payout
     - Click "Set Payout Pref" of the target pool;
     - Type in the payout in the pop-up window; the default payout is 0, and it can be set between 0-100%;
     - Click "Confirm" to send the transaction;
     - The payout value will be updated in the Stakepool list;
   - Set Staking Capacity
     - Click "Set Cap" of the target pool;
     - Type in the Staking capacity in the pop-up window; the default capacity is unlimited, and it can be set between "Total Stake Now" to unlimited number;
     - Submit the transaction;
     - The capacity value will be updated in the Stakepool list;
4. Add a Miner
   - Select `...` (next to "Info") "Add Worker" (highlited red) in the target pool;
     ![](/images/docs/khala-mining/add-worker.png)
   - Copy & paste your `WorkerPublicKey` of your miner in the pop-up window;
   - Submit the transaction;
   - Your miner will now be listed;
5. Staking
   - After creating the StakePool, you can invite other Stakers to invest or stake yourself;
   - To stake yourself
     - Click "Stake" of the target pool;
     - Click "Contribute" in the pop-up window;
     - Type in the amount to stake, it should be less than your "Transferrable Balance" and "Pool Cap Gap";
     - Submit the transaction;
     - Click the "Stake" of the target pool, and you should see the change of "Locked" amount in "Your Stake Info";
6. Begin Mining
   - Click "Start" of the Worker in "Ready" state;
   - Type in the stake amount for the Worker, it should be more than "Smin" and less than "Smax" and "Pool Free Balance". Noted that you **CAN NOT** change the stake amount during mining;
   - Submit the transaction;
   - The Worker state should transit from "Ready" to "Mining";
7. Get Rewards
   - Click "Claim" of the target pool;
   - You can see your rewards from this pool, including "Owner Reward" and "Staker Reward";
   - Choose your account to claim the rewards;
   - Submit the transaction;
   - Your account balance should be increased accordingly;

### Other Operations

1. Withdraw Staking
   - Click "Stake" of the target pool;
   - Click "Withdraw" in the pop-up window;
   - Type in the amount to withdraw;
   - Submit the transaction;
   - You may wait for at most 14 days to get all your staking (check [staking mechanism]({{< relref "docs/tokenomic/1-mining-staking" >}})). You can check the frozen amount in the "Withdraw Queue" of "Stake" pop-up;
2. Stop Mining
   - Click "Stop" of the Worker in "Mining" or "Unresponsive" state;
   - Submit the transaction;
   - The Worker state should transit to "CoolingDown";
3. Remove Worker
   - Click "Remove" of the Worker in "Ready" state;
   - Submit the transaction;
   - The Worker should be removed from list;

### Explanations of Fields

#### Stakepool List

1. You can click "Create Pool" to create a new StakePool if the list is empty;
2. Fields
   - Owner Reward: The owner reward from the payout which can be claimed immediately;
   - Total Shares: The total staking amount;
   - Free Stake: The free staking amount;
   - Releasing Stake: The total staking amount of the pool Workers in "CoolingDown" state;
3. "Show this only" Button: only show the Workers of the target pool;

#### Worker List

1. You can click "Add Worker" to add a new Worker if the list is empty;
2. Fields
   - Mining Core: The number of CPU cores in use;
   - State: Includes Ready, Mining, Unresponsive and CoolingDown (will turn to Ready after 7 days);
   - Total Reward: All the historical rewards of the Worker;

#### Stake Info

1. Withdraw Queue lists all the funds to be withdrawn, ordered by the issue time. Noted that unmet withdrawal requests can cause **ALL** the Workers to stop mining.
   - Staker: The Staker sending the withdrawal requests.
   - Shares: The reminder funds to be withdrawn.
   - Countdown: If there are still Shares after Countdown reaching 0, all the Workers in the pool will be forced into a 7-day freeze period.
2. Your Stake Info
   - Locked: Your staking amount in this pool.
   - Shares: Your staking amount in use.