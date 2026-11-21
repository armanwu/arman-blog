---
title : "Eroded Monolith"
date : 2026-01-20
draft : false
description : "A high-rise typology study using spherical voxel subtraction in Julia to carve a stepped vertical park and Corten-lined sky terraces into an orthogonal corner tower."
tools : "Julia · Computational"
image : "/img/projects/eroded-monolith/eroded-monolith-exterior.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-exterior.webp" >}}

## Concept & The Porous High-Rise

**Eroded Monolith** challenges the sealed, air-conditioned box of conventional high-rise architecture. Positioned at a prominent urban street corner, the project asks what happens when we subtract a massive three-dimensional void out of a rigid modernist grid—opening the heart of the tower to daylight, cross-breezes, and stepped communal gardens.

## The Computational Logic: Carving with Distance

Written in **Julia**, the form-finding workflow uses a **subtractive** method rather than stacking blocks upward. As shown in the concept diagram below, the algorithm sculpts the tower by subtracting an invisible sphere from a solid grid of cubes:

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-process-diagram.webp" >}}

* **A Solid Voxel Tower (`11 × 11 × 21` Blocks):** The building begins as a complete 3D matrix of `2,541` cubic modules (`x` and `y` from `-5` to `5`, and `z` from `0` to `20`), establishing a disciplined orthogonal tower envelope.
* **The Spherical Cutter (`Radius = 7.0`):** An invisible cutting sphere is centered along the front corner edge at mid-height (`x = 5, y = 5, z = 10`). For every cube in the tower, the script measures its straight-line distance (`d`) to the center of the sphere and removes any block where `d ≤ 7.0`.
* **Progressive Real-Time Growth:** Using Julia's live visualization stream, the remaining blocks assemble floor by floor from the ground up and are shaded along an ice-blue gradient according to their distance from the carved void.

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-julia-mesh.webp" >}}

<details class="code-drawer">
<summary>View Julia Source Code (eroded_monolith.jl)</summary>

```julia
using GLMakie
using GeometryBasics
using LinearAlgebra

# Define the voxel grid boundaries
x_range = -5:5
y_range = -5:5
z_range = 0:20

# Define the spherical cutting tool
cutter_center = Point3f(5, 5, 10)
cutter_radius = 7.0

# Initialize observable data for real-time visualization
voxels_live = Observable(Point3f[])
colors_live = Observable(Float64[])

# Create the visualization figure and 3D scene
fig = Figure(size=(1000, 1000))
ax = LScene(fig[1, 1])

# Define the site boundary as a transparent wireframe box
site_bounds = Rect3f(Vec3f(-7, -7, 0), Vec3f(14, 14, 21))
wireframe!(ax, site_bounds, color=:grey30, linewidth=0.2, transparency=true)

# Visualize voxels using individual cube markers
# Color intensity represents the distance from the cutting tool center
meshscatter!(ax, voxels_live,
    marker=Rect3f(Vec3f(-0.5), Vec3f(1)),
    markersize=1.0,
    color=colors_live,
    colormap=:ice,
    colorrange=(0, 15),
)

# Display the initial visualization
display(fig)

# Execute the voxel generation process asynchronously
@async begin
    println("Starting voxel generation...")

    # Create temporary arrays to store voxel positions and colors
    temp_voxels = Point3f[]
    temp_colors = Float64[]

    # Iterate through the entire 3D voxel grid
    for z in z_range
        for x in x_range
            for y in y_range

                # Define the current voxel position
                point = Point3f(x, y, z)

                # Calculate the Euclidean distance from the cutting tool center
                distance = norm(point - cutter_center)

                # Retain voxels located outside the spherical cutting region
                if distance > cutter_radius
                    push!(temp_voxels, point)
                    push!(temp_colors, distance)

                    # Update the visualization in real time
                    voxels_live[] = temp_voxels
                    colors_live[] = temp_colors

                    # Introduce a short delay for progressive visualization
                    sleep(0.00001)
                end
            end
        end

        # Allow the rendering process to refresh between layers
        sleep(0.00001)
    end

    println("Voxel generation completed!")
end
```

</details>

## Architectural Translation: The Carved Vertical Park

While the Julia script simply filters out cubes within a `7.0-unit` radius, architectural interpretation turns that stepped mathematical cavity into a dramatic public and bioclimatic heart for the building:

* **Outer Grid vs. Inner Canyon:** The untouched outer faces of the tower keep a clean concrete-and-glass curtain wall anchored by dark stone piers at street level. Where the sphere cuts into the corner, the exposed voxel steps are clad in warm weathered Corten steel and board-formed concrete—visually expressing the "cut" as if a geological layer has been exposed.
* **Stepped Amphitheater Gardens:** The lower half of the carved sphere naturally forms a cascading series of planted terraces and outdoor stairs, while the upper half creates a dramatic stepped concrete soffit that shelters the gardens below from direct rain and midday sun.

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-sky-garden.webp" >}}

## Experience Inside the Void

Standing on the elevated timber deck inside the carved corner, the abstract voxel grid becomes a tangible spatial experience. Overhead, the stepped concrete ceiling recalls the cubic resolution of the Julia algorithm, while double-height glass walls connect the indoor workspaces directly to the planted terraces. Rather than isolating occupants behind a sealed glass facade, the eroded corner frames panoramic views down the tree-lined boulevard at sunset—turning a simple distance formula into a living vertical park.
