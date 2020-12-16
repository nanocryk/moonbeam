# General structure of a pallet

## Config (trait)

Generic description of the types used this pallet. Since it's associated types, no concrete types
are provided, and the config only define requirement on them. Concrete types will be provided when
implemented on the Runtime.

Get traits are used to describe value instead of types, which can be either static or dynamic.

Pallets can be instancied, allowing the same logic to be used in different settings in the same
Runtime.

## Store (trait)

Description of the blockchain state storage used by the pallet. A set of markers (thanks to the
macro) allow to define getters or declare maps/double maps.

`add_extra_genesis` allow to generate a XXXConfig struct containing the genesis values. It must be
providedwhen creating the runtime.

For each storage slot a struct implementing `StorageValue<T>` is generated. This is where the real
interaction with the storage takes place, with handling of reverts and optimisations.

## Event (enum)

List the events the pallet can emit. `where` clause allow to make shorthands for the Config
associated types.

## Errors (enum)

List the erros a pallet can emit.

## Module (struct) + Call (enum)

Contains the logic of the pallet, and defines the operations that can be called on the blockchain.
Thanks to `decl_module!`, both are declared at the same time. `Module` contains the function code,
while `Call` mirrors the functions signatures in variant of an enum. This allow to encode the
parameters into a call/transaction, which can be used to call the `Module` function when the
call/transaction needs to be executed.

With attributes, execution weight (transaction fees and protection against the halting problem) can
be calculated. A function can also choose to consume all the estimated weight, or return a smaller
more accurate weight based on the execution.

## Origin (enum)

The pallet can define its own origin type, acting like a strongly typed "address". Other pallets
could then only accept calls from this origin, or even only a subset of them based on the variant
and its content.

## Usage in a Runtime

In the node code, the `Config` trait is implemented for the Runtime with concrete types. Most of the
types above must be declared in the `construct_runtime!` macro to incorportate them in the Runtime
types (containing - likely in an enum - the types used by each pallet).

# pallet_collective

The collective pallet allow to create a collective of members who can perform calls in the name of
the pallet. The member set can be changed using the root function `set_members` or throught
implementing `ChangeMembers` (TODO: where ?).

A prime member may be set to determine the default vote behavior with the help of a stategy type in
the `Config`.

A call can be proposed using the `propose` function, which can only be called by a member. If the
vote threshold is 1, the call is directly dispatched. Otherwise it is writen in the storage with the
vote parameters. Any member can then `vote` for or against a proposal. A vote will never trigger a
dispatch directly. When there are enough votes on a proposal, anyone can call `close` to dispatch
the call. It can also be called if the vote period as ended, removing the proposal without
dispatching the call.

A member can choose to ignore the vote and directly execute a proposal.

> It would be great if there was an option to make a proposition without the ability the "shortcut"
> the vote.

Lastly, the root account can close any proposal without dispatching the call.