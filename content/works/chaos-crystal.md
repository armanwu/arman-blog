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

## The Computational Pipeline

The form-finding workflow is written entirely in **Julia** (`GeometryBasics` and `GLMakie`), translating spherical trigonometry into a watertight, triangulated surface mesh through four sequential operations:

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-generative-process.webp" >}}

* **Discrete Spherical Parametrization (12 × 12 Grid):** Instead of a high-density mesh that would yield noisy surface ripples, the script locks the polar (`θ ∈ [0, π]`) and azimuthal (`φ ∈ [0, 2π]`) domains to a deliberate `12 × 12` resolution with a base radius of `r₀ = 12.0`. At this coarse resolution, every grid cell naturally assumes the scale of a primary architectural facade module.
* **Asymmetric Radial Perturbation (`Δr ∈ [-5.0, +15.0]`):** Each vertex radius is displaced stochastically via `r = r₀ + Δr`, spanning an actual radial envelope of `7.0` to `27.0` units. Inward compressions (`-5.0`) carve deep re-entrant valleys, while outward extensions (`+15.0`) project dramatic cantilevered spires.
* **Cartesian Transformation & Ground-Datum Clamping (`Z = 0`):** Once projected into Cartesian space `(x, y, z)`, the entire geometry is shifted upward by `+10.0` units and clamped against the ground plane using `z = max(r cos θ + 10, 0)`. This single mathematical constraint flattens the lower hemisphere into a stable, load-bearing architectural footprint.
* **Manifold Seam Closure & Grid-Based Triangulation:** To ensure a closed solid without naked edges, the azimuthal seam (`φ = 2π → 0`) and polar apexes are stitched before each quadrilateral cell `(p1, p2, p3, p4)` is subdivided into two outward-facing `GLTriangleFace` elements.

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

## Planar Triangulation & Tectonic Rationalization

A recurring challenge in computational architecture is translating stochastic, freeform meshes into constructible building envelopes. Because four-point quadrilateral faces almost always warp into non-planar surfaces under random vertex perturbation, the script explicitly triangulates every cell into rigid three-point `GLTriangleFace` primitives.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-elevation-drawing.webp" >}}

Notice how the `GLMakie` viewport in the script is configured as a true parallel projection (`elevation = 0`, `perspectiveness = 0`, `azimuth = -π / 2`). This allows the raw mathematical output to translate directly into a scaled architectural orthographic elevation. Against the heavy `Z = 0` ground datum line (`final_z = max(z + 10, 0)`), each primary triangular facet is further rationalized into secondary tectonic components: board-formed concrete base walls at pedestrian level, standing-seam titanium-zinc cladding across the opaque shards, and subdivided curtain-wall glazing modules calibrated to human scale.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-maquette.webp" >}}

Because three points in space always define a perfectly flat plane, the entire crystalline envelope is 100% developable from flat sheet stock. As demonstrated in the architectural study model above—fabricated from matte museum board, laser-scored triangular facets, transparent acrylic glazing, and a stepped basswood plinth—the exact topology exported from Julia translates directly into physical assembly without requiring doubly curved panels.

## Interior Tectonics & Spatial Experience

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-interior.webp" >}}

Inside the building, the geometric aggression of the exterior shell opens into a luminous, multi-level cultural atrium. Rather than concealing the structural logic behind false ceilings, the interior expresses the triangulated geometry as a primary tectonic system:

* **Inclined Concrete Cores & Skybridges:** Monumental board-formed concrete piers anchor the exhibition galleries, connected across the void by slender steel pedestrian bridges that allow visitors to experience the crystal from multiple elevations.
* **Diagrid Glazing & Folded Acoustic Soffits:** Where the faceted envelope opens to the sky, inclined steel mullion grids trace the underlying mesh topology, while the opaque upper cones are lined with warm-lit acoustic panels that diffuse daylight deep into the public concourse.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-perimeter-promenade.webp" >}}

Along the building's perimeter, the outward radial projections (`Δr > 0`) become habitable triple-height viewing pockets where cantilevered walkways and floating stairs converge directly behind the faceted glass skin. Here, the multi-nodal steel diagrid junctions—corresponding directly to the perturbed vertices of the Julia mesh—frame oblique vistas back down toward the stepped civic plaza and across the surrounding urban skyline while casting dynamic diagonal sun patterns across the polished concrete decks.

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-interior-mezzanine.webp" >}}

From the upper mezzanine promenade, the spatial choreography between the outer shell and the inner structural frame becomes fully legible. A row of raked board-formed concrete columns supports the cantilevered gallery walkways, while a central stair and escalator spine draws visitors upward through the folded volume—framing dramatic views across the concourse against the fractured glazing.

## Diurnal Tectonic Inversion & Architectural Reading

{{< figure src="/img/projects/chaos-crystal/chaos-crystal-night-plaza.webp" >}}

While the Julia script itself only computes spherical coordinates and random radial offsets (`Δr ∈ [-5, 15]`), architectural translation turns this raw mathematical mesh into a phenomenological instrument of time and light. Under daylight, the curated form reads as an earthbound, gravitational monolith carved by deep shadow; as the sun lowers, the perturbed diagrid nodes frame shifting geometric shadows across the interior concourse. After dusk, this reading inverts—warm interior illumination radiates outward through the triangular apertures while linear grazing lights trace the folded ridge seams, transforming the heavy shell into a glowing civic lantern on the urban plaza.
