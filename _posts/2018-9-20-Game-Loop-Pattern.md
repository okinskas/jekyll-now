---
layout: post
title: Game Loop
---

## What is a game loop? ##

The Game Loop pattern is used when creating games, animations or UIs that need to update state irrespective of user input.
Using this pattern, you are able to allow the system to update state while handling the rendering separately,
effectively allowing you to control the user's frame rate in whichever way suits the application.

## Purpose of Example ##

Given a game/animation in which a ball travels and bounces from the boundaries of a box,
define interchangeable loop sequences that will update the game and display.

![_config.yml]({{ site.baseurl }}/images/gameloop/gameloop.png)

## Implementations ##

- Game State = The real state of the game.
- Render State = The state of the display / render.

This means that game state and render state can be different in some examples and can have positive or negative
side effects depending on the implementation.

### Basic ###

```java
public void gameLoop() {
  while (true) {
    updateState();
    updateView();
  }
}
```

The basic loop updates the game state and render state at the same time. This means the game and render states
are always the same. However, the basic loop is not limited and will simply run as fast as the hardware can handle.
This is not useful as it means the game/animation will run at different speed depending on the speed of the hardware.

### Locked ###

```java
public void gameLoop() {

  final int framesPerSecond = 25;
  final long skipTicks = 1000 / framesPerSecond;

  long sleepPeriod;
  long nextUpdateTick = System.currentTimeMillis();

  while (true) {

    updateState();
    updateView();

    nextUpdateTick += skipTicks;
    sleepPeriod = nextUpdateTick - System.currentTimeMillis();

    if (sleepPeriod >= 0) {
      try {
        sleep(sleepPeriod);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
}
```

The locked loop also updates game state and render state at the same time. To combat the issue of state updates 
being dependent on hardware speed, a frame-cap is set. This ensures that the loop must wait for a uniform interval
between updates to ensure a consistent updates of game state and render state.

The issue with this implementation occurs is when slow hardware runs the game/animation. If the hardware cannot
keep up with the number of updates that need to occur per second to ensure a consistent experience, the loop
will continue as soon as it can to try to meet the requirement but will cause stutter in game and frame states as
updates slow down.

Another downside of this implementation is that fast hardware will handle it well, but the hardware's potential
is being limited.



### Capped ###

```java
public void gameLoop() {

  final long ticksPerSecond = 50;
  final long skipTicks = 1000 / ticksPerSecond;
  final int maxFrameSkip = 10;

  long nextUpdateTick = System.currentTimeMillis();
  int loops;

  while (true) {
    loops = 0;
    while (System.currentTimeMillis() > nextUpdateTick && loops < maxFrameSkip) {
      updateState();
      nextUpdateTick += skipTicks;
      loops++;
    }
    updateView();
  }
}
```

The capped loop is the first of these example to allow separate updates of game state and frame state. In this
case, a game state update-rate is set to ensure game state is updated consistently. In that sense,
this implementation attempts to do the same as the locked loop.

In most cases, we can make the assumption that rendering a frame is more intensive than updating game state.
Given this assumption, this loop ensures that if slow hardware is falling behind the update-rate, the loop
will prioritise updating game state and stop rendering while it catches up. A 'max-frame-skip' is often implemented
to ensure that the game/animation does not just freeze on a single frame if the hardware is struggling to meet
the update-rate demand.

This loop allows slower hardware to perform better than the locked loop. However, fast hardware is still bottle-necked
by the update-rate -- the frame-rate will never exceed the game state update-rate.

### Independent ###

```java
public void gameLoop() {

  final long ticksPerSecond = 25;
  final long skipTicks = 1000 / ticksPerSecond;
  final int maxFrameSkip = 5;

  long nextUpdateTick = System.currentTimeMillis();
  int loops;
  float interpolation;

  while (true) {
    loops = 0;
    while (System.currentTimeMillis() > nextUpdateTick && loops < maxFrameSkip) {
      updateState();
      nextUpdateTick += skipTicks;
      loops++;
    }
    interpolation =
        (System.currentTimeMillis() + skipTicks - nextUpdateTick)
            / (float) skipTicks;
    updateView(interpolation);
  }
}
```

The most powerful loop in these examples is the independent loop. As the name states, this loop allows game state
and render state to be updated independently of one another. Given the handicap of the previous loop, you may be
wondering how it is possible to render more frames than there are game states. This loop achieves this via
interpolation. To do this, the game/animation code must also be changed so it is not completely interchangeable 
with the others.

Interpolation produces a value used to understand what the game state would be equivalent to at an exact point
between two states. With this value the render can predict a state between two real state updates and produce a
frame that would not actually exist given any real game state.

The source code and working example for this can be found in the following Git Repository:
[Java Design Patterns / game-loop](https://github.com/okinskas/java-design-patterns/tree/game-loop/game-loop)