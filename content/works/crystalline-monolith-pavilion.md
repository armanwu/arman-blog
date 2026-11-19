---
title : "Crystalline Monolith Pavilion"
date : 2026-09-08
draft : false
description : "A rule-based deconstructivist pavilion synthesized in Python using Blender's bmesh library, exploring interlocking opaque volumes and glazed voids."
tools : "Python · Blender"
image : "/img/projects/crystalline-monolith-pavilion/crystalline-monolith-exterior.webp"
layout : "single-no-meta"
---

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-exterior.webp" >}}

## Concept & Spatial Tectonics

Rather than manually sculpting static forms, **Crystalline Monolith Pavilion** investigates a rule-based approach to deconstructivist architecture. Set beside a tranquil reflecting pool within an urban park, the project explores the spatial tension between monolithic opaque volumes and faceted glass voids—testing how fractured geometric wedges can anchor firmly to the ground plane while projecting dramatic cantilevers over the water's edge.

The resulting pavilion forms a luminous civic volume where massive white architectural concrete wedges collide and interlock, pierced by cyan-tinted structural glass fissures that filter daylight through a triangulated steel mullion framework.

<video autoplay loop muted playsinline width="100%">
  <source src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-video.mp4" type="video/mp4">
</video>

## The Computational Pipeline

The entire composition is synthesized via a standalone Python script (`bpy` and `bmesh`) in Blender. As illustrated in the workflow diagram below, the algorithm executes six sequential procedural stages:

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-workflow.webp" >}}

* **Scene & Shader Network Initialization (`clear_all`, `get_materials`):** Resets the 3D environment and programmatically constructs four node-based materials: low-roughness (`0.18`) architectural solid concrete (`Mat_Decon_Solid`), refractive cyan-tinted glass (`Mat_Decon_Glass`, `IOR = 1.52`, `Roughness = 0.02`), dark structural steel frames (`Mat_Mullion`, `Roughness = 0.25`), and the plaza datum surface (`Mat_Plaza`).
* **Stochastic Convex-Hull Synthesis (`create_polyhedral_wedge`):** For each volume, `5` to `7` radial vertex pairs are distributed across randomized polar angles with dual-level `Z` offsets. Calling `bmesh.ops.convex_hull` wraps these point clouds into watertight polyhedral wedges free of self-intersecting faces.
* **Planar Ground Bisecting (`Z = 0.02`):** After applying 3D Euler rotation and translation matrices, vertices extending below grade (`-0.2 × scale_z`) are sliced cleanly along the horizontal datum plane (`Z = 0.02`) via `bmesh.ops.bisect_plane(clear_inner=True)`—automatically establishing flat structural footprints.
* **Solid vs. Glazed Void Assembly (`build_pure_deconstructivism_scene`):** The script spawns `6` to `8` primary solid concrete masses interlocked with `3` to `4` transparent glass volumes. Each glass mesh is duplicated in memory and assigned a procedural `WIREFRAME` modifier (`thickness = 0.12 m`, `use_replace = True`) to generate a matching structural mullion cage automatically.

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-pavilion-blender.webp" >}}

<details class="code-drawer">
<summary>View Python Source Code (crystalline_monolith.py)</summary>

```python
import bpy
import bmesh
import random
import math
from mathutils import Vector, Euler


# Clear existing scene objects and unused geometry data
def clear_all():
    # Remove all scene objects except the camera
    for obj in bpy.context.scene.objects:
        if obj.type != 'CAMERA':
            bpy.data.objects.remove(obj, do_unlink=True)

    # Remove existing mesh, material, and light datablocks
    for b in (bpy.data.meshes, bpy.data.materials, bpy.data.lights):
        for item in b:
            b.remove(item)


# Configure the ambient world environment
def setup_scene_ambient():
    # Create a world environment if one does not already exist
    world = bpy.context.scene.world or bpy.data.worlds.new("World")
    bpy.context.scene.world = world
    world.use_nodes = True

    # Set the background color and ambient illumination strength
    bg = world.node_tree.nodes.get("Background")
    if bg:
        bg.inputs['Color'].default_value = (0.05, 0.06, 0.08, 1.0)
        bg.inputs['Strength'].default_value = 1.0


# Create and configure the materials used in the architectural composition
def get_materials():

    # Define the primary solid architectural material
    mat_solid = bpy.data.materials.new("Mat_Decon_Solid")
    mat_solid.use_nodes = True
    bsdf_s = mat_solid.node_tree.nodes.get("Principled BSDF")

    if bsdf_s:
        bsdf_s.inputs['Base Color'].default_value = (0.22, 0.24, 0.28, 1.0)
        bsdf_s.inputs['Roughness'].default_value = 0.18

        # Configure specular reflection based on Blender shader version
        if 'Specular IOR Level' in bsdf_s.inputs:
            bsdf_s.inputs['Specular IOR Level'].default_value = 0.7
        elif 'Specular' in bsdf_s.inputs:
            bsdf_s.inputs['Specular'].default_value = 0.7

    # Create a transparent glass material using the Glass BSDF shader
    mat_glass = bpy.data.materials.new("Mat_Decon_Glass")
    mat_glass.use_nodes = True
    nodes_g = mat_glass.node_tree.nodes
    links_g = mat_glass.node_tree.links

    # Remove default shader nodes before creating the glass shader network
    nodes_g.clear()

    # Create the material output and glass shader nodes
    out_g = nodes_g.new('ShaderNodeOutputMaterial')
    bsdf_g = nodes_g.new('ShaderNodeBsdfGlass')

    # Define glass color, surface roughness, and refractive index
    bsdf_g.inputs['Color'].default_value = (0.6, 0.85, 0.98, 1.0)
    bsdf_g.inputs['Roughness'].default_value = 0.02
    bsdf_g.inputs['IOR'].default_value = 1.52

    # Connect the glass shader to the material surface output
    links_g.new(bsdf_g.outputs['BSDF'], out_g.inputs['Surface'])

    # Define the dark structural material for the glass mullion system
    mat_mullion = bpy.data.materials.new("Mat_Mullion")
    mat_mullion.use_nodes = True
    bsdf_m = mat_mullion.node_tree.nodes.get("Principled BSDF")

    if bsdf_m:
        bsdf_m.inputs['Base Color'].default_value = (0.05, 0.05, 0.06, 1.0)
        bsdf_m.inputs['Roughness'].default_value = 0.25

    # Define the material for the surrounding ground plane
    mat_ground = bpy.data.materials.new("Mat_Plaza")
    mat_ground.use_nodes = True
    bsdf_gr = mat_ground.node_tree.nodes.get("Principled BSDF")

    if bsdf_gr:
        bsdf_gr.inputs['Base Color'].default_value = (0.06, 0.07, 0.09, 1.0)
        bsdf_gr.inputs['Roughness'].default_value = 0.5

    # Return all four materials for use in the architectural scene
    return mat_solid, mat_glass, mat_mullion, mat_ground


# Generate an irregular polyhedral wedge through procedural geometry
def create_polyhedral_wedge(center, scale_vec, rot_euler):

    # Initialize an empty BMesh structure for direct mesh manipulation
    bm = bmesh.new()

    # Randomly determine the number of radial vertex pairs
    num_pts = random.randint(5, 7)

    # Define the initial radial dimension of the polyhedral geometry
    base_r = scale_vec.x

    # Generate randomly distributed angular positions around the center
    angles = sorted([random.uniform(0, 2 * math.pi) for _ in range(num_pts)])

    # Define a displaced upper region to create an irregular wedge profile
    top_offset = Vector((
        random.uniform(-scale_vec.x * 0.5, scale_vec.x * 0.5),
        random.uniform(-scale_vec.y * 0.5, scale_vec.y * 0.5),
        scale_vec.z
    ))

    # Generate alternating lower and upper vertices
    for a in angles:

        # Introduce radial variation to avoid regular geometric patterns
        r = base_r * random.uniform(0.7, 1.3)

        # Calculate the lower vertex coordinates
        bx = math.cos(a) * r
        by = math.sin(a) * r * (scale_vec.y / scale_vec.x)

        # Create the lower vertex of the polyhedral structure
        bm.verts.new(Vector((bx, by, -scale_vec.z * 0.2)))

        # Calculate the upper vertex with additional positional variation
        tx = bx * random.uniform(0.4, 1.2) + top_offset.x
        ty = by * random.uniform(0.4, 1.2) + top_offset.y
        tz = top_offset.z + random.uniform(-scale_vec.z * 0.15, scale_vec.z * 0.15)

        # Create the upper vertex of the polyhedral structure
        bm.verts.new(Vector((tx, ty, tz)))

    # Construct a convex hull from the generated vertices
    # This creates an irregular solid volume without predefined geometry
    bmesh.ops.convex_hull(bm, input=bm.verts)

    # Convert the specified Euler rotation into a transformation matrix
    rmat = rot_euler.to_matrix().to_4x4()

    # Apply the object's spatial translation
    rmat.translation = center

    # Transform the entire polyhedral geometry using rotation and translation
    bmesh.ops.transform(bm, matrix=rmat, verts=bm.verts)

    # Apply a horizontal planar bisect operation at Z = 0.02
    # This removes geometry on one side of the cutting plane
    bmesh.ops.bisect_plane(
        bm,
        geom=bm.verts[:] + bm.edges[:] + bm.faces[:],
        plane_co=(0, 0, 0.02),
        plane_no=(0, 0, 1),
        clear_inner=True
    )

    # Update vertex indexing and recalculate surface normals
    bm.verts.ensure_lookup_table()
    bm.normal_update()

    # Return the completed procedural geometry
    return bm


# Assemble the complete deconstructivist architectural composition
def build_pure_deconstructivism_scene():

    # Reset the scene and initialize its environment
    clear_all()
    setup_scene_ambient()

    # Retrieve all architectural materials
    mat_solid, mat_glass, mat_mullion, mat_ground = get_materials()

    # Generate a randomized collection of solid architectural volumes
    num_solids = random.randint(6, 8)

    for i in range(num_solids):

        # Randomize the spatial position of each solid volume
        center = Vector((
            random.uniform(-9.0, 9.0),
            random.uniform(-9.0, 9.0),
            random.uniform(4.0, 11.0)
        ))

        # Randomize the proportions of each architectural volume
        scale_vec = Vector((
            random.uniform(14.0, 22.0),
            random.uniform(7.0, 12.0),
            random.uniform(6.0, 10.0)
        ))

        # Apply random three-dimensional rotations
        rot_euler = Euler((
            math.radians(random.uniform(-22, 22)),
            math.radians(random.uniform(-22, 22)),
            math.radians(random.uniform(0, 360))
        ), 'XYZ')

        # Generate the irregular polyhedral geometry
        bm_s = create_polyhedral_wedge(center, scale_vec, rot_euler)

        # Convert BMesh geometry into a Blender mesh datablock
        mesh_s = bpy.data.meshes.new(f"Solid_Mesh_{i+1}")
        bm_s.to_mesh(mesh_s)
        bm_s.free()

        # Create the architectural solid object and link it to the scene
        obj_s = bpy.data.objects.new(f"Solid_Block_{i+1}", mesh_s)
        bpy.context.collection.objects.link(obj_s)

        # Assign the primary solid material
        obj_s.data.materials.append(mat_solid)

    # Generate a separate collection of transparent glass volumes
    num_glass_volumes = random.randint(3, 4)

    for j in range(num_glass_volumes):

        # Randomize the position of each glass volume
        g_center = Vector((
            random.uniform(-5.0, 6.0),
            random.uniform(-5.0, 6.0),
            random.uniform(4.0, 9.0)
        ))

        # Define randomized dimensions for the glass structures
        g_scale = Vector((
            random.uniform(10.0, 16.0),
            random.uniform(8.0, 14.0),
            random.uniform(8.0, 15.0)
        ))

        # Introduce independent rotational variations
        g_rot = Euler((
            math.radians(random.uniform(-25, 25)),
            math.radians(random.uniform(-25, 25)),
            math.radians(random.uniform(0, 360))
        ), 'XYZ')

        # Generate the procedural glass geometry
        bm_g = create_polyhedral_wedge(g_center, g_scale, g_rot)

        # Convert the generated geometry into a Blender mesh
        mesh_g = bpy.data.meshes.new(f"Glass_Mesh_{j+1}")
        bm_g.to_mesh(mesh_g)
        bm_g.free()

        # Create the transparent architectural glass object
        obj_glass = bpy.data.objects.new(f"Glass_Structure_{j+1}", mesh_g)
        bpy.context.collection.objects.link(obj_glass)

        # Assign the transparent glass material
        obj_glass.data.materials.append(mat_glass)

        # Duplicate the glass object to generate its structural frame
        obj_frame = obj_glass.copy()
        obj_frame.data = obj_glass.data.copy()
        obj_frame.name = f"Glass_Mullions_{j+1}"
        bpy.context.collection.objects.link(obj_frame)

        # Replace the glass material with the dark mullion material
        obj_frame.data.materials.clear()
        obj_frame.data.materials.append(mat_mullion)

        # Apply a Wireframe modifier to generate the structural framework
        mod_wf = obj_frame.modifiers.new("Mullion_Wireframe", 'WIREFRAME')

        # Define the thickness of the mullion elements
        mod_wf.thickness = 0.12

        # Replace the original surface with the wireframe geometry
        mod_wf.use_replace = True

    # Create the ground plane using BMesh
    bm_p = bmesh.new()

    # Generate a large planar grid to represent the surrounding plaza
    bmesh.ops.create_grid(bm_p, x_segments=1, y_segments=1, size=300.0)

    # Convert the ground geometry into a Blender mesh
    mesh_p = bpy.data.meshes.new("Plaza_Mesh")
    bm_p.to_mesh(mesh_p)
    bm_p.free()

    # Create the plaza object and link it to the scene
    plaza = bpy.data.objects.new("Plaza_Ground", mesh_p)
    bpy.context.collection.objects.link(plaza)

    # Assign the ground material
    plaza.data.materials.append(mat_ground)

    # Deselect all objects to leave the scene in a neutral state
    bpy.ops.object.select_all(action='DESELECT')


# Execute the procedural architectural generation
build_pure_deconstructivism_scene()
```

</details>

## Tectonic Rationalization & Physical Study Model

Because `bmesh.ops.convex_hull` constructs each volume from flat polygonal planes and `bmesh.ops.bisect_plane` trims the base cleanly at `Z = 0.02`, the resulting geometry avoids non-developable double curvature.

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-maquette.webp" >}}

As verified in the physical architectural study model above, the procedural polyhedra translate directly into crisp, planar white concrete shells and triangulated cyan-acrylic glazing cages mounted over a recessed reflecting pool plinth—allowing complex spatial collisions to be fabricated from flat formwork and linear steel profiles.

## Interior Spatial Experience & Nocturnal Presence

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-interior.webp" >}}

Inside the pavilion, the collision between opaque and transparent volumes creates a dramatic double-height exhibition hall:

* **Inclined Concrete Shear Walls & Mezzanine Promenade:** Massive tilted concrete planes—washed by warm recessed cove lighting along their folded ceiling joints—guide visitors up a central stair to a cantilevered glass-balustrade mezzanine.
* **Crystalline Glazing & Pool Vistas:** On the park-facing flank, the faceted cyan-blue glass volume opens the lounge and gallery floor to panoramic views of the reflecting water basin and city skyline, while its triangulated mullion cage projects sharp geometric shadow patterns across the polished concrete floor.

{{< figure src="/img/projects/crystalline-monolith-pavilion/crystalline-monolith-exterior-night.webp" >}}

At twilight, architectural grazing uplights accentuate the sharp creases and overhanging soffits of the white monolithic wedges, while warm interior illumination radiates outward through the glazed central fissure and mirrors across the calm water plane.