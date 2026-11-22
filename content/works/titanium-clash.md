---
title : "Titanium Clash"
date : 2026-01-20
draft : false
description : "An experimental deconstructivist massing study in Julia that collides eight tetrahedral shards into a cantilevered civic landmark clad in brushed titanium and inclined curtain walls."
tools : "Julia · Generative"
image : "/img/projects/titanium-clash/titanium-clash-exterior.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/titanium-clash/titanium-clash-exterior.webp" >}}

## Concept & Spatial Collision

**Titanium Clash** explores the boundary between algorithmic randomness and architectural order. Drawing inspiration from deconstructivist form-making, the project tests how a simple script can generate high-energy spatial collisions—where sharp, soaring spires pierce and counter-balance one another above a calm reflecting pool.

## The Computational Logic: Stochastic Shard Aggregation

Instead of starting with a manual sketch or a clay model, this study begins with a script in **Julia** that builds a complex deconstructivist massing by aggregating eight irregular tetrahedral shards around a shared origin:

{{< figure src="/img/projects/titanium-clash/titanium-clash-process-diagram.webp" >}}

* **Tetrahedral Geometry Logic (`p1, p2, p3` + `apex`):** Each shard is constructed from four randomized points—three base vertices scattered near ground level (`x, y ∈ [-35, 35]`, `z ∈ [-5, 5]`) and one elevated apex projected high into the air (`x, y ∈ [-55, 55]`, `z ∈ [60, 85]`). Connecting these four points forms an irregular four-face tetrahedron whose surfaces are always flat triangles.
* **Sequential Aggregation Loop (`1` to `8` Shards):** In an iterative loop (`i = 1` to `8`), eight independent shards are generated around `Point3f(0, 0, 0)`. As the loop progresses from a single pyramid to the full eight-shard cluster, the leaning volumes pierce and interlock with one another—rendered in a four-tone metallic slate palette with crisp white wireframe edges.

{{< figure src="/img/projects/titanium-clash/titanium-clash-julia-mesh.webp" >}}

<details class="code-drawer">
<summary>View Julia Source Code (titanium_clash.jl)</summary>

```julia
using GLMakie
using GeometryBasics
using Random
using Colors

# Initialize the 3D visualization environment
fig = Figure(size=(800, 800))

# Configure the 3D camera and viewing angle
ax = Axis3(fig[1, 1],
    aspect=:data,
    perspectiveness=0.2,
    azimuth=0.35π,
    elevation=0.15π
)

# Hide axis decorations and spines
hidedecorations!(ax)
hidespines!(ax)

# Define the spatial boundaries
limits!(ax, -60, 60, -60, 60, 0, 90)

# Display the visualization window
display(fig)

# Generate an aggressive shard geometry
function create_aggressive_shard(center)

    # Generate three randomly displaced base vertices
    p1 = center + Point3f(rand(-35:35), rand(-35:35), rand(-5:5))
    p2 = center + Point3f(rand(-35:35), rand(-35:35), rand(-5:5))
    p3 = center + Point3f(rand(-35:35), rand(-35:35), rand(-5:5))

    # Generate a randomly displaced elevated apex
    apex = center + Point3f(rand(-55:55), rand(-55:55), rand(60:85))

    # Assemble the four vertices of the shard
    vertices = [p1, p2, p3, apex]

    # Define the triangular faces of the geometry
    faces = [
        TriangleFace(1, 2, 4),
        TriangleFace(2, 3, 4),
        TriangleFace(3, 1, 4),
        TriangleFace(1, 2, 3)
    ]

    # Construct and return the triangular mesh
    return GeometryBasics.Mesh(vertices, faces)
end

# Define the architectural shard color palette
shard_palette = [
    RGBAf(0.5, 0.6, 0.7, 0.95),
    RGBAf(0.3, 0.4, 0.5, 0.95),
    RGBAf(0.7, 0.8, 0.9, 0.9),
    RGBAf(0.2, 0.2, 0.3, 1.0)
]

# Generate and visualize eight randomly shaped shards
@async begin
    for i in 1:8

        # Generate a new shard at the origin
        shard = create_aggressive_shard(Point3f(0, 0, 0))

        # Render the shard with a randomly selected color
        mesh!(ax, shard,
            color=rand(shard_palette),
            shading=true
        )

        # Highlight the triangular edges with white wireframes
        wireframe!(ax, shard,
            color=:white,
            linewidth=2.0
        )

        # Introduce a short delay for sequential generation
        sleep(0.3)
    end
end
```

</details>

## The Role of the Architect: Curation & Physical Model

While the computer generates raw geometric collisions in seconds, the role of the architect shifts from manual drafter to **spatial curator**. Running the script dozens of times produces countless variations—some too fragmented, others too flat. The final composition was selected because its leaning spires achieve a dynamic visual equilibrium.

{{< figure src="/img/projects/titanium-clash/titanium-clash-maquette.webp" >}}

Because every shard is a pure four-point tetrahedron, every single surface in the composition is a flat triangle. As demonstrated in the physical study model above—built from matte metallic grey board, blue-tinted acrylic glazing panels, and a contoured plywood site base—these colliding digital shards can be unrolled and fabricated entirely from flat sheet materials without any curved panels.

## Interior Atrium & Architectural Translation

{{< figure src="/img/projects/titanium-clash/titanium-clash-interior.webp" >}}

Because the Julia script only outputs raw tetrahedral meshes and color tones, translating the computational cluster into architecture involves four deliberate design interpretations:

* **Titanium-Clad Planes & Inclined Glazing:** The opaque gray and graphite faces of the digital mesh are interpreted as brushed titanium cladding panels, while the ice-blue facets become soaring, inclined glass curtain walls that flood the public lobby with daylight.
* **Exposed Trusses & Helical Staircase:** Overhead, long-span steel roof trusses and suspended polyhedral volumes reveal how the cantilevered spires lock together structurally, while a sweeping stainless-steel spiral staircase rises through the atrium as a fluid counterpoint to the sharp exterior angles.

{{< figure src="/img/projects/titanium-clash/titanium-clash-exterior-night.webp" >}}

* **White Edge Highlighting:** At night, the white wireframe overlay plotted in the Julia script (`linewidth = 2.0`) is reimagined as recessed linear LED lighting along the sharp ridge lines of the titanium shards—turning the computational wireframe into a glowing nocturnal silhouette reflected across the water basin.
