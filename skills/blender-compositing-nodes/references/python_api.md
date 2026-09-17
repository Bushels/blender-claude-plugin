# Blender Compositor Nodes - Python API Reference

Verified headless on **Blender 5.2.1**. Every code block below runs, top to bottom, as one script.

## Look Up Names Live — Do Not Guess

The compositor API changed heavily in 5.0: many node types were removed or merged, and most node settings
moved from Python properties to node **input sockets**. A static catalog of type strings and properties goes
stale on every release, so this file no longer carries one.

Before using any node type, property or socket name, confirm it against the running Blender:

- Blender MCP: `describe_node_type` (inputs, outputs and properties of a node type) and `bpy_api_lookup`.
- Headless probe: create the node and print what it actually has:

```python
import bpy

probe = bpy.data.node_groups.new("Probe", "CompositorNodeTree")
node = probe.nodes.new("CompositorNodeGlare")  # the type you want to use
print("inputs: ", [s.name for s in node.inputs])
print("outputs:", [s.name for s in node.outputs])
print("props:  ", [p.identifier for p in node.bl_rna.properties if not p.is_readonly])
bpy.data.node_groups.remove(probe)
```

Known 5.x changes (so an old tutorial does not mislead you):

| Old (pre-5.0) | 5.x |
|---|---|
| `scene.use_nodes = True` / `scene.node_tree` | `scene.compositing_node_group` holding a `CompositorNodeTree` |
| `CompositorNodeComposite` | group `Image` output socket + `NodeGroupOutput` |
| `CompositorNodeSplitViewer`, `CompositorNodeMixRGB`, `CompositorNodeMath` | removed — probe for current equivalents |
| node properties such as `blur.filter_type`, `color_balance.correction_method`, `keying.clip_black` | mostly **input sockets** now, e.g. `node.inputs['Type'].default_value` |
| `view_layer.use_pass_denoising_normal/albedo` | `view_layer.cycles.denoising_store_passes = True` (Cycles) |
| `file_output.base_path` | removed — probe `CompositorNodeOutputFile` before scripting file output |

## Setting Up the Compositor

```python
import bpy

scene = bpy.context.scene

# Create the compositor node group and assign it to the scene
# (Blender 5.0+: scene.node_tree was removed)
tree = scene.compositing_node_group
if tree is None:
    tree = bpy.data.node_groups.new("Compositor", "CompositorNodeTree")
    scene.compositing_node_group = tree
nodes = tree.nodes
links = tree.links

# Clear defaults
nodes.clear()
```

## Final Output

```python
# 5.0+: the group needs an Image output socket, then link into a Group Output node.
tree.interface.new_socket(name="Image", in_out='OUTPUT', socket_type='NodeSocketColor')
group_out = nodes.new('NodeGroupOutput')

# Viewer (backdrop preview)
viewer = nodes.new('CompositorNodeViewer')
```

## Settings Live on Input Sockets

```python
hue_sat = nodes.new('CompositorNodeHueSat')
hue_sat.inputs['Saturation'].default_value = 1.1

color_bal = nodes.new('CompositorNodeColorBalance')
# Menu inputs take the display string: 'Lift/Gamma/Gain', 'Offset/Power/Slope (ASC-CDL)', 'White Point'
color_bal.inputs['Type'].default_value = 'Lift/Gamma/Gain'
```

## Linking, Positioning and Frames

```python
rl = nodes.new('CompositorNodeRLayers')

# By socket name (preferred) or by index
links.new(rl.outputs['Image'], hue_sat.inputs['Image'])
links.new(hue_sat.outputs['Image'], group_out.inputs[0])

rl.location = (-600, 0)
hue_sat.location = (0, 0)
group_out.location = (300, 0)

frame = nodes.new('NodeFrame')
frame.label = "Grade"
hue_sat.parent = frame
```

## Complete Pipeline: Denoise + Colour Grade

Run on its own; it rebuilds the tree from scratch.

```python
import bpy

scene = bpy.context.scene
tree = scene.compositing_node_group or bpy.data.node_groups.new("Compositor", "CompositorNodeTree")
scene.compositing_node_group = tree
nodes = tree.nodes
links = tree.links
nodes.clear()
tree.interface.clear()

# Denoising data passes (Cycles only)
scene.render.engine = 'CYCLES'
bpy.context.view_layer.cycles.denoising_store_passes = True  # adds Denoising Normal/Albedo outputs

rl = nodes.new('CompositorNodeRLayers')
rl.location = (-600, 0)

denoise = nodes.new('CompositorNodeDenoise')
denoise.location = (-300, 0)

color_bal = nodes.new('CompositorNodeColorBalance')
color_bal.location = (0, 0)
color_bal.inputs['Type'].default_value = 'Lift/Gamma/Gain'

hue_sat = nodes.new('CompositorNodeHueSat')
hue_sat.location = (300, 0)
hue_sat.inputs['Saturation'].default_value = 1.1

tree.interface.new_socket(name="Image", in_out='OUTPUT', socket_type='NodeSocketColor')
comp = nodes.new('NodeGroupOutput')
comp.location = (600, 0)

viewer = nodes.new('CompositorNodeViewer')
viewer.location = (600, -200)

links.new(rl.outputs['Image'], denoise.inputs['Image'])
links.new(rl.outputs['Denoising Normal'], denoise.inputs['Normal'])
links.new(rl.outputs['Denoising Albedo'], denoise.inputs['Albedo'])
links.new(denoise.outputs['Image'], color_bal.inputs['Image'])
links.new(color_bal.outputs['Image'], hue_sat.inputs['Image'])
links.new(hue_sat.outputs['Image'], comp.inputs['Image'])
links.new(hue_sat.outputs['Image'], viewer.inputs['Image'])
```
