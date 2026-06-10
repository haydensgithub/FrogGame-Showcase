# Code Samples

These are selected excerpts from the private FrogGame codebase. They are included to demonstrate gameplay programming style, system design, and Unreal Engine implementation work while keeping the complete project source private in case development resumes.

The samples are intentionally focused. They are not full compile-ready project files; project-specific references, asset names, debug code, and unrelated surrounding logic have been removed or simplified for public presentation.

## Samples

| Sample | What It Demonstrates |
|---|---|
| [Boss Melee Selection](enemy-ai/boss-melee-selection/) | Spatial combat decision-making based on the player's relative position |
| [Minion Spawn Selection](enemy-ai/minion-spawn-selection/) | Safe randomized enemy spawning using traces and surface validation |
| [Attack Variety Selection](enemy-ai/attack-variety-selection/) | Enemy attack selection with anti-repetition behavior and delayed attack timing |
| [Wall Interaction Detection](player-movement/wall-interaction-detection/) | Ledge, wall climb, wall scrape, and wall run detection using traces and movement state |
| [Input Buffering](player-controller/input-buffering/) | Buffered player actions used to improve responsiveness during movement/combat lockouts |
