---
layout: post
title: Java Flip Tile
---

A simple tile-based game built in JavaFX. The aim of the game is to flip white tiles on a board until all are
black. The complexity comes with the fact that flipping a single tile will also flip all adjacent ones.
The Java Flip Tile project can be found [here](https://github.com/okinskas/FlipTile).

First, Download/clone the repository.

```bash
git clone git@github.com:okinskas/FlipTile.git
```

From the home directory, build the jar via Maven.

```bash
$ mvn clean install
```

Start the game using the newly created jar.

```bash
$ java --jar target/fliptile-1.0-SNAPSHOT.jar 
```

You will see the game with a blank board of tiles.

![_config.yml]({{ site.baseurl }}/images/fliptile/blank-board.png)

Click on tiles to flip the colour of each tile along with the tiles touching each side.

![_config.yml]({{ site.baseurl }}/images/fliptile/in-game-board.png)

Once all tiles have been flipped to black -- you win! Click OK to start again.

![_config.yml]({{ site.baseurl }}/images/fliptile/complete-message.png)

_Source code for the project can be found [here](https://github.com/okinskas/FlipTile)_.