can you 

# docs/entropy_cube.md

# Project 1: The 'Entropy Cube'

This is a box. It has tiny balls in it. The box is *also* made of a bunch of smaller boxes, and they light up based on how many balls are inside them.

That's pretty much it. My simple way of "seeing" entropy. If all the balls are in one corner (low entropy), that corner will be bright red. If they're all spread out (high entropy), the whole box will be a nice, even color.
I thought about using a python package called fast_boltzmann to actually calculate the entropy, but I never actually got around to doing it. Maybe another time.

## Anyway. Go play with it!


<iframe src="https://www.glowscript.org/#/user/ciuli06ciuli/folder/MyPrograms/program/entropycube/embed" width="800" height="800" sandbox="allow-scripts allow-same-origin" style="border: 1px solid #ccc; border-radius: 8px;"></iframe>

## How It Works

### 1. Making the Grid (The Hard Way)

At first, I just made a bunch of cubes using VPython's `box()` object. Super simple. **This was a mistake.**

I realized I couldn't color each *individual* cube. The solution was a pain, but it works: I had to build every single tiny cubelet *manually* out of 6 faces. Each "face" is just a `box()` object that's really, really thin.

```python
# This is inside a giant triple-loop (for i, j, k)
CUBELET_SIZE = CONTAINER_SIZE / GRID_DIMENSION
FACE_THICKNESS = 0.05
hs = CUBELET_SIZE / 2 # hs just stands for "half-size"

faces = {
    'left':   box(pos=pos - vector(hs, 0, 0), size=vector(FACE_THICKNESS, CUBELET_SIZE, CUBELET_SIZE), opacity=0.2),
    'right':  box(pos=pos + vector(hs, 0, 0), size=vector(FACE_THICKNESS, CUBELET_SIZE, CUBELET_SIZE), opacity=0.2),
    'bottom': box(pos=pos - vector(0, hs, 0), size=vector(CUBELET_SIZE, FACE_THICKNESS, CUBELET_SIZE), opacity=0.2),
    # ...and so on for top, back, and front
}
```
This way, I can store all 6 faces for grid[i][j][k] and then color them however I want.


### 2. The "Shell View"
Fun little button. How do you see inside? Just make all the internal faces invisible. Just checks which faces are on the edge

Simple, but it looks pretty cool.

### 3. The "Entropy" Color
This is the main logic, running every single frame:

Count: Make a big 3D list full of zeros. Loop through every particle, figure out which (i, j, k) cubelet it's in, and add +1 to that cube's counter.

Calculate: Loop through all the counts. I use a simple formula (-prob * log(prob)) to get an "entropy" value. I also find the highest entropy value in the whole grid (max_entropy).

Color: I "normalize" the value by dividing it by max_entropy. This gives me a number between 0 and 1. I turn that number into a color (0 is blue, 0.5 is green, 1 is red).

Apply: I take that color and apply it to all 6 faces of that cubelet.

And that's it! The particles just bounce off the walls, and the grid just colors itself based on where they are.