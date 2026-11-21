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

## The Computational Logic: Parametric Arches

Written in **Julia**, the script builds the shell by generating a row of semicircular arches along a straight spine and connecting them into a continuous surface:

{{< figure src="/img/projects/flux-vaults/flux-vaults-process-diagram.webp" >}}

* **A Row of 30 Arches (`100 m` Length):** A `100-meter` axis is divided into `30` equally spaced arch ribs, with each arch curve plotted using `30` points from one side to the other.
* **Varying Width, Height, and Shift:** Instead of repeating identical arches, the script randomly varies three dimensions for each rib—its span width (`5` to `15` units), its peak height (`4` to `20` units), and its sideways shift (`-5` to `+5` units).
* **Grounded Arch Geometry:** Each arch curve is drawn using simple sine and cosine waves. Because a half-sine wave always starts and ends at zero, every arch—no matter how tall, wide, or shifted—lands both of its feet cleanly on the flat ground plane (`Z = 0`).
* **Connecting Ribs into a Shell:** Finally, the script connects each arch to its neighbor with a grid of flat triangles, creating a continuous vaulted roof structure.

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

## From Wireframe Mesh to Physical Model

To turn the continuous digital mesh into a buildable roof structure, neighboring ribs are grouped into overlapping vault shells separated by crescent-shaped glass skylights.

{{< figure src="/img/projects/flux-vaults/flux-vaults-maquette.webp" >}}

As shown in the physical study model above—built from layered grey museum-board arches, segmented white shell panels, basswood bases, and radial acrylic glazing—dividing each arch into `30` straight segments allows the curved vaults to be constructed from simple flat panel strips and standard arch ribs.

## Interior Concourse & Evening Atmosphere

{{< figure src="/img/projects/flux-vaults/flux-vaults-interior.webp" >}}

Inside the building, the height differences (`4` to `20` meters) and sideways shifts between consecutive arches become the main architectural feature:

* **Monumental Concrete Arches:** Thick board-formed concrete arch ribs carry the roof loads down to the floor while framing a wide, column-free public hall with escalators and an upper mezzanine bridge.
* **Crescent Skylights:** Wherever one arch rises higher or shifts sideways next to its neighbor, the gap between them is filled with curved clerestory glass—washing the concrete vaults with natural daylight and opening views out to the surrounding water.

{{< figure src="/img/projects/flux-vaults/flux-vaults-exterior-night.webp" >}}

While the Julia script simply randomizes arch widths, heights, and offsets along a line, architectural interpretation turns those numbers into a physical rhythm of expansion and compression as people walk through the `100-meter` hall. At dusk, warm uplighting washes the underside of the concrete arches while interior light glows through the glazed entrance and crescent skylights, mirroring the vaults across the still water basin.
