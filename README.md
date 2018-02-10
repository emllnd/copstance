# copstance
Tips for Houdini procedural texturing workflow in COPs, inspired by Substance Designer





SUBSTANCE/HOUDINI COPs
======================


==============
node reference
==============


operation                           recommended             notes
---------                           -----------             -----

levels/fit/curves(all)              lookup                  set LUT Source default as Mono Ramp Parameter

curves(RGB individual)              ??? color curve ???

blend                               composite               background as second input,
                                                            Foreground Weight controls amount

color correction                    hsv

shuffle(nuke)/                      channelcopy
channel boolean(fusion)


displace/directional warp —-> deform?

clamp —-> limit


===========
scene setup
===========


>>>>>>>> /obj

grid:
    - uvunwrap (Spacing 0)
    - uvtransform (for tiling, promote to /obj)
    - Render tab, Display As: Subdivision Surface / curves
        * otherwise not proper resolution in viewport displacement/tessellation

env light
    - exposure -4
    - can use environment map if you want
    - (adjust to taste)

point light
    - bluish tint (adjust to taste)




>>>>>>>> Display Options (press D with mouse over viewport)

- Geometry tab, Level Of Detail = 4 (under Tessellation)




>>>>>>>> /img/comp1

- create nulls for color, roughness, metallic, height, normal
- create gradient node & connect height --> gradient (Output: Normal Map) --> normal
- make sure color (=albedo/diffuse/basecolor) has alpha = 1
    * channelcopy, Target = A, Source = Set to 1
    * if you actually need opacity/transparency in your material you need to set up another output null and point to that in /mat/principledshader1 Textures tab



>>>>>>>> /mat

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
    * enable Base Color, path "op:/img/comp1/color"




============
general info
============

need to be mindful of image planes, masking & frame scope in COP nodes

exporting?
tiling? (Repeat parm in transform nodes)
resolution?
- where to adjust?
- limitations in Indie/Apprentice

