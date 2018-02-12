# COPSTANCE
Tips for Houdini procedural texturing workflow in COPs, inspired by Substance Designer.

Work in progress, star or check back every now and then. Suggestions & fixes welcome as: github issues, email: [hi@emill.fi](mailto:hi@emill.fi?Subject=Copstance%20Ideas&Body=Copstance%20would%20be%20much%20better%20if...) or twitter (at or DM): [@\_\_emill](https://twitter.com/__emill). There will be more images and example files.



# general info

need to be mindful of image planes, masking & frame scope in COP nodes

exporting? tiling? resolution?



# node reference

operation (in other software)                | recommended COP node      | notes
-------------                                | -------------             | -------------
levels/fit/curves(all)/Ramp Parameter        | lookup                    | set LUT Source default as Mono Ramp Parameter
curves(RGB individual)                       | color curve               |
blend/mix                                    | composite                 | background as second input, Foreground Weight controls amount
color correction/hue shift/saturation        | hsv                       |
shuffle(nuke)/channel boolean(fusion)        | channelcopy               |
directional warp(sdesigner)/displace(fusion) | deform                    |
clamp                                        | limit                     |
SVG(sdesigner)/roto(nuke)                    | rotoshape                 |



# scene setup


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


