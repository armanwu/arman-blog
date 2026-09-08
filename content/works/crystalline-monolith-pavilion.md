---
title : "Crystalline Monolith Pavilion"
date : 2026-09-08
draft : false
image : "/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-exterior.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-exterior.webp" >}}

## Concept & Spatial Tectonics

Rather than manually sculpting static forms, Crystalline Monolith Pavilion investigates a rule-based approach to deconstructivist architecture. The project explores the spatial tension between opaque volumes and faceted glass voids—testing how fractured geometric shards can anchor firmly to the earth while maintaining dynamic, cantilevered silhouettes.

The resulting pavilion forms a public volume where massive concrete wedges crash and interlock, punctuated by glazed apertures that filter daylight through steel mullion frameworks.

<video autoplay loop muted playsinline width="100%">
  <source src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-video.mp4" type="video/mp4">
</video>

## The Computational Pipeline

The entire form is synthesized via a standalone Python script using Blender's bmesh library. The generative workflow follows three distinct spatial operations:
* **Stochastic Hull Generation:** Vertices are generated across randomized radial angles and dual-level Z offsets. A convex hull calculation wraps these points into a clean polyhedral wedge without self-intersecting faces.
* **Grounding via Planar Bisecting:** To prevent floating cantilevers and establish structural grounding, vertices are anchored into negative Z-space (-0.2 × scale_z) and cleanly cut at the base plane (Z = 0.02) using `bisect_plane`.
* **Solid vs. Glazed Void Interlocking:** The program instantiates two distinct spatial layers: opaque monolithic solids (`Mat_Decon_Solid`) and transparent volumes (`Mat_Decon_Glass`). The glass masses are duplicated and modified with a wireframe modifier (thickness: 0.12 m) to construct structural mullions automatically.

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-blender.webp" >}}

## Core Code Implementation

```python
def create_polyhedral_wedge(center, scale_vec, rot_euler):
    bm = bmesh.new()
    num_pts = random.randint(5, 7)
    base_r = scale_vec.x

    angles = sorted([random.uniform(0, 2 * math.pi) for _ in range(num_pts)])
    top_offset = Vector((
        random.uniform(-scale_vec.x * 0.5, scale_vec.x * 0.5),
        random.uniform(-scale_vec.y * 0.5, scale_vec.y * 0.5),
        scale_vec.z
    ))

    for a in angles:
        r = base_r * random.uniform(0.7, 1.3)
        bx = math.cos(a) * r
        by = math.sin(a) * r * (scale_vec.y / scale_vec.x)
        bm.verts.new(Vector((bx, by, -scale_vec.z * 0.2)))

        tx = bx * random.uniform(0.4, 1.2) + top_offset.x
        ty = by * random.uniform(0.4, 1.2) + top_offset.y
        tz = top_offset.z + random.uniform(-scale_vec.z * 0.15, scale_vec.z * 0.15)
        bm.verts.new(Vector((tx, ty, tz)))

    bmesh.ops.convex_hull(bm, input=bm.verts)

    rmat = rot_euler.to_matrix().to_4x4()
    rmat.translation = center
    bmesh.ops.transform(bm, matrix=rmat, verts=bm.verts)

    bmesh.ops.bisect_plane(
        bm,
        geom=bm.verts[:] + bm.edges[:] + bm.faces[:],
        plane_co=(0, 0, 0.02),
        plane_no=(0, 0, 1),
        clear_inner=True
    )

    bm.verts.ensure_lookup_table()
    bm.normal_update()
    return bm
```

## Materiality & Architectural Balance

The script balances monolithic mass with light permeability by contrasting three dedicated material trees:

* **Deconstructive Solid:** A low-roughness (0.18) charcoal concrete block with tuned specular response.
* **Translucent Void:** A glass BSDF material (IOR 1.52, roughness 0.02) tinted with a cool cyan-blue hue.
* **Mullion Framework:** Dark structural frames (0.05 base color) wrapping the glazing modules to provide scale, rhythm, and structural readability.

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-interior.webp" >}}