---
title : "Eroded Monolith"
date : 2026-01-20
draft : false
description : "A high-rise typology study using algorithmic voxel subtraction in Julia to carve communal vertical gardens into a modernist tower."
tools : "Julia · Computational"
image : "/img/projects/eroded-monolith/eroded-monolith-1.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-1.webp" >}}

**Eroded Monolith** challenges the traditional typology of the skyscraper. Usually, high-rise buildings are sealed, air-conditioned glass boxes that isolate people from the elements. This project asks: What happens if we "break" the box to let nature in?

The concept creates a dialogue between two opposing forces: the perfection of the modernist grid versus the imperfection of organic erosion.

## The Computational Logic: Carving with Distance

The form-finding process is driven by a script in **Julia**. Unlike additive workflows that stack blocks upward, this project uses a **subtractive** method—carving space out of a solid tower volume:

* **A Solid Tower Grid (`11 × 11 × 21` Blocks):** The building starts as a regular 3D grid of cubes (`11 × 11` units wide and `21` stories tall), representing a standard orthogonal office tower.
* **Carving with an Invisible Sphere:** An invisible spherical "cutting tool" (radius `7.0`) is placed along the corner of the tower at mid-height (`x = 5, y = 5, z = 10`). As the script checks every block in the grid, it measures the straight-line distance from that block to the center of the sphere—deleting any block that falls inside the sphere's radius.
* **Floor-by-Floor Growth:** As the code runs, it builds the remaining blocks layer by layer from the ground up and colors each block according to its distance from the carved void, revealing a stepped spherical canyon cut deep into the corner of the monolith.

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-2.webp" >}}

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

## Tectonic Contrast & Vertical Sky Gardens

In translating the eroded voxel matrix into high-rise architecture, the design emphasizes the tectonic duality between the intact perimeter and the excavated void:

* **The Orthogonal Curtain-Wall Shell:** The un-eroded faces retain a disciplined, reflective glass curtain wall that responds to the urban street grid and maximizes floor-plate efficiency.
* **The Excavated Biophilic Core:** Where the spherical boolean operation carves into the corner mass (`Point3f(5, 5, 10)`), the exposed stepped voxel terraces are clad in warm timber soffits and planted as multi-level sky gardens.

{{< figure src="/img/projects/eroded-monolith/eroded-monolith-3.webp" >}}

This algorithmic excavation functions as a bioclimatic lung for the tower—introducing deep self-shading cantilevers, natural cross-ventilation pockets, and communal outdoor terraces that reconnect high-rise occupants with the vertical landscape.
