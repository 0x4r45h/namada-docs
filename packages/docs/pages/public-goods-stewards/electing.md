# Becoming a steward

A public goods steward can consist of an arbitrary number of people, and can be a single person. The only requirement is that the steward's multisignature account is elected by the community, through a governance proposal.

For this reason, the first step to becoming a steward is to create a multisignature account. This can be done using the commands found in (TODO: link to multisig docs). Once the multisig account is created, the steward can submit a governance proposal to elect the account as a steward.

## Governance proposal

The governance proposal required to elect a new steward is `StewardProposal`.

### Crafting the `proposal.json` file for a `StewardProposal`

The `steward_proposal.json` file contains the information about the proposal. It is a JSON file with the following structure:

```json
{
    "proposal" :{
        "content": {
            "title": "Stewie for Steward 2024",
            "authors": "stewie@heliax.dev",
            "discussions-to": "forum.namada.net/t/stewies-manifesto/1",
            "created": "2024-01-01T00:00:01Z",
            "license": "MIT",
            "abstract": "Stewie is running for steward, with a focus on technical research. The technical research I will be focused on will definitely not be for weapons of mass destruction. There is some possibility however that I may be focusing somewhat on open source software for weapons of mass destruction.",
            "motivation": "Nobody knows technical reserach better than me. Trust me. I know it. I have the best technical research. I will be the best steward. Last night, Namada called me and said, Stewie, thank you. I will make public goods funding great again",
            "details": "As a genius baby, I possess an unmatched level of intelligence and a visionary mindset. I will utilize these qualities to solve the most complex problems, and direct public goods funding towards weapons of mass destruction ... i mean open source software for weapons of mass destruction",
        },
        "author": "stewie",
        "voting_start_epoch": 3,
        "voting_end_epoch": 6,
        "grace_epoch": 12,
    },
    "data" : [
        {
            "action" : "add",
            "address" : "atestatest1v4ehgw36g4pyg3j9x3qnjd3cxgmyz3fk8qcrys3hxdp5xwfnx3zyxsj9xgunxsfjg5u5xvzyzrrqtn"
        }
    ]     
}
```

The `"data"` field contains the structure that allows either the addition or removal of a multisignature account from the list of stewards. In this case, the `"action"` is `"add"`, and the `"address"` is the address of the multisignature account that will be elected as a steward. If the `"action"` was `"remove"`, the `"address"` would be the address of the multisignature account that would be removed from the list of stewards.

```admonish note
In the motivation and abstract field, it is important to make clear what type of public goods funding the steward will be focusing on. The *areas of public goods funding* can be found in the [public goods funding specs](https://specs.namada.net/economics/public-goods-funding.html#funding-focuses).
```

### Submitting the proposal to the ledger

The CLI command to submit a proposal is:

```bash
namadac init-proposal \
        --pgf-stewards \
        --data-path <path/to/steward_proposal.json> \
```

```admonish note
Where `<path/to/steward_proposal.json>` is the path to the `steward_proposal.json` file.
```

### Becoming elected

Once the proposal is submitted, it will be voted on by the community (see [voting](./voting.md)). If the proposal passes, the account will become a steward. If the proposal fails, the account will not become a steward.

Once a multisignature account becomes elected (which will happen at the end of the `grace_epoch`), it will be able to submit proposals to the public goods funding pool (see [submitting proposals](./proposing.md#proposing-funding).