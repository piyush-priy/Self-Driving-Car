# Self-Driving Car in a Virtual World

A browser-based self-driving car simulation that learns to navigate complex, custom-built worlds. The car's "brain" is a tiny feed‑forward neural network trained via a simple genetic algorithm, so it can evolve better driving behavior over generations. A built-in world editor lets you create and save your own road networks and markings.

No server or build step required—just open the HTML files locally in a modern browser.

## What’s included

- The Car Simulation (root `index.html`): runs hundreds of cars in parallel, tracks the best performer, and visualizes its neural network and a mini map.
- The World Editor (`world/index.html`): a standalone graphical editor to draw road graphs, add markings (stop, yield, lights, crossing, parking, start, target), and save them as `.world` files.

## Features

- Visual world editor to design road graphs and city scenery
- Markings system: Start, Stop, Yield, Crossing, Parking, Traffic Lights, Target
- Procedural buildings/trees around roads for depth and context
- Neural Network Brain: each car is controlled by its own feed‑forward neural network
- Genetic Algorithm: selection + mutation over generations (no RL required)
- Dynamic simulation: run hundreds of cars (N configurable in `main.js`, default 1000)
- Live visualization:
  - Main view: camera follows the current best car
  - Neural network view: real‑time activation/weight visualization of the best brain
  - Mini map: top‑down map of the road graph
- Persistent training: save the best brain to localStorage and reload to spawn a new generation
- Save/load worlds (`.world`) and car brains (`.car` or browser localStorage)
- Pathfinding: Dijkstra-based shortest path for corridor generation and editor preview

## Folder layout

```
/Self Driving Car/
├── index.html          # Main simulation page, loads all scripts
├── main.js             # Core simulation loop, runs the genetic algorithm
├── car.js              # Defines the Car class (physics, sensors, brain, fitness)
├── network.js          # Defines the NeuralNetwork class (feedForward, mutate)
├── sensor.js           # Defines the Sensor class (ray-casting)
├── visualizer.js       # Code for drawing the neural network visualization
├── miniMap.js          # Code for drawing the top-down mini-map
│
├── saves/
│   └── right_hand_rule.car # A pre-trained brain file (can be loaded in main.js)
│
└── world/
    ├── index.html      # The World Editor application
    ├── world.js        # Defines the World class (procedural generation)
    │
    ├── /js/
    │   ├── /editors/   # Code for all tools in the editor (GraphEditor, StopEditor)
    │   ├── /math/      # Contains Graph.js (the data structure for the world)
    │   ├── /markings/  # Code for all road markings (Stop, Start, Target)
    │   └── /primitive/ # Defines basic shapes (Point, Segment, Polygon)
    │
    └── /saves/
        ├── big_with_target.world # A default world file
        └── ...                   # Place your custom .world files here
```

## Quick start (run the simulator)

1) Open the repo folder in VS Code.
2) Open `index.html` in your browser.
   - You’ll see a “Load a world file” modal.
   - Choose an existing `.world` file (you can create one in the editor below) or click “Start with empty world”.
3) The simulator will load and start running N AI cars. The best-performing car is highlighted; its neural network is rendered in the right panel and a mini map below it.

Tips
- Top-left buttons in the simulator:
  - 🗃️ Save: store the current “best brain” to your browser localStorage
  - 🗑️ Discard: remove the saved brain from localStorage
  - 🌍 Pick a different world: clear world selection and reload
- A `saves/right_hand_rule.car` is pre-included and used by default to initialize cars in `main.js`.

## Create and edit a world

Open the world editor: `world/index.html`.

- Canvas controls
  - Mouse wheel: zoom
  - Middle mouse drag: pan
- Modes (toolbar buttons)
  - 🌐 Graph: draw your road network
  - 🛑 Stop, ⚠️ Yield, 🚶 Crossing, 🅿️ Parking, 🚦 Light, 🚙 Start, 🎯 Target: place markings aligned to lanes
- Graph editing
  - Left click: add/select a point
  - Left click on another point: create a road segment
  - Drag: move a selected point
  - Right click on a point: delete it
- Markings editing
  - Select a marking tool, then left click near a lane to place it (snaps to lane center)
  - Right click a marking to delete it
- Corridor preview (optional)
  - While in Graph mode, hover over lanes and press `s` to set a Start and `e` to set an End; a red path preview shows the shortest corridor between them
- OSM import (optional)
  - Click 🗺️ to open the OSM panel
  - Paste OSM JSON (e.g., from Overpass) and click ✔️ to parse it into a road graph
- Save your world
  - Click 💾 (Save). This downloads a `name.world` file and also stores the current world in localStorage

What is a .world file?
- A `.world` is a small JavaScript file that calls `World.load({...})`.
- In the simulator, you can pick a `.world` file from disk; the script is injected and sets a global `world` instance.

## Run the simulator with your world

- From the project root, open `index.html` in your browser.
- In the “Load a world file” modal, pick your saved `.world` file.
- Optionally, keep or remove the preloaded `saves/right_hand_rule.car` (see below).

No server needed
- Both the editor and simulator work from local files via the File API and localStorage.
- If you prefer running a local server (for caching or extension convenience), use VS Code’s Live Server extension or any static server.

## Training the car

The AI cars use a simple feed-forward neural network with binary activations. Training here means evolutionary search: spawn many cars from an existing brain, apply random mutations, and keep the best performer.

Where to tweak training
- Open `main.js`
  - `const N = 1000;` — number of cars to simulate in parallel (reduce for performance)
  - In the block loading `bestBrain` from localStorage, the mutation rate is set in `NeuralNetwork.mutate(cars[i].brain, 0.2)`
- Fitness
  - Each car’s `fittness` increases with distance traveled (proportional to speed per frame)
- Saving progress
  - Click 🗃️ in the simulator to save the current best brain to localStorage (`bestBrain`).
  - On next load, all cars clone that brain, and all but one receive a mutation.
  - Click 🗑️ to clear `bestBrain`.

About the sensors and network
- Rays: Each car has a forward-facing ray sensor (`sensor.js`) that measures distances to road borders and traffic; readings feed the network.
- Network: Defined in `network.js` (e.g., `[rayCount, 6, 4]`). Outputs map to controls: forward, left, right, reverse.
- Visualizer: The right panel (`visualizer.js`) animates activations and weights. Output nodes are labeled with arrows.

Targets and corridors
- If your world includes a 🎯 Target marking, the simulator builds a corridor from the Start to Target using the road graph’s shortest path. Road borders for collision are then derived from this corridor (red line overlay).
- Without a Target, the full road borders are used.
- Pathfinding: shortest paths are computed using Dijkstra’s algorithm on the road graph (see `world/js/math/graph.js`).

## Mini map

The mini map gives a zoomed-out top‑down view of the road graph:

- It renders the graph segments in white and keeps your car centered in the mini map.
- It tracks the camera/viewport, so as the best car moves, the mini map recenters accordingly.
- It’s optimized for readability (simplified lines, constant scale) and is not interactive.
- Implementation: see `miniMap.js` for how the world graph is scaled and drawn.

## How it works: the genetic algorithm

This project uses a lightweight genetic algorithm (not reinforcement learning):

- Population: on load, it spawns a population of N cars (configurable in `main.js`; default 1000). If there’s no saved brain in localStorage, all cars start with random neural networks.
- Fitness: each car’s `fittness` increases as it moves (proportional to its speed per frame). When a car crashes (`damaged = true`), its fitness stops increasing.
- Selection: the simulator continuously tracks the single best car (highest fitness) and centers the camera on it.
- Reproduction & mutation: when you click 🗃️ Save, the best car’s brain is persisted to localStorage (`bestBrain`). After you reload the page, the new generation clones that brain into all cars, and all but the first one get mutated via `NeuralNetwork.mutate(brain, amount)` (e.g., `0.2`).
- Evolution: repeat the cycle (run → save best → reload). Over time, the population converges toward better driving performance in your world.

Car brains (.car)
- `saves/right_hand_rule.car` is an example that encodes a car setup/brain. In `index.html`, it’s included via `<script src="saves/right_hand_rule.car"></script>`.
- In `main.js`, each spawned car calls `car.load(carInfo)`, which applies the brain and sensor parameters from that file.
- You can generate your own `.car` files offline or modify the included one; ensure it defines a global `carInfo` object.

## Troubleshooting

- Nothing moves or car spawns off-screen
  - Ensure your world has at least one 🚙 Start marking (this defines spawn position and direction)
- Can’t load a world
  - Use the file picker modal in the simulator; select a `.world` produced by the editor
  - To reset selection, click 🌍 in the simulator to clear the stored world and reload
- Poor performance
  - Lower `N` in `main.js` (e.g., 50–200) and/or mutation rate
- Changes didn’t apply
  - Clear browser localStorage for this site or use the 🌍 button to pick a new world and reload

## How to run (optional local server)

If you prefer a local server (not required), here are a few easy options:

- VS Code extension: Live Server — right-click `index.html` > “Open with Live Server”
- Node: `npx serve` (in the repo folder)
- Python: `python -m http.server 5500`

On Windows/PowerShell these commands run in an integrated terminal; adjust the port as you like.

## Contributing / customizing

- Worlds: Use the editor to author new layouts and markings; save and load into the simulator.
- Cars: Tweak `car.js` (physics), `sensor.js` (ray geometry), `network.js` (topology/mutation), and `main.js` (spawn count, mutation rate, fitness).
- Visuals: Adjust `style.css` and `world/styles.css`.

## Credits

This project is inspired by the classic "Self-Driving Car in JavaScript" tutorial series by Radu Mariescu-Istodor (Radu's YouTube channel), and has been extended with a full-featured world editor.

