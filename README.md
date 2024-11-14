# Card Games

They're neat

## Description

This is first and foremost an attempt to classify a broad category of card games in a data-driven structure.
No card game specified has any play rules implemented, but data describes cards, hands, play areas, visibility of those areas, and so forth.
A properly designed game engine could take these game and deck structures to implement any of a wide variety of card games.

## Data Model

Each game requires one game fule, and one or more deck files.
Each of these is a json file, and follows the schemas located at [`/data/schemas`](./data/schemas/).
Each of these is also described below.

### Game

A game consists of `decks`, `zones` (both shared and player), as well as a few additional rule options, such as `tricks`.

#### Decks

The `decks` member describes which and how many decks of cards are used.
It expects an array of objects defining decks.
For the game Nerts, it may look as follows

```json
"decks": [
    {
        "name": "deck",
        "source": "decks/standard.json",
        "each_player": true
    }
]
```

This indicates that

- the deck described in "decks/standard.json" should be used
- the deck is names and can be referred to later (when describing zones) as "deck"
- each player will have such a deck (and therefore the deck should not be referred to by name in )

#### Other rules:

##### Tricks

The `tricks` field is optional takes a boolean.
If not specified, it is assumed to be false.
When the tricks field is true, the following zones are created:

- `trick_play` this is a public zone, controlled and visible to all players
- `trick_scoring` this is a player zone, controlled and visible to the owner

TODO: there should be more details here indicating which decks these zones should use, and whether these zones should be face-up or face-down

### Zones

Zones come in two varieties:

- shared: only one is created globally, and all players interact with it per the specifications
- player: one is created for each player in the game

#### Visibility

Who can see the cards in this pile

- `all`: all players
- `active-player`: only the active player
- `inactive-player`: only inactive players
- `allies`: the active player and their teammates
- `opponents`: opponents of the active player

#### Control

Who can interact with the cards in this pile

Takes the same arguments as `visibility`

#### Deck

Must contain the name of a deck specified in the top-level `decks` field.

#### Deck pile

This is a boolean indicating whether the zone will contain all reserve cards from the specified deck, as a draw pile.

#### Size

Any size limitation on the zone.
This can be
- ommitted if the size is unlimited
- an integer, denoting a fixed size
- a range (2-element list) defining legal upper and lower bounds for the zone
