---
title : "Jenga Village"
date : 2026-01-20
draft : false
image : "/img/projects/jenga-village/jenga-village-1.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/jenga-village/jenga-village-1.webp" >}}

**Jenga Village** is a playful yet serious exploration of modular housing. While modern apartment blocks often feel repetitive and isolating, this project asks: Can we use code to create a "vertical village" where every unit feels unique?

Using the metaphor of the game *Jenga*, the design stacks living units (voxels) in a seemingly random arrangement, creating a rich texture of solids and voids that adapts to the hillside topography.

## The Logic: Voxel Aggregation

Written in **Julia**, the script for this project operates on a voxel-based grid system. Unlike traditional modeling where we draw walls and floors, here the code treats space as a collection of 3D pixels.

{{< figure src="/img/projects/jenga-village/jenga-village-3.webp" >}}

The algorithm generates these stacks with specific rules:
1.  **Adjacency:** Units must connect logically to support one another.
2.  **Variation:** The script introduces random shifts to prevent the façade from becoming flat.
3.  **Subtraction:** Strategic removal of blocks creates pockets for open spaces.

## Breathing Room in High Density

The beauty of this algorithmic approach is the accidental quality of space it creates. By shifting the blocks, the roof of one unit becomes the terrace of the neighbor above.

This transforms a high-density complex into a series of garden homes. The "randomness" of the code ensures that almost every resident gets a private outdoor area, natural light, and a view—luxuries often lost in standard boxy apartments.

## Visualization

The scene was rendered using **Nano Banana**, focusing on the warmth of the materials to contrast the digital origin of the form.

{{< figure src="/img/projects/jenga-village/jenga-village-2.webp" >}}

I combined exposed concrete with timber cladding to give the voxels a tactile, human scale. The extensive greenery placed on the terraces softens the hard edges of the geometry, blending the architecture into the surrounding green hill. It is a vision of how computational design can coexist with nature.
