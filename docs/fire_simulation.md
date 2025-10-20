# docs/fire_simulation.md

# Project 2: The Fire Sim

This one was a lot harder, still fun tho. It's a particle simulation that actually tries to look and "feel" like a campfire.

It's got physics, color blending, and a few clever tricks to run fast and look good. It even calculates the *real* Boltzmann Entropy of the fire, which is some pretty high-level stuff.

## Play With It!
Make sure to toggle "Show Entropy Bins", it took to long to implement for noone to see it


<iframe src="https://www.glowscript.org/#/user/ciuli06ciuli/folder/MyPrograms/program/firesim/embed" width="800" height="800" sandbox="allow-scripts allow-same-origin" style="border: 1px solid #ccc; border-radius: 8px;"></iframe>

## The Interesting Bits

I learned some cool stuff making this

### 1. Particle Pooling (aka Recycling)

If you have 2000 particles, creating a new one and destroying an old one every frame is **slow**. It will kill your browser.

**The Solution:** Particle Pooling. Never destroy a particle.
When a particle gets old and "dies" (its `age` is greater than its `lifetime`), we just call `reset_particle(p)`. This function instantly teleports it back to the bottom of the fire, gives it a new random velocity and lifetime, and makes it visible again.

It *looks* like a new particle, but it's just an old one being recycled. It's way, way faster.

### 2. The Bell Curve

If you just spawn particles at random spots, the fire looks like a square block. Real fire is hotter and denser in the middle.

You need a **Gaussian (bell curve) distribution** for your spawn points. The problem? Glowscript's `random` module doesn't have a built-in function for this.

**The Solution:** The **Box-Muller Transform**. It's a weird math trick that uses two *normal* random numbers (from `random()`) to create two *Gaussian* random numbers. I just copied it from a YouTube video, and it works perfectly. It makes particles spawn in the middle way more often than the edges.

```python
# This is inside the reset_particle() function
while True:
    r1 = random() + 1e-10  # Can't be zero
    r2 = random()
    
    # The magic math voodoo
    px = GAUSS_STD_DEV * sqrt(-2 * log(r1)) * cos(2 * pi * r2)
    pz = GAUSS_STD_DEV * sqrt(-2 * log(r1)) * sin(2 * pi * r2)
    
    p.pos = vector(px, 0, pz)
    
    # Keep trying until we get one inside the hearth
    if mag(p.pos) <= hearth_radius:
        break
```
### 3. The Colors (aka "Lerp")
How do you smoothly blend from yellow to red? Or from white to blue for a magic fire? You use **Linear Interpolation**, or "Lerp". It's just a function that finds the color between two other colors.

```
python

def mix_colors(color1, color2, fraction):
    """Blends two colour vectors. 'fraction' (0-1) is the mix amount."""
    fraction = max(0, min(1, fraction))
    return color1 * (1 - fraction) + color2 * fraction
```
I use this to blend the start/end colors based on the fire_power slider, and then I use it again to blend that color based on the particle's height.

### 4. The Real Math: Boltzmann Entropy
This was the coolest part. I wanted to calculate the actual entropy of the fire.

!!! info "The Nerdy Stuff I'm not sure even I fully understand

**The Concept:** The famous formula is $S = k_B \ln(W)$. We ignore $k_B$ (it's a constant). We just need to find $W$, which is the number of ways you can arrange the particles.

**The Problem:** The formula for $W$ is $W = N! / (N_1! \cdot N_2! \cdot ...)$ where $N$ is the total particle count and $N_1$, $N_2$ are the counts in each little 1x1x1 bin.

This is **impossible** to calculate. $1000!$ (1000 factorial) is a number with 2,568 digits. Your computer will melt.

**The Solution: Stirling's Approximation.**
We don't need $W$, we just need $\ln(W)$. A very smart dead guy named Stirling figured out an approximation: $\ln(N!) \approx N \ln(N) - N$.

If you plug this into the formula for $\ln(W)$, all the annoying $-N$ terms magically cancel out!

**The Final Formula:**
You are left with this beautiful, simple equation that's super fast to calculate:

$S \approx N \ln(N) - \sum(N_i \ln(N_i))$

And that's exactly what the code does!
```python
# N * ln(N)
n = len(particles)
entropy = n * log(n) 

# Subtract the Sum(Ni * ln(Ni))
sum_ni_ln_ni = 0
for ni in bins.values(): # bins is just a list of counts in each box
    if ni > 0:
        sum_ni_ln_ni += ni * log(ni)
        
entropy -= sum_ni_ln_ni
entropy_label.text = f'Boltzmann Entropy (S): {entropy:.2f}'
```
Resources That Helped Me
I didn't invent this, but I learned a lot of stuff. I mostly just watched YouTube. These might help you too.

What is Linear Interpolation? (Lerp) https://www.youtube.com/watch?v=YJB1QnEmlTs

What is the Box-Muller Transform? https://www.youtube.com/watch?v=4fVQrH65aWU

Particle Pooling (Concept) https://www.youtube.com/watch?v=9dp0mAc2vvY