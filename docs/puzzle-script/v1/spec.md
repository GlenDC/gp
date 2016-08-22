What follows is the specification of the original (first) version of a typical puzzle script file, as found on the website http://www.puzzlescript.net, created by [Stephen Lavelle](http://www.increpare.com/), creator of the original Puzzle Script and the entire web-based environment that's linked to it. This document describes my understanding of this version, out of which the idea for this project has grown.

# File Format (.ps)

A game made with puzzle script has one big script file, which is responsible for describing the entire game. What follows is a description of each section of a puzzle script file, in order.

## Prelude

The prelude has no header, as it is always at the top of the file. The prelude describes all the different game options, each of which has its own line. Most of the options are optional and have a default value.

Editor options can be described as follows:

+ Set title of the game: `title <TITLE>` (required)
+ Set name of the author: `author <NAME>` (required)
+ Set website of the author: `homepage <URL>`
+ If player doesn't move element, cancel it: `require_player_movement`
+ Set key repeat interval: `key_repeat_interval <REAL(def:0.12)>` (optional)
+ Set Color Palette: `color_palette <INDEX(def:1)>`
    + Index possibilities:
        + `1`: mastersystem (variation of [Arne](http://androidarts.com/palette/16pal.htm])'s 16-Colour palette)
        + `2`: gameboycolour
        + `3`: amiga
        + `4`: arnecolors
        + `5` : famicom
        + `6` : atari
        + `7` : pastel
        + `8` : ega
        + `9` : amstrad
        + `10` : proteus_mellow
        + `11` : proteus_rich
        + `12` : proteus_night
        + `13` : c64
        + `14` : whitingjp
+ Event trigger interval: `again_interval <REAL(def:0.1)`
+ Global background color: `background_color <NAME(def:black)/#HEX>` (optional)
+ Show compile instructions: `debug`
+ Hide action key (X) instruction from title screen: `noaction`
+ Let action button only respond to individual presses: `norepeat_action`
+ Disable undo key: `noundo`
+ No Restart: `norestart`
+ Run game rules at startup: `run_rules_on_level_start`
+ Only draw every other line: `scanline`
+ Set font color: `text_color <NAME(def:white)/#HEX>`
+ Log all rule information at runtime: `verbose_logging`
+ Play youtube video in background: `youtube <YOUTUBE_ID>`
+ Stop player from moving crazy fast (in realtime games): `throttle_movement`
+ Center camera to WxH section of the player: `zoomscreen <W>x<H>`
+ Make the game work in real time: `realtime_interval <REAL>`

## Objects

header:
```
========
OBJECTS
========
```

An object consists of an _identifier_ and a _color_, each of which are one their own line. Each object is separated by an empty newline.

The simplest way to describe an object is as follows:

```
<IDENTIFIER>
<COLOR>
```

Following color values are supported: black, white, grey, darkgrey, lightgrey, gray, darkgray, lightgray, red, darkred, lightred, brown, darkbrown, lightbrown, orange, yellow, green, darkgreen, lightgreen, blue, lightblue, darkblue, purple, pink, transparent

Hex values (#000000 format) can also be used instead of names for color values.

An object can however also be described as follows:

```
<IDENTIFIER>
<COLOR#1> <COLOR#2> <COLOR#2>
.222.
.000.
22122
.222.
.2.2.
```

This allows you to have a custom icon, instead of the default rectangle. Every number represents the color, indexed by the order given. A dot (.) represents a transparent tile.

## Legend

header:
```
========
LEGEND
========
```

The legend sections links an _identifier_, as specified in the _Objects_ section above, with a unique _character_. This _character_ will can later be used to describe a level tile.

Each icon in the legend is described as follows:

```
<ICON> = <IDENTIFIER>
```

You can also have a icon stand for multiple objects at once (each object of this group has to be on a unique layer):

```
<ICON> = <IDENTIFIER#1> and <IDENTIFIER#2>
```

The second purpose of this to describe properties of objects, that can be referred in rules and other sections. Example:
```
<PROPERTY#1> = <IDENTIFIER#1> or <IDENTIFIER#2>
<PROPERTY#2> = <IDENTIFIER#3> or <IDENTIFIER#4> or <IDENTIFIER#5>
```

## Sounds

header:
```
========
SOUNDS
========
```

Links sounds to actions, played when that action gets executed. Example:

```
player move  up 142315
Player Move  down 142313
Player Move  right 142311
Crate Move 412312
Player CantMove up 41234
Crate CantMove 41234
Crate Create 41234123
CloseMessage 1241234
Sfx0 213424
Sfx3 213424
```

Full list of possible sound delecrations at [PuzzleScript/Sounds](http://www.puzzlescript.net/Documentation/sounds.html).

## Collision Layers

header:
```
================
COLLISIONLAYERS
================
```

Within this section you describe the different collision layers. A layer consists of a single or multiple object-identifiers. Objects can only interact with one another when they are on the same layer.An object can only be on one layer. A layer can have has many objects as there are objects. Layers are defined from bottom (first) to top (last).

A layer with one object is described as follows:

```
<IDENTIFIER>
```

A layer with multiple objects is described as follows:

```
<IDENTIFIER#1>, <IDENTIFIER#2>
```

The background layer (`Background`) is a special layer. Every game must have one and it must be on its own layer, defined first (the bottom most layer).

You can however have several types of background layer by specifying aliases as follows:

```
Background = Background1 or Background2
```

Some identifiers are reserved keywords, such as:

+ `Background`: The background (Optional ID, e.g. `Background1`) (Required)
+ `Player`: A player (Optional ID, e.g. `Player1`) (Required)

## Rules

header:
```
======
RULES     
======  
```

The rules section allows you to specify rules, that form the core of your game mechanics. Each rule has to be on its own line. Rules are all about replacement patterns (what replaces what when the user presses a button?).

Each rule has a left-hand side and a right-hand side. The left-hand side is the pattern that is looked for, and the right-hand side describes what will happen when that pattern is found.

The order of rules matters, whatever is defined first, is executed first (iff executed).

A turn is like this:

1. The player is marked as being someone who wants to move
2. Each of your rules is applied as often as it can be, before moving to the next.
3. Movement happens.
4. An optional extra stage for rules you want to apply at the very end.

The simples rules (does nothing) looks as follows:

```
[] -> []
```

Within the pair of brackets you can define the patterns. To see if 2 objects overlap, you can put their identifiers simply next to each-other:

```
[ Player Spikes ] -> [ DeadPlayer Spikes ]
```

To indicate the direction of an object, you can use the icons: > (right), < (left), ^ (up), V (down). The following rule example allows a player to move a crate when coming from the left:

```
[ > Player | Crate ] -> [ > Player | > Crate ]
```

An object can have all kind of options, not just directions. Other possible options are:

+ `STATIONARY`: The object is currently not moving
+ `MOVING`: The object is currently moving
+ `UP`, `RIGHT`, `DOWN`, `LEFT`, `Horizontal`, `VERTICAL`: Specific Absolute directions (e.g. gravity)
+ `ORTHOGONAL`: Any absolute direction
+ `NO`: Look for anything that is not this object
+ `RANDOM`: have this object be there on random moments (used on the Right Hand Side only)
+ `RANDOMDIR`: move an object in a random direction
+ `ACTION Player`: Respond to when a player pressed the action button

In the previous example you also noticed the `|` in between the `Player` and the `Crate`. This indicates that the objects are next to one another, rather than overlapping (as in the first example). By default we don't are how the different objects object specified touch. This can be changed by specifying the direction before the replacement pattern. The same values can be used as for absolute directions. If you want to check for example if a `Player` is falling on top of a `Monster`, and make it disappear if so, you can do that as follows:

```
UP [ DOWN Player | Monster ] -> [ | DOWN Player ]
```

A rule can also be specified at random occasions as follows:

```
RANDOM [ Wall ] -> [ ]
```

To specify a variable amount of spaces in between 2 objects you can put an ellipse `...` in between the 2 objects as follows:

```
[ LavaGun | ... | Mortal ] -> [ LavaGun | ... | Corpse ]
```

Patterns can also trigger commands rather than replace something. In that case your left-hand side would still be thing you're looking for, but the right-hand side becomes the command you want to trigger. Following commands are available:

+ `[ <IDENTIFIER> ] again`: allows you to trigger another turn after this one, without player input
+ `CANCEL`: cancel the whole move
+ `SF<##>`: trigger a sound effect
+ `CANCEL SF<##>`: cancel the sound effect specified by `##` [0-10]
+ `CHECKPOINT`: save the game state, now whenever the `R` key is pressed or the `RESTART` command is triggered, the player will start from this game state, rather than the initial level state
+ `RESTART`: same as pressing `R`, restart to the initial level state or the last-saved game-state
+ `WIN`: wins the level
+ `MESSAGE`: gives the player a message

A rule can also describe both a normal replacement pattern and a command. Example:

```
[ > Player | Person ] -> [ < Player | Person ] MESSAGE Woah, buddy, back off!
 ```

 Rules when compiled are put into groups. The interpreter will run as many rules of the same group as possible, before moving on to the next group. An example of a specified rule and its compiled version (group):

 ```
 [ > Player | Crate ] -> [ > Player | > Crate ]
 ```

 ```
 Rule Assembly : (4 rules )
===========
(52) UP [ up player | crate ] -> [ up player | up crate ]
(52) DOWN [ down player | crate ] -> [ down player | down crate ]
(52) LEFT [ left player | crate ] -> [ left player | left crate ]
(52) RIGHT [ right player | crate ] -> [ right player | right crate ]
===========
```

You can also grouping rules together manually, by prepending all rules except the first one with a plus sign `+`, as follows:

```
[ > Player | Crate ] -> [ > Player | > Crate ]
+[< Player | Crate ] -> [ < Player | < Crate ]
```

## Win Conditions

header:
```
==============
WINCONDITIONS
==============
```

Here are possible formats of win conditions:

```
(neko puzzle - you win if there's no fruit left)
No Fruit

(sokoban - you win if every target point has a crate on it)
All Target On Crate

(you win if Love exists somewhere)
Some Love

(you win if there's some gold in the chest)
Some gold on chest

(you win if all gold has been taken from the chest)
No gold on chest
```

If multiple win conditions have been defined, all of them have to be satisfied in order to win a level.

## Levels

header:
```
=======     
LEVELS
=======
```

A level is always rectangular, on which each tile (grid cell) is described in the form of a single character. This character is the icon as defined in the `OBJECTS` section. Optionally you can also specify a message that's shown when the level was solved.

Example of a `LEVELS` section, describing 3 levels, 2 of which have a end-of-level message specified:

```
#########
#.*.*.O.#
#..*..O.#
#.P.*.*.#
#########

MESSAGE Woah, that was tough, eh?!

#########
#.....@.#
#.P.*.O.#
#.......#
#########

#########
#.....@.#
#.P.*.O.#     
#.......#
#########

MESSAGE Thank you for playing the game.
```
