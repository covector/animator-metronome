# Animator Metronome

## Controls

Add markers: `m`/`Spacebar`/`Enter`

Clear markers: `c`/`r`/`Escape`

## Settings

Settings can be configured with URL parameters.

Examples:

```
https://covector.github.io/animator-metronome/?fps=24&interval=6
```

```
https://covector.github.io/animator-metronome/?fps=12&interval=1&speed=0.5
```

```
https://covector.github.io/animator-metronome/?fps=4&noblink
```

`fps`: Number of steps per second for the metronome. Number of white squares in the website.

`interval`/`intv`: Brighter blink for every `interval` number of squares.

`speed`: Time scaling for speeding up or slowing down the metronome.

`no-blink`: Disable blinking.

`max-lines`: Maximum lines of markers before it clears itself automatically.

`show-all-delta`: Shows frame difference between frames even for a small delta.

`hide-delta`: Do not display delta information.
