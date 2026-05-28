# Bottom of the 9th

A [Wherigo](https://www.wherigo.com/) geocaching cartridge that simulates a walk-off baseball scenario across two real-world Seattle baseball diamonds. Built with the [wheriflo](https://test.pypi.org/project/wheriflo/) toolkit.

## The game

Bottom of the ninth inning. The Mariners and Astros are tied. The Mariners need one run. You're up.

Start in the alley between two diamonds and choose your ballpark: the Kingdome on the north field or T-Mobile Park on the south field. Pick your player from seven real Mariners -- Edgar, Griffey, Ichiro, Julio, Cal, Naylor, or Randy -- each with unique stats and hitting styles. Set your batting order, then physically walk to home plate on the selected diamond to begin.

Every at-bat is pitch-by-pitch. You choose your swing approach, how big a lead to take, and whether to steal or stay. Your teammates bat behind you. If you get on base, you physically round the bases while the game plays out. The pitcher throws pickoff attempts. The catcher guns you down if you're not careful. One run wins it.

**Starting location:** The alley between the two diamonds near `47.6596, -122.3613`.

**Final cache:** `N47° 39.574' W122° 21.686'`

## Features

- **7 playable Mariners** with distinct stats (power, contact, bunting, speed, IQ) and hitting styles
- **Two selectable ballparks** -- Kingdome on the north field, T-Mobile Park on the south field -- with field-specific coordinates and venue flavor
- **Pitch-by-pitch gameplay** with swing choice, lead size, and steal/stay decisions
- **Physical baserunning** -- you walk to each base on the selected real diamond
- **Two-stage pickoff system** -- the pitcher throws over based on your lead size, base, and how aggressive you've been. Smart runners (high IQ) dive back safely; reckless ones get tagged out
- **Trivia for items** -- answer Mariners trivia from Lou Piniella or Dan Wilson, plus the first base coach, to earn items (Rally Cap, Pine Tar, Scouting Report, Moose Magic)
- **Replay-aware dialogue** -- play again and the game remembers. The skipper intro changes, the dugout feels familiar, and you get deja vu at first base
- **Walk-off celebration** with Humpy the salmon, venue-specific announcer flavor, a Gatorade shower, and the Golden Trident replay item

## Building

Requires [wheriflo](https://test.pypi.org/project/wheriflo/):

```bash
# Install and build
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo build bottom-of-the-9th/BottomOfThe9Th.lua

# Validate the local GWC
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo validate bottom-of-the-9th/BottomOfThe9Th.gwc

# Compile with Groundspeak (requires wherigo.com credentials)
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo compile bottom-of-the-9th/BottomOfThe9Th.lua

# Test in web player with live reload
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo play --watch bottom-of-the-9th/BottomOfThe9Th.lua

# Open the copy editor on port 8001 (watch mode is on by default)
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo edit --port 8001 bottom-of-the-9th/BottomOfThe9Th.lua
```

Set credentials first:
```bash
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==2.0.5 wheriflo credentials set
```

Before publishing, use the official compile step. Local `build`/`validate` can pass even when Groundspeak rejects a cartridge.

## Project structure

```
bottom-of-the-9th/
  BottomOfThe9Th.lua          # Main cartridge source (~3,500 lines)
  BottomOfThe9Th.gwc          # Built cartridge (local)
  BottomOfThe9Th.gwz          # Built cartridge package
  BottomOfThe9Th_compiled.gwc # Groundspeak-compiled cartridge
  icon.png                    # Cartridge icon
```

## Author

**-Bigfoot-** (detectivebigfoot on wherigo.com)
