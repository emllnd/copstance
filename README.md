
# COPSTANCE
Tips for Houdini procedural texturing workflow in COPs, inspired by Substance Designer.

This guide is a work in progress, star or check back every now and then. Suggestions & fixes welcome as: github issues, email: [hi@emill.fi](mailto:hi@emill.fi?Subject=Copstance%20Ideas&Body=Copstance%20would%20be%20much%20better%20if...) or twitter (at or DM): [@\_\_emill](https://twitter.com/__emill). There will be more images and example files.


# getting started

![alt text](https://github.com/emillxyz/copstance/raw/master/img/copstance_dl_github.png "Press that DL button!")

Download the contents of this repository, open the example scene, set up your panels (Scene View, Composite View, Parameters, Network View), check the included images & instructions below to see that everything looks correct. Then just delete the example nodes in /img/comp1 (outside the frame) and start building your network. Heightmap is usually a good place to start.

![alt text](https://github.com/emillxyz/copstance/raw/master/img/copstance_example_scene_01.png "Example scene content & recommended panel layout")


# project goal & future improvements

To show how to use Houdini COPs to create procedural textures (PBR, 2D). [[TODO proper paragraph]]

For who?
- for those who already know Substance Designer, UE4/Unity/other node-based material system or compositing software (Nuke, Fusion, After Effects) and are curious about Houdini
- if you're totally new to this stuff, Allegorithmic's Youtube channel [[TODO link]] is a good place to start learning

stuff to cover in the future:
- making assets of these of procedural textures
- usage in engine/renderer/Unity/Unreal/Marmoset
    * recommended to just export textures
    * [[TODO make/link tutorial]]


# node reference

operation (in other software)                | recommended COP node      | notes
-------------                                | -------------             | -------------
levels/fit/curves(all)/Ramp Parameter        | lookup                    | set LUT Source default as Mono Ramp Parameter and both spline point defaults as B-Spline (we're actually misusing a LUT operation here, just a nice way to get a simple but expressive ramp)
curves(RGB individual)                       | colorcurve                |
blend/mix                                    | composite                 | background as second input, Foreground Weight controls amount
color correction/hue shift/saturation        | hsv                       |
shuffle(nuke)/channel boolean(fusion)        | channelcopy               |
directional warp(sdesigner)/displace(fusion) | deform                    |
clamp                                        | limit                     |
SVG(sdesigner)/roto(nuke)                    | rotoshape                 |



# general info and tips & tricks

- panel layout a.k.a Desktop
    * this repository includes the Copstance.desk file that should help you to set up your panel layout in a suitable way for this kind of work [[TODO how to import]]
    * can also be done manually [[TODO link docs/tutorial]]
        - (Scene View, Composite View, Parameters, Network View)
    * hey SideFX, any good way to persist a Desktop across crashes or save it with the scene file?
- image planes
    * C = color, A = alpha
    * recommended to work mostly using C, because that allows for easy node thumbnails & Composite View previews
    * VOPCOP generators & filters: bind export creates a new image plane
- masking (COP node Mask tab)
    * [[TODO]] add image
    * the last input pin on each COP node is the mask input, which allows you to control to which part of the image should the effect be applied, using a mask texture
    * Effect Amount = blend
    * Mask Plane (often: C, Luminance)
- Frame Scope
    * the Frame Scope tab in COP nodes can generally be ignored (it's meant for compositing work: "which frames on the timeline should this node affect?"")
- Scene View lighting modes (metallic unsupported in High Quality Lighting)
- [[TODO]]: Composite View handles
- VOPCOP generators & filters
    * [[TODO]]: how to make a custom noise node
    * bind export creates new image planes
- node defaults
    * [[TODO]]: how to set defaults
    * [[TODO]]: some useful examples

### viewport vs render
- if displacement is set up correctly results are fairly representative
    * [[TODO add comparison images]]

### displacement / height / bump / normals
Three main ways to display the height: displacement, POM (parallax occlusion mapping), bump/normals
- [[TODO]]: link explanations/example images

Recommended to work using displacement as the height map preview. POM (parallax occlusion mapping) would probably be more practical if displacement is too slow, but no such shader is included with Houdini unfortunately (correct me if/when I'm wrong). Bump/normals is better than nothing, but quite inaccurate.

- Setup:
    * in preview object (/obj/grid1)
        - Render tab, Display As: Subdivision Surface / curves
    * Display Options (press D with mouse over viewport)
        - Geometry tab, Level Of Detail = 4 (under Tessellation)
    * in material (/mat/principledshader1, Displacement tab)
        - enable Texture Displacement
        - path "op:/img/comp1/height"

### resolution
- Edit > Compositing Settings or Alt+Shift+I
    * set resolution to 1024x1024 or 2048x2048 or similar
    * this controls the default size on generator COP nodes (can be overridden on a per-node basis e.g. Color COP node, Image tab, Override Size)
- if you're using Houdini Apprentice resolution in Compositing Settings is capped at 720p
    * you can still work in e.g. 2048x2048 by manually setting each generator node to that resolution on the Image tab
    * cannot export larger than 720p from Apprentice

### exporting
- box select all /img/comp1/export_* nodes and click render to export texture set
- export path? naming? (make easy to change export path & master name)
- Indie max export resolution has just been bumped up to 4k (4096x4096), yay!

### [[TODO]]: tiling
- transform COP repeat parameter
- any other tips on making sure the textures tile?

### [[TODO]]: importing geometry & using SOPs
- scatter-based techniques
- SOP Import COP
- cool grid + ray SOP based techniques
    * ray SOP with Direction from: Vector and Ray Direction: [0,1,0] is a nice way to shape a grid with arbitrary geometry
    * what's a quick & efficient way to get that to COPs?
![alt text](https://github.com/emillxyz/copstance/raw/master/img/raysop.png "Grid + Ray SOP")


# scene setup

### (a.k.a how to go from an empty houdini scene to the copstance example file)

#### /obj ####

grid:
- uvunwrap (Spacing 0)
- uvtransform (for tiling, promote to /obj)
- Render tab, Display As: Subdivision Surface / curves
    * otherwise not proper resolution in viewport displacement/tessellation

env light:
- exposure -4
- can use environment map if you want
- (adjust to taste)

point light:
- bluish tint (adjust to taste)


#### /img/comp1 ####

- create nulls for color, roughness, metallic, height, normal
- create gradient node & connect height --> gradient (Output: Normal Map) --> normal
- make sure color (=albedo/diffuse/basecolor) has alpha = 1
    * channelcopy, Target = A, Source = Set to 1
    * if you actually need opacity/transparency in your material you need to set up another output null and point to that in /mat/principledshader1 Textures tab
- Edit > Compositing Settings or Alt+Shift+I
    * set resolution to 1024x1024 or 2048x2048 or similar
    * this controls the default size on generator COP nodes (can be overridden on a per-node basis e.g. Color COP node, Image tab, Override Size)



#### /mat ####

principled shader (not core):

- Surface:
    * Base Color HSV = 0,0,1
    * Roughness = 1

- Textures:
    * enable Base Color, path "op:/img/comp1/color"
    * enable Roughness, path "op:/img/comp1/roughness"
    * enable Metallic, path "op:/img/comp1/metallic"

- Bump & Normals (leave off if using displacement, only use this if can't use displacement for some reason)
    * enable Base, path "op:/img/comp1/normal"

- Displacement
    * enable Texture Displacement, path "op:/img/comp1/height"


#### Display Options (press D with mouse over viewport) ####

- Geometry tab, Level Of Detail = 4 (under Tessellation)


