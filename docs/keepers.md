# Keepers

## Overview

## Common Conventions

The Keeper class follows similar conventions as the smart contracts. See [smart_contracts.md](./smart_contracts.md) for explanations.

## Classes
### Keeper

The base interface for all other keeper classes. `generate_actions_for_timestep` returns an array of `action` objects that is run by [experiment.py](../experiments/experiment.py)

### VaultKeeper
> superclass: Keeper

Class representation of a keeper that owns vaults. `open_max_vaults` will open the maximum possible number of vaults for each ilk.

### NaiveVaultKeeper
> superclass: VaultKeeper

The Naive Vault Keeper will open the maximum number of vaults possible at each timestep.

### AuctionKeeper
> superclass: VaultKeeper

The base interface for auction keeper classes.
Refer to [Maker's Auction Keeper Bot Setup Guide](https://docs.makerdao.com/keepers/auction-keepers/auction-keeper-bot-setup-guide) for documentation for the output format.
- `find_bids_to_place` will take `now` (current timestep as String TODO) as input and will output a dictionary with `ilk_ids` as keys and lists of bids as values.
- `run_bidding_model` will take `bid`, `ilk_id` as inputs and will output a dictionary with "price" as a key and the bid price as the value.
- `place_bid` will take `bid_id`, `price`,`ilk_id`, `now` as inputs and will output an `action` object based on whether the auction is in a `TEND` or `DENT` phase. This function will also verify that the bid is valid based on the checks that Maker makes (proposed bid is greater than latest bid, etc.)
- `find_bids_to_deal` will take `now` as input and will output a dictionary of valid bids to be dealt. It filters through the bids of a specific `ilk_id`, finding the bids that belong to the current keeper.
- `deal_bid` will take `bid_id`, `ilk_id`, `now` as inputs and will output an `action` that will deal the bid.

### FlipperKeeper
> superclass: FlipperKeeper

The base interface for all other `FlipperKeeper` classes. Inherits from `VaultKeeper`. It will also find, deal, and place bids during both dent and tend phases, and keeps track of a set of Flipper contracts for every Ilk type that it's interested in.
- `find_and_deal_bids` uses `find_bids_to_deal` and `deal_bids` to append multiple action objects to the actions array.

### NaiveFlipperKeeper
> superclass: FlipperKeeper

The Naive Flipper Keeper will start an auction with a bid value that is 5% of the tab/lot. Consecutive bids will increase by the beg + a small, random amount of padding.

### PatientFlipperKeeper
> superclass: FlipperKeeper

Runs the same bidding model as the Naive Flipper Keeper, however will only place bids when the auction has not started yet and there does not exist another keeper with enough DAI to cover the auction's `tab`.

### SpotterKeeper
> superclass: Keeper

Will update the `ilk_id` prices using the `poke` function that is part of the `spotter` smart contract.

### BiteKeeper
> superclass: Keeper

Will bite urns with liquidation ratios that are too low using the `bite` function that is part of the `cat` smart contract.
