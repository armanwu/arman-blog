---
title : "Flux Vaults"
date : 2026-01-20
draft : false
description : "A civic cultural concourse synthesized from parametric sine and cosine arch modulations in Julia, translating mathematical rib-lofting into interlocking concrete vaults and crescent clerestories."
tools : "Julia · Parametric"
image : "/img/projects/flux-vaults/flux-vaults-exterior.webp"
aliases : ["/works/flux-paviliun/", "/works/flux-pavilion/"]
layout : "single-no-meta"
---

{{< figure src="/img/projects/flux-vaults/flux-vaults-exterior.webp" >}}

## Concept & Vaulted Tectonics

**Flux Vaults** investigates how serial mathematical variation can generate a monumental vaulted civic concourse. Rather than treating a long-span roof as a uniform extrusion, the project conceives the building envelope as a rhythmic sequence of offset semicircular shells that rise, widen, and shift laterally along a 100-meter axis—creating an interlocking topography of board-formed concrete arches and glazed clerestory crescents poised above a reflecting water basin.

## The Computational Pipeline: Parametric Rib Lofting

Written in **Julia** (`GLMakie` and `GeometryBasics`), the generative algorithm synthesizes the shell by lofting a series of trigonometrically parameterized arches across a longitudinal spine:

{{< figure src="/img/projects/flux-vaults/flux-vaults-process-diagram.webp" >}}

* **Serial Transverse Rib Array (`L = 100`, `N = 30`, `R = 30`):** The 100-unit longitudinal axis (`building_length = 100`) is subdivided into `rib_count = 30` transverse arch stations, each discretized into `curve_resolution = 30` angular increments over `θ ∈ [0, π]`.
* **Stochastic Amplitude, Span & Axis Modulation:** At each station `i`, three geometric parameters are varied independently—half-span width (`wᵢ ∈ [5.0, 15.0]`), apex height (`hᵢ ∈ [4.0, 20.0]`), and lateral centerline drift (`Δyᵢ ∈ [-5.0, 5.0]`).
* **Trigonometric Arch Projection (`cos θ` & `sin θ`):** Each rib profile is computed via `y = Δyᵢ - (wᵢ * cos(θ))` and `z = hᵢ * sin(θ)`. Because `sin(0) = 0` and `sin(π) = 0`, every arch—regardless of its randomized height or lateral drift—automatically anchors both springing points cleanly onto the `Z = 0` ground datum.
* **Rib-to-Rib Triangulated Lofting (`GLTriangleFace`):** Adjacent ribs (`current_rib` and `next_rib`) are stitched together segment by segment, subdividing each four-node bay `(p1, p2, p3, p4)` into two planar triangles to form a continuous structural shell.

{{< figure src="/img/projects/flux-vaults/flux-vaults-julia-mesh.webp" >}}

<details class="code-drawer">
<summary>View Julia Source Code (flux_vaults.jl)</summary>

```julia
using GLMakie
using GeometryBasics
using Colors
using Random

# Initialize the 3D visualization environment
fig = Figure(size=(1000, 600), backgroundcolor=:grey10)
ax = Axis3(fig[1, 1], aspect=:data, azimuth=0.4π, elevation=0.2π)

hidedecorations!(ax)
hidespines!(ax)

# Create the ground plane
wireframe!(ax, Rect3f(Point3f(-10, -30, 0), Vec3f(120, 60, 0)),
    linewidth=0.0
)

# Generate a parametric building structure using a series of irregular ribs
function generate_building()

    # Define the main geometric parameters
    building_length = 100
    rib_count = 30
    curve_resolution = 30

    # Initialize geometry containers
    points = Point3f[]
    faces = GLTriangleFace[]
    ribs_data = Vector{Point3f}[]

    # Generate individual curved ribs along the building length
    for i in 1:rib_count

        # Define the position and random dimensions of each rib
        position_x = (i - 1) * (building_length / rib_count)
        width = rand(5.0:15.0)
        height = rand(4.0:20.0)
        offset_y = rand(-5.0:5.0)

        # Generate points along a semicircular profile
        rib_points = Point3f[]

        for j in 1:curve_resolution
            theta = π * (j - 1) / (curve_resolution - 1)

            y = offset_y - (width * cos(theta))
            z = height * sin(theta)

            push!(rib_points, Point3f(position_x, y, z))
        end

        # Store the generated rib geometry
        push!(ribs_data, rib_points)
    end

    # Connect adjacent ribs to generate the building surface
    index_counter = 1

    for i in 1:(rib_count-1)

        current_rib = ribs_data[i]
        next_rib = ribs_data[i+1]

        # Construct triangular faces between neighboring ribs
        for j in 1:(curve_resolution-1)

            p1 = current_rib[j]
            p2 = next_rib[j]
            p3 = next_rib[j+1]
            p4 = current_rib[j+1]

            # Add vertices to the geometry
            push!(points, p1, p2, p3, p4)

            # Define two triangular faces for each surface segment
            push!(faces, GLTriangleFace(index_counter, index_counter + 1, index_counter + 2))
            push!(faces, GLTriangleFace(index_counter + 2, index_counter + 3, index_counter))

            index_counter += 4
        end
    end

    # Return the completed building mesh
    return GeometryBasics.Mesh(points, faces)
end

# Generate the building geometry
building_mesh = generate_building()

# Render the translucent building surface
mesh!(ax, building_mesh,
    color=(:teal, 0.5),
    transparency=true,
    shading=true
)

# Display the structural wireframe
wireframe!(ax, building_mesh,
    color=(:white, 0.4),
    linewidth=1.0
)

# Display the final 3D visualization
display(fig)
```

</details>

## Tectonic Rationalization & Physical Study Model

To bridge the continuous triangulated mesh (`GeometryBasics.Mesh`) and real-world construction, consecutive arch ribs are clustered into distinct vault shells separated by crescent-shaped clerestory setbacks.

{{< figure src="/img/projects/flux-vaults/flux-vaults-maquette.webp" >}}

As explored in the physical architectural scale model above—constructed from layered grey museum board arches, segmented white shell cladding, basswood foundation plinths, and radially mullioned acrylic glazing—discretizing each arch into `R = 30` linear chords allows the curved vaults to be assembled from standardized, developable panel strips and planar structural ribs.

## Interior Concourse & Nocturnal Reflection

{{< figure src="/img/projects/flux-vaults/flux-vaults-interior.webp" >}}

In translating the parametric rib sequence into a full-scale civic building, the differential height (`hᵢ ∈ [4.0, 20.0]`) and lateral displacement (`Δyᵢ ∈ [-5.0, 5.0]`) between consecutive arches are exploited as a primary architectural device:

* **Monumental Concrete Arch Ribs:** Primary board-formed concrete arches transfer roof loads down to the concourse perimeter while framing an expansive, column-free public hall, mezzanine bridge, and vertical escalator circulation.
* **Crescent Clerestory Apertures:** Where adjacent ribs shift in radius and elevation, the resulting offset gaps are glazed as curved clerestory ribbons and lateral curtain walls—washing the acoustic plaster soffits with daylight and opening views out to the surrounding reflecting pool.

{{< figure src="/img/projects/flux-vaults/flux-vaults-exterior-night.webp" >}}

At dusk, warm grazing uplights illuminate the deep concrete arch intrados while interior concourse lighting radiates through the radial entrance portal and crescent clerestories—mirroring the rhythmic sequence of vaults across the still water basin.
