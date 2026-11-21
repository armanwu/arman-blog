---
title : "Chaos Crystal"
date : 2026-01-20
draft : false
description : "An urban cultural landmark synthesized via spherical radial perturbation and grid-based triangulation in Julia, translating mathematical mesh topology into folded zinc, concrete, and diagrid glass tectonics."
tools : "Julia · Computational"
image : "/img/projects/chaos-crystal/chaos-crystal-exterior.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-exterior.webp" >}}

## Concept & Urban Tectonics

**Chaos Crystal** investigates how a purely mathematical coordinate transformation can be disciplined into a buildable civic landmark. Rather than sculpting deconstructivist forms by hand, the project begins with an analytical sphere and subjects it to controlled radial perturbation—producing an expressive crystalline silhouette that anchors a stepped urban plaza.

To ground its aggressive geometry within the city fabric, the building is organized as a tectonic dialogue between three primary systems: a heavy, board-formed concrete base that absorbs pedestrian circulation from the plaza; an interlocking opaque shell of matte charcoal titanium-zinc panels; and triangular structural glass apertures that slice across the folds to reveal the warm, multi-level public atrium within.

## The Computational Logic: From Sphere to Crystal

The form-finding workflow is written in **Julia**, transforming a simple mathematical sphere into a faceted building envelope through four clear geometric steps:

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-generative-process.webp" >}}

* **Coarse Spherical Grid (`12 × 12` Resolution):** Instead of a high-density mesh that would create noisy surface ripples, the script divides a sphere of radius `12.0` into a deliberate `12 × 12` grid. At this coarse scale, every grid cell naturally becomes the size of a full architectural facade facet.
* **Controlled Radial Push and Pull (`-5.0` to `+15.0`):** Every point on the sphere is pushed outward or pulled inward by a random distance between `-5.0` and `+15.0` units. Inward pulls carve deep alcoves and valleys, while outward pushes project dramatic cantilevered spires.
* **Flattening the Base on the Ground (`Z = 0`):** The entire crystal is lifted `+10.0` units upward, and any point falling below ground level is automatically flattened to `Z = 0`. This single rule turns the bottom of the floating sphere into a stable, flat floor plate on the plaza.
* **Closing the Surface with Flat Triangles:** Finally, the script stitches the seams and poles together and divides every four-sided grid cell into two flat triangular faces—ensuring a completely closed, buildable volume.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-julia-mesh.webp" >}}

<details class="code-drawer">
<summary>View Julia Source Code (chaos_crystal.jl)</summary>

```julia
using GLMakie
using GeometryBasics
using Random
using Colors

# Create the 3D drawing environment
fig = Figure(size=(800, 800))
ax = Axis3(
    fig[1, 1],
    aspect=:data,
    azimuth=(-π / 2),
    elevation=0,
    perspectiveness=0
)

hidedecorations!(ax)
hidespines!(ax)

# Create the ground plane
wireframe!(
    ax,
    Rect3f(Point3f(-30, -30, 0), Vec3f(60, 60, 0)),
    color=(:white, 0.1),
    linewidth=0.0
)

# Generate the Chaos Crystal geometry
function chaos_crystal()

    # Define the main parameters
    resolution = 12
    base_radius = 12

    # Initialize mesh data
    points = Point3f[]
    faces = GLTriangleFace[]

    # Create a grid to store 3D points
    grid_points = Matrix{Point3f}(undef, resolution, resolution)

    # Generate points using spherical coordinates
    for i in 1:resolution
        for j in 1:resolution

            theta = π * (i - 1) / (resolution - 1)
            phi = 2π * (j - 1) / (resolution - 1)

            # Apply random radius variation
            random_radius = base_radius + rand(-5.0:15.0)

            # Convert spherical coordinates into Cartesian coordinates
            x = random_radius * sin(theta) * cos(phi)
            y = random_radius * sin(theta) * sin(phi)
            z = random_radius * cos(theta)

            # Raise the geometry above ground
            final_z = max(z + 10, 0)

            grid_points[i, j] = Point3f(x, y, final_z)
        end
    end

    # Close the spherical surface
    for i in 1:resolution
        grid_points[i, resolution] = grid_points[i, 1]
    end

    # Connect the top and bottom poles
    for j in 1:resolution
        grid_points[1, j] = grid_points[1, 1]
        grid_points[resolution, j] = grid_points[resolution, 1]
    end

    # Connect neighboring points to create triangular faces
    for i in 1:(resolution-1)
        for j in 1:(resolution-1)

            p1 = grid_points[i, j]
            p2 = grid_points[i, j+1]
            p3 = grid_points[i+1, j+1]
            p4 = grid_points[i+1, j]

            push!(points, p1, p2, p3, p4)

            # Divide each quadrilateral into two outward-facing triangles
            current = length(points) - 3

            push!(
                faces,
                GLTriangleFace(current, current + 3, current + 2),
                GLTriangleFace(current + 2, current + 1, current)
            )
        end
    end

    # Return the final 3D mesh
    return GeometryBasics.Mesh(points, faces)
end

# Generate the crystal
crystal_mesh = chaos_crystal()

# Display the solid surface
mesh!(
    ax,
    crystal_mesh,
    color=(RGB(0.50, 0.70, 0.68), 0.9),
    shading=true
)

# Display the mesh edges
wireframe!(
    ax,
    crystal_mesh,
    color=(:black, 0.28),
    linewidth=1.2
)

# Render the final result
display(fig)
```

</details>

## Why Triangles Matter: From Elevation to Physical Model

A classic challenge in freeform architecture is building complex surfaces without expensive curved glass or twisted metal panels. Because a four-sided polygon almost always warps when its corners move randomly, the Julia script explicitly splits every grid cell into **triangles**.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-elevation-drawing.webp" >}}

In the script, the 3D camera is set to a pure orthographic front view—allowing the raw Julia output to translate directly into a scaled architectural elevation drawing. Resting firmly on the flat `Z = 0` ground line, the large triangular facets are broken down into human-scaled building systems: a board-formed concrete base at street level, standing-seam titanium-zinc panels across the solid peaks, and subdivided curtain-wall glazing.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-maquette.webp" >}}

Because any three points in space always form a perfectly flat plane, the entire building skin can be cut from flat sheets. As shown in the physical study model above—built from grey museum board, flat acrylic glazing, and a stepped basswood plaza—the geometry from Julia translates directly into physical construction without a single curved panel.

## Interior Space & Light

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-interior.webp" >}}

Inside, the sharp exterior shell opens up into a bright, multi-level public atrium where the triangular geometry becomes the main structural expression:

* **Sloped Concrete Piers & Skybridges:** Massive cast-in-place concrete piers support the upper exhibition galleries, linked across the central void by slender steel bridges that let visitors experience the space from different heights.
* **Triangulated Skylights & Acoustic Ceilings:** Where the shell opens toward the sky, angled steel mullions follow the triangular mesh lines, while the solid peaks are lined with warm-lit acoustic panels that bounce daylight deep into the concourse.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-perimeter-promenade.webp" >}}

Along the outer edge of the building, the outward-pushed points of the crystal become triple-height viewing galleries where walkways and stairs meet right behind the glass facade. From here, the steel diagrid joints frame diagonal views back down to the stepped plaza and out toward the city skyline while casting long afternoon shadow patterns across the floor.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-interior-mezzanine.webp" >}}

From the upper mezzanine walkway, the relationship between the outer shell and the inner concrete frame becomes clear. Inclined concrete columns support the cantilevered gallery floors, while a central stair and escalator spine draws visitors upward through the folded volume.

## Day-to-Night Transformation

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-night-plaza.webp" >}}

While the Julia script only calculates spherical coordinates and random radial distances, architectural interpretation turns that raw mesh into an experience of light and time. By day, the crystal reads as a heavy, sculptural monolith carved by shadow. At dusk, that reading flips completely—warm light from the galleries glows outward through the triangular glass facets while subtle linear lights trace the folded seams, turning the building into a welcoming lantern on the public plaza.
