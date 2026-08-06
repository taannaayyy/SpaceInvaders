# Space Invaders

A desktop Space Invaders clone written in Java Swing, built as a university Software Engineering course project (CS320, 2024).

![gameplay](docs/gameplay.gif)

<!-- TODO: record a short gameplay clip and save it to docs/gameplay.gif (the docs/ folder does not exist yet). -->

## Features

All of the following are implemented in the source:

- Main menu with username entry, a Start Game button, a Show Scoreboard button, and a volume slider.
- Player ship with left/right movement (bounded to the play area), animated sprites, and spacebar shooting on a 300 ms cooldown.
- Enemies arranged in a grid that march horizontally and drop down when hitting a screen edge, with a random enemy firing back.
- Wave progression: clearing a wave advances the stage number, grows the enemy grid (capped), and raises both the enemy fire chance and enemy bullet damage.
- Scoring (+10 per enemy destroyed) and a live HUD showing username, score, health, stage, and volume.
- Persistent high-score board saved to `resources/scoreboard.txt`, sorted highest-first, viewable from the menu and while paused.
- Pause screen with Resume, Main Menu, Show Scoreboard, and Exit.
- Sound effects (shoot, hit, game over) and looping background music, with volume control.
- Game-over screen shown when player health reaches zero.

## Tech stack

- Language: Java (uses arrow-`switch` and `Stream.toList()`, so Java 16 or newer is required).
- UI/graphics: Java Swing / AWT (no external game engine).
- Audio: `javax.sound.sampled`.

## Build and run

There is no build script. Compile and run from the project root with the JDK directly. The working directory must be the project root, because assets are loaded via paths relative to it.

```sh
# from the repository root
javac -d out $(find src -name '*.java')
java -cp out Main.GameLauncher
```

## Controls

Read from `src/Controller/InputHandler.java`:

- Left arrow: move left
- Right arrow: move right
- Spacebar: shoot

Pause, resume, scoreboard, and exit are mouse-driven buttons, not key bindings. There is no keyboard shortcut to quit or restart.

## Architecture

The code follows an MVC layout by package:

- `Model` — game data and rules. `GameState` owns the player, enemies, and bullets and runs per-frame updates, collisions, and wave logic. `Entity` (interface), `EntityBase` (abstract base), and the concrete `Player`, `Enemy`, and `Bullet` form the entity hierarchy. `ScoreboardModel` reads and writes the high-score file.
- `View` — Swing frames and rendering. `MainMenu`, `GameView`, `PauseScreen`, and `ScoreboardView` are the windows; `GameRenderer` is the `JPanel` that paints the background, player, bullets, and enemies each frame.
- `Controller` — wiring and input. `MainMenuController`, `GameController`, and `ScoreboardController` connect views to the model and handle button events. `InputHandler` is a `KeyAdapter` tracking key state.
- `Utils` — `SoundManager`, a singleton for audio playback.
- `Main` — `GameLauncher`, the entry point.

Design patterns actually present: MVC, Singleton (`SoundManager`), and an abstract-base entity hierarchy. The game loop runs on a manual background `Thread` at roughly 60 FPS in `GameController`.

## Known issues and what I would do differently

- Asset loading depends on the current working directory (relative paths and `System.getProperty("user.dir")`), so the game only runs from the project root. Loading assets from the classpath would fix this and make a runnable JAR possible.
- No build tool. Adding Maven or Gradle would pin the Java version, standardise the build, and enable packaging.
- No automated tests. The game logic in the `Model` package could be unit-tested independently of Swing.
- `SoundManager` mixes classpath (`getResource`) and filesystem (`new File(...)`) loading and has unused/dead members (the 30-slot URL array, `play`/`loop`/`stop` over a clip that is never assigned). It would be cleaner to load every clip one consistent way.
- UI updates are driven by several separate Swing `Timer`s polling the model; binding the view to state changes would be simpler and less redundant.
