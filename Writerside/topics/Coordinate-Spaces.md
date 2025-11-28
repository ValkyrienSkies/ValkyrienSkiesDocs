# Coordinate Spaces

Valkyrien Skies has three **coordinate spaces**, also known as **reference frames**, **coordinate systems**, or **vector spaces**.

## World Space

> **Alternative Names**: Scene Space (PhysX), Global Space (PhysX, Unity)

**World space** is the space where the player exists—the coordinate system inherent to Minecraft. In vanilla Minecraft, blocks always have integer coordinates in world space. However, Valkyrien Skies allows blocks to move "off grid," so blocks on ships can have non-integer coordinates in world space.


## Model Space (Ship Space)

> **Alternative Names**: Ship Space (VS), Actor Space (PhysX), Local Space (Unity, Unreal Engine)

**Model space** is the space used by collision shape of a body. In model space, the body always has zero rotation.

For ships specifically, model space is also called **ship space**. In ship space, the center of mass is a [shipyard](The-Shipyard.md) coordinate—for example, (-2 million, 0, -2 million). Blocks on a ship always have integer coordinates in that ship's ship space.

For bodies with other collision shapes, refer to the documentation for that shape.

## Body Space

> **Alternative Names**: Mass Space, CMassLocal (PhysX), Local Mass Space (Unreal Engine), Body Frame, Center Of Mass Frame, Body Fixed Frame (Real Physics)

**Body space** is the space relative to the center of mass. In body space, the body always has zero rotation and the center of mass is always (0, 0, 0). 
