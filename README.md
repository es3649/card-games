# Card Games

They're neat

## Description

This is first and foremost an attempt to classify a broad category of card games in a data-driven structure.
No card game specified has any play rules implemented, but data describes cards, hands, play areas, visibility of those areas, and so forth.
A properly designed game engine could take these game and deck structures to implement any of a wide variety of card games.

# Data Model

Each game requires one game fule, and one or more deck files.
Each of these is a json file, and follows the schemas located at [`/data/schemas`](./data/schemas/).
Each of these is also described below.

## Game

A game consists of `decks`, `zones` (both shared and player), as well as a few additional rule options.

### Decks

The `decks` member describes which and how many decks of cards are used.
It expects an array of objects defining decks.
Each entry is an object taking the following structure:

* `name` (required; string): the name used to refer to this deck in the zones
* `source` (required; string): a path pointing to the location of a json file containing the definition of this deck
* `each_player` (boolean; default false): whether a copy of this deck should be created for each player

### Zones

Zones come in two varieties:

- shared: only one is created globally, and all players interact with it per the specifications
- player: one is created for each player in the game

Each propertiy&mdash;`shared_zones` and `player-zones`&mdash;is sepcified as a list of zones.
Both keys are required, so if a game consists of no zones of that type, simply leave the list empty.

#### Name

The `name` property contains a single string naming the zone.
It is only required if another zone must reference this zone, i.e.: for tricks, or replenishment

#### Card Visibility

The `card_visibility` field specifies how many cards from the zone are visible at a given time.
It expects either a number, specifying the number of visible cards, or one of the following strings:

- `all`: denotes that all cards in the stack zone are visible
- `none`: denotes tht no card in the zone is visible

By default, no cards are visible, unless the zone_visibility property is set independently, then it defaults to "all".

For a game engine implementation, it is recommended that these cards splay out, or show in a stack.

#### Zone Visibility

Determines who is allowed to see the cards in the zone.
If `card_visibility` is set for the zone, then this refers only to those cards, otherwise it applies to all cards in the zone.
It takes one of the following string values:

- `all`: all players
- `owner`: only the owner of the zone (player_zone only)
- `active-player`: only the active player
- `inactive-player`: only inactive players
- `allies`: the active player and their teammates
- `opponents`: opponents of the active player

#### Control

Who can interact with the cards in this pile

Takes the same arguments as [`zone_visibility`](#zone-visibility)

#### Decks

Must contain the name of a deck specified in the top-level `decks` field.
Only cards from the specified decks are allowed in this field.

#### Is Library

The `is_library` field is a boolean indicating whether the zone will contain all reserve cards from the specified deck, as a draw pile.
If multiple zones are designated as libraries, leftover cards from the deck will be distributed randomly and evenly among the llibraries.

#### Size

Any size limitation on the zone.
This can be

* omitted if the size is exactly 1
* an integer, denoting a fixed size
* a range (2-element list) defining legal lower and upper bounds for the zone. The lower bound comes first, and the upper second, as in a methematical interval.
* the literal string `*` if the size is unlimited

#### Order

The `order` property defines the order of cards in the zone.
It can take the following values:

* `fixed-fifo`: the order is fixed and the first cards in are the first cards out (oldest cards are drawn/visible first). This is the most common behaviour for games where you play from the front of your hand and draw from to the back of your hand.
* `fixed-lifo`: the order is fixed and the last cards in are the first cards out (newest cards are drawn/visible first). This the the most common behavior for discard piles
* `shuffled`: the order is randomized whenever one or more cards are added to the zone. This is the most commong behavior for draw piles
* `any`: the cards can appear in any order (and in particular, can be rearranged by the player). This is the most common behavior for games where you have a hand

#### Count

The `count` property specifies how many copies of this zone can be created.
It can take the same properties as [`size`](#size)

#### Replenish

The `replenish` property specifies mechanics for replenishing this zone.
It expects and object with the following properties:

* `when` (required; array): this describes when the zone will replenish. Its elements takes the following values
    * `on_empty`: when the zone becomes empty, or drops below its specifies minimum
    * `on_input`: the zone waits for user input to replenish
    * `turn_start`: replenishes at the start of turn: for player_zones, this means the start of that player's turn
    * `turn_end`: replenishes at the end of turn: for player_zones, this means the end of that player's turn
    * `round_start`: replenishes at the start of a round
    * `round_end`: replenishes at the end of a round
* `from` (required; array): the names of the zones from which cards will be pulled to replenish this zone
* `up_to` (integer > 1): the number of cards this zone will replenish to
* `shuffle` (boolean): should the cards be shuffled when replenishing this zone

When designates, if the zone drops below its legal threshold as described in the [`size`](#size) property, or is less than the `up_to` property here specified, the zone will be replenished by pulling cards from the designated zone(s) until the zones are empty, or until the size of the zone is equal to the value specified in `up_to`, whichever is first.

A common use for this property is to dhuffle a discard pile into the deck when the deck is empty.
This can be acheived with the following:

```json
{
    "when": "on_empty",
    "from": ["discard"],
    "shuffle": true
}
```

It can also be used for drawing up to _n_ cards in hand, as follows (for _n_ = 7):

```json
{
    "when": "turn_end",
    "from": "Draw Pile",
    "up_to": 7
}
```

#### Designations

Sometimes different zones gain different designations.
The `designations` property specifies these designations.
It expects an object with the following structure:

* `values` (requires; array): a list of strings with the designation names
* `default` (string): if one designation is specified by default, then specify it here
* `mutually_exclusive` (boolean; default false): are the designations mutually exclusive, or can several be specified at once

### Other rules:

#### Turns

The `turns` property is a boolean indicating whether the game is taken in turns

#### Players

The number of players this game accommodates.
It takes the same arguments as the [`size`](#size) property for zones.

#### Tricks (Not implemented)

The `tricks` field is an object which specifies .
If not specified, it is assumed to be false.
When the tricks field is true, the following zones are created:

- `trick_play` (required; string or shared_zone): this is or references a public zone, controlled and visible to all players
- `trick_play_from` (required; string or player_zone): the zone from which cards can be played into a trick
- `trick_scoring` (required; string or player_zone): this is or references a player zone, controlled and visible to the owner

TODO: there should be more details here indicating which decks these zones should use, and whether these zones should be face-up or face-down

## Deck

A deck represents some deck of cards.
It's not extensible enough to support most TCGs, unless you want to cram a lot of stuff into the text field, but it can support non-standard suit-rank combos, or un-suited cards, like wild cards in Uno.

A deck can be specified by enumerating cards, or through suit-rank pairings.
For raw cards, a suit is not required, but a pairing must specify a collection of suits and a collection of ranks.
Cards are generated for the pairing by peforming hte cartesion product of the provided suits and ranks.

For example, to generate a standard 52 card deck, the suits&mdash;Spades, Hearts, Diamonds, and Clubs&mdash;can be specified, as well as the ranks&mdash;A, 2, 3, 4, 5, 6, 7, 8, 9, 10, J, Q, K&mdash;and the resulting deck will be all combinations of suits and ranks, creating the standard deck.

### Raw

A list of individual card objects.
Each card can have the following properties

* `value` (required; number or string): the value of the card
* `suit` (string): the suit to which this card belongs
* `color` (3- or 6-digit hex color code): the color of the card. This is cosmetic information for a game engine to use
* `text` (string): any additional text to include on the card
* `copies` (number; default 1): the number of copies of this card to include in the deck

### Pairs

A collection of objects each specifying a list of suits and ranks, and an optional number of copies.
The cartesion product will be performed on the suits and ranks of each object independently, and each element of the result will be added to the deck a number of times equal to the copies member (or once, if not specified)

Each object can have the followin properties:

* `suits` (required; array): an array of suit objects, each object having the following properties
    * `name` (required; string): the name of the suit
    * `color` (3- or 6-digit hex color code): a color for the suit. This is cosmetic information for a game engine to use
* `ranks` (required; array): an array of ranks objects, each having the following properties
    * `value` (required; number or string): the value of the rank
    * `text` (string): any additional text to include on the card
* `copies` (humber; default 1): the number of times to include each element of the cartesion product in the deck

## Improvements

The following are suggestions for augmenting the the data structure

* Ability to specify a total ordering for cards in decks
    * Inherit from a suit-rank dictionary ordering
    * Specify positions for un-suited cards or separate rank-suit blocks
    * Specify a trump suit/card(s)
* Ability to specify certain common card game mechanics such as
    * Trick taking
* Game-level designations or rounds
    * Possibly including hierarchies of rounds, or stages in rounds, for things like draw/pegging/show in cribbage
* Game/zone designation specific visibilities/properties
* Allow multiple sets of (non-)mutually exclusive designations, instead of just one set
* Ability to add one or more scoreboards
* Leftover pile in the event cards cannot be evenly distributes between zones designated as libraries, or in case libraries become full
* Allow a deck source to contain a full ad-hoc deck definition