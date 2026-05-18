# Bottom of the 9th

A [Wherigo](https://www.wherigo.com/) geocaching cartridge that simulates a walk-off baseball scenario on a real diamond. Built with the [wheriflo](https://test.pypi.org/project/wheriflo/) toolkit.

## The game

Bottom of the ninth inning. The Mariners need one run. You're up.

Pick your player from seven real Mariners -- Edgar, Griffey, Ichiro, Julio, Cal, Naylor, or Randy -- each with unique stats and hitting styles. Set your batting order, then physically walk to home plate on a real baseball diamond to begin.

Every at-bat is pitch-by-pitch. You choose your swing approach, how big a lead to take, and whether to steal or stay. Your teammates bat behind you. If you get on base, you physically round the bases while the game plays out. The pitcher throws pickoff attempts. The catcher guns you down if you're not careful. One run wins it.

**Location:** A baseball diamond in Seattle (near 47.6565, -122.3489)

## Features

- **7 playable Mariners** with distinct stats (power, contact, bunting, speed, IQ) and hitting styles
- **Pitch-by-pitch gameplay** with swing choice, lead size, and steal/stay decisions
- **Physical baserunning** -- you walk to each base on the real diamond
- **Two-stage pickoff system** -- the pitcher throws over based on your lead size, base, and how aggressive you've been. Smart runners (high IQ) dive back safely; reckless ones get tagged out
- **Trivia for items** -- answer Mariners trivia from Dan Wilson and the first base coach to earn items (Rally Cap, Pine Tar, Scouting Report, Moose Magic)
- **Replay-aware dialogue** -- play again and the game remembers. Dan Wilson's intro changes, the dugout feels familiar, and you get deja vu at first base
- **Walk-off celebration** with Humpy the salmon, Rick Rizzs, and a Gatorade shower

## Building

Requires [wheriflo](https://test.pypi.org/project/wheriflo/):

```bash
# Install and build
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==0.4.2 wheriflo build bottom-of-the-9th/BottomOfThe9Th.lua

# Compile with Groundspeak (requires wherigo.com credentials)
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==0.4.2 wheriflo compile bottom-of-the-9th/BottomOfThe9Th.lua

# Test in web player
uvx --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ --from wheriflo==0.4.2 wheriflo play bottom-of-the-9th/BottomOfThe9Th.lua
```

Set credentials first:
```bash
wheriflo credentials set
```

## Project structure

```
bottom-of-the-9th/
  BottomOfThe9Th.lua          # Main cartridge source (~2,900 lines)
  BottomOfThe9Th.gwc          # Built cartridge (local)
  BottomOfThe9Th.gwz          # Built cartridge package
  BottomOfThe9Th_compiled.gwc # Groundspeak-compiled cartridge
  icon.png                    # Cartridge icon
```

## Author

**-Bigfoot-** (detectivebigfoot on wherigo.com)
