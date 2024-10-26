---
layout: post
title:  "Generating maps for gnomes: non-linear noise combinations"
date:   2024-10-13 00:00:00
categories: generation
image: gnome1.jpg
---

Perlin noise is a default go-to option if you need to generate a terrain for your game. Simple to understand, computationally light (as long as you know what you are doing), and is either already implemented in your favorite engine, or can be easily found and copypasted in a language you need. The results are usually reasonable, but way too boring and predictable if terrain plays a huge role in your game. Experiments and fine-tuning of parameters can only go so far towards making a remarkable terrian, so this post is about some more advanced techniques developers can utilize. They may be not perfect for all needs, but definitely better than an x-ray of a random cloud.

## Fractal noise aka linear combination

The easiest way to generate a somewhat realistic terrain is to use a *fractal noise*. Just slap together a few octaves with different frequencies and amplitudes and you are good to go:

![noise1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/noise1.png)

Where graphs left-to-right show:
- Projection function (will be explained in the next section)
- Grayscale highmap
- Color interpretation of the highmap

The only problem is that it looks too random and cloudy - terrains in real life are not like that. Fantasy maps are usually not so tedious as well. What lacks here is some kind of structure and meaning, and here's how multiple layers of noise can be defined:

1. Low frequency, high amplitude - large scale features, like mountains or large lakes
2. Moderate frequency, moderate amplitude - smaller landscape features: mounds, valleys, low hills
3. High frequency, low amplitude - holes, bumps, etc.

First create the character of the terrain: either it is a mountain range or a flat plain, or something different. Second one adds randomness to it, so no two hills look the same. Latter ones just add small details, which make terrain more interesting, otherwise player may get a feeling that they slide on a surface of some boring monotonic function.  

Obviously this is only one of infinite possible ways how to assign meaning to layers of noise. More layers can be added, more types of geographical features can be represented. But the price is increased computational complexity and additional time spent on fine-tuning the parameters. For now let's stick to this simple three layers model.

## Non-linear transformations

If you ever saw a mountain, you will probably agree that their surfaces dont look like *y=x* function. If so, why our heights have a linear relationship with the noise values?  
To understand how to come up with better function, let's use an illustration from nature lessons in primary school:

![book_landscape](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/water_circulation.jpg)

Here coast has it's own curve, flatlands another one, and mountains yet another. If to draw a function, which represents height above sea level, it will look like a set of disconnected segments, where each segment has it's own slope. Let's call it a `projection function`. In our game I used a function similar to this one:

```
def forestProjection(x):
    base = (x + 1) / 2
    scale = 0
    if base <= 0.4:
        scale = np.power(base / 0.4, 3) * 0.3
    elif base <= 0.8:
        scale = 0.3 + np.power((base - 0.4) / 0.4, 3) * 0.5
    else:
        scale = base

    smoothnessCoef = 0.2
    scale = scale * (1 - smoothnessCoef) + base * smoothnessCoef
    return scale
```

After defining all required projection functions we can combine them to archieve the final height map:

```
generalScale = 50
l1NoiseScale = 0.06 * generalScale
l2NoiseScale = 0.2 * generalScale
l3NoiseScale = 1 * generalScale

l1NoiseStrength = 6000.0
l2NoiseStrength = 500.0
l3NoiseStrength = 50.0

l1Generator = PerlinNoise2D(seed=101, scale=l1NoiseScale)
l2Generator = PerlinNoise2D(seed=102, scale=l2NoiseScale)
l3Generator = PerlinNoise2D(seed=103, scale=l3NoiseScale)

def forestNoise(x, y):
    return (forestProjection(l1Generator.noise(x, y)) * l1NoiseStrength \
        + linearProjection(l2Generator.noise(x, y)) * l2NoiseStrength \
        + linearProjection(l3Generator.noise(x, y)) * l3NoiseStrength) \
        / (forestProjection(1) * l1NoiseStrength + linearProjection(1) * l2NoiseStrength + linearProjection(1) * l3NoiseStrength)

```

And result is subjectively better: we have lakes with natural looking borders and better pronounced hills. Landscape doesn't look like a random cloud anymore.

![noise2](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/noise2.png)

## Multi-component noise

There's one thing that perlin noise is not very good at: it's too predictable. High and low values appear in rather regular intervals. You can look like a valley, or a ridge, but not a lonely mountain, or huge plateau. To create this kind of form we replace of the components (large scale to have the most impact) with a function of two perlin noises. Different functions can be used to create different shapes. For example `max` can be used to create a plateau, and `min` can be used to create a canyon. In this example we will use `clamp` of a sum of two perlin noises.

```
def mountainProjection(x):
    base = (x + 1) / 2
    base = max(base, 0)
    base = min(base, 1)
    scale = 0
    threshold = 0.6
    flatLevel = 0.2
    if base <= threshold:
        scale = base / threshold * flatLevel
    else:
        scale = flatLevel + np.power((base - threshold) / (1 - threshold), 2) * (1 - flatLevel)
    return scale

def mountainNoise(x, y):
    return (mountainProjection(l1c1Generator.noise(x, y) + l1c2Generator.noise(x, y)) * l1NoiseStrength \
        + linearProjection(l2Generator.noise(x, y)) * l2NoiseStrength \
        + linearProjection(l3Generator.noise(x, y)) * l3NoiseStrength) \
        / (mountainProjection(1) * l1NoiseStrength + linearProjection(1) * l3NoiseStrength)
```

Here we can clearly see two mountains in the middle of the grasslands. It can also be used, for example, as a deep ocean trench or high radiation zone.

![noise3](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/noise3.png)


## Experimenting with functions

In previous two sections we used projection functions inspired by real world geography. The less realistic projection function is - the harder it can be to interpret result as something meaningful. But this also opens a door to creating completely different landscapes. For example, if we use a sinusoid function to transform noise values, we can create something that looks like sand dunes:

```
def sinProjection(x):
    base = x * math.pi
    base = max(base, -math.pi / 2)
    base = min(base, math.pi / 2)
    return (np.sin(base) + 1) / 2
```
![noise4](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/noise4.png)

Going even further, we can make the projection function non-monotonic, for example, a mirrored exponential curve can produce something resembling flatlands with canals or rivers:

```
def spikeyProjection(x):
    base = x + 1
    base = max(base, 0)
    base = min(base, 2)
    scale = 0
    if base <= 1:
        scale = np.power(base, 3)
    else:
        scale = np.power((2 - base), 3)
    return 1 - scale
```
![noise5](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/perlin-noise/noise5.png)

I'm sure it is possible to produce many more interesting shapes with different projection functions, combinations of multiple noises and other mathematical tricks. I also tried to find some other interesting application of perlin noise out of map/image generation context, but didn't find anything. That's a pity.