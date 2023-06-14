# Bootstrapping the bridge

Namada's Ethereum bridge can be instantiated in one of two ways:

1. At genesis.
2. Through a governance proposal.

Validator set update proofs can be obtained from genesis onwards, in Namada.
Whether the bridge is started at genesis or post-genesis (through a governance
proposal), validator set update proofs are always available to bootstrap the
bridge and maintain its liveness.

## Ethereum bridge parameters

To bootstrap the Ethereum bridge, there are seven parameters which
must be written to storage:

- `eth_start_height` - The Ethereum block height when the smart contracts
  were deployed. This value also corresponds to the height of the first
  block processed by Namada's Ethereum oracle.
- `eth_bridge_min_confirmations` - The minimum number of block confirmations
  on Ethereum required for any given event to be voted on by Namada validators.
- `eth_bridge_bridge_address` - The address of the `Bridge` contract, used to
  perform transfers in either direction (Namada <> Ethereum).
- `eth_bridge_bridge_version` - The version of the `Bridge` contract, starting
  from 1.
- `eth_bridge_governance_address` - The address of the `Governance` contract,
  used to perform administrative tasks, such as updating validator sets in
  Ethereum.
- `eth_bridge_governance_version` - The version of the `Governance` contract,
  starting from 1.
- `eth_bridge_wnam_address` - The address of the deployment of the native
  ERC20 token, representing NAM in Ethereum.

## Enabling the bridge through a governance proposal

To enable the Ethereum bridge through a governance proposal, the following
conditions should be met:

1. The [smart contracts](./ethereum_smart_contracts.md) of the Ethereum bridge
   should be deployed beforehand, with an initial validator set and other parameters,
   such as nonces, from epoch $E_0$. This step can be done by anyone, but ultimately
   it's up to Namada stakers & validators to decide which contracts to accept.
2. A governance proposal should be held to agree on an epoch $E_{bridge} > E_0$ at
   which to launch the Ethereum bridge. The proposal should be executed at some
   grace epoch $E_{grace}$, with $E_{grace} \le E_{bridge}$. The wasm code of
   the proposal should modify storage with:

    - The epoch $E_{bridge}$ when the bridge is to be enabled.
    - The aforementioned parameters of the Ethereum bridge.

   Additionally, the knowledge of $E_0$ must be communicated through the governance
   proposal's JSON data. This is important, as subsequent validator sets from epochs
   $E$ such that $E > E_0$ should be relayed if the governance proposal passes.
3. Validators should vote on the proposal if the wasm code correctly updates
   Namada's storage, the proposal contains the epoch $E_0$ of the first set of
   validators in the deployed contracts and the contracts are initialized correctly.
4. Eventually, the proposal passes at some epoch $E_{end} \le E_{grace}$, if enough
   validators vote on it. Then, the Ethereum oracle receives an update command, so it
   can start processing Ethereum blocks to extract confirmed events. Should the proposal
   not pass, no state updates are applied in Namada, therefore the bridge is not enabled.

Notice that nothing stops users from preemptively updating the smart contracts
deployed as part of the proposal with validator sets more recent than $E_0$, since
the Namada ledger provides validator set update proofs from genesis, and the address of
the contracts is public knowledge. This should not deter the validity of any
governance proposal, though. If it does not pass, unfortunately this will mean
that some individual(s) have just wasted their tokens on Ethereum. Otherwise,
any observer can detect state changes in the contracts by querying validator set
update Ethereum events since their deployment, which should lead back to the
set of validators at $E_0$.

From $E_{bridge}$ onwards, the bridge is launched and it may start being used.
Validators' ledger nodes will immediately and automatically coordinate in order
to craft the first Bridge pool root's vote extension, used to prove the existence
of a quorum decision on the root of the merkle tree of transfers to Ethereum and
its associated nonce.

### Example

In this example, we assume that all epochs have equal duration and that
the consensus validator set does not change at any point.

1. Putative Ethereum bridge smart contracts are deployed at epoch $30$, with, e.g.
   the `Governance` contract located at `0x00000000000000000000000000000000DeaDBeef`.
2. A governance proposal is made to launch the Ethereum bridge at epoch $36$.
    ```json
    {
        "content": {
            "title": "Launch the Ethereum bridge",
            "authors": "hello@heliax.dev",
            "discussions-to": "hello@heliax.dev",
            "created": "2023-01-01T08:00:00Z",
            "license": "Unlicense",
            "namada_start_epoch": "30"
        },
        "author": "hello@heliax.dev",
        "voting_start_epoch": 30,
        "voting_end_epoch": 33,
        "grace_epoch": 36,
        "proposal_code": "<wasm code>"
    }
    ```
3. Voting on the governance proposal takes place until epoch $33$,
   which includes verifying the validity of the wasm code, the
   deployed contracts, etc.
4. The governance proposal passes at epoch $33$.
5. The bridge is enabled at epoch $36$, which should give enough time for the
   validator sets in the smart contracts to be brought up to date.
6. The Ethereum oracle is started, and validators start voting on confirmed
   Ethereum events, signing Bridge pool merkle roots, acting on transfers
   to Ethereum, etc.

## Enabling the bridge at genesis

The Ethereum bridge can be enabled at genesis by undergoing a procedure
very similar to an Ethereum bridge governance proposal. The main difference
between the two lies in the source of the bridge parameters.

At genesis, Ethereum bridge parameters are written to a genesis file, along
with the genesis validators, and other data. This effectively constitutes the
initial state of the Namada state machine. The addresses of the Ethereum smart
contracts in the genesis file ought to point to valid deployments, and the initial
set of validators in the contracts must reflect the same set of validators present
in the genesis file of Namada.
