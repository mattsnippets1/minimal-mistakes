---
title: "Expanding procedural terrain generation features in Balance"
date: 2021-04-29 10:00:00 +0200
tags: TPS procedural-generation chunks
---
I wanted to write a post following the recent overhaul of Balance demo. A short summary of some of the changes made to Balance's procedural generation logic.
<!--more-->

So let's revisit how it transformed from this:

![Balance old prototype screenshot]({{site.url}}/assets/images/balance/balance-old-prototype.jpg){: .align-center}

To this: 
![Balance alpha demo screenshot]({{site.url}}/assets/images/balance/balance-5-tn.jpg){: .align-center}

As you can see the old prototype was a rather simplistic experiment. During the first iterations the game's main goal was cleaning up a sort of corruption in the forest and restoring it to its former state (hence the title Balance). Some grey tiles representing the corruption can be seen in the first screenshot. So I spent quite a bit of time creating a 3D tile based world where tiles could have certain states e.g. corrupted, clean, protected as well as implementing a planting mechanic.

The tilemap implementation was really rudimentary: only a flat mesh which basically used a stretched texture having the same pixel dimensions as the actual map chunk. A splat map shader was utilized so based on the color of the pixel the map "tiles" would also change color e.g. dark grey for corrupted, green for clean and brown for planted areas.

Of course this soon proved to be really limiting as it would become problematic to support different biomes, lots of different textures etc. I first changed it to a temporary solution using a sprite atlas and assigning the texture to certain areas of the chunk mesh with **Texture.SetPixels()**. This was a low performance way of doing it and I eventually replaced it by using actual UVs and mapping the tile textures to triangles with UV coordinates.

![Balance old prototype screenshot]({{site.url}}/assets/images/balance/old-balance-world.jpg){: .align-center}
*A zoomed out screenshot of the first prototype world*
{: .text-center}

Apart from the texture work the main task was to allow the terrain supporting a heightmap making it much more engaging and natural looking. As I've mentioned in the Balance introduction post [Sebastian Lague](https://www.youtube.com/channel/UCmtyQOKKmrMVaKuRXz02jbQ)'s procedural generation tutorials were a huge help in all of this and I recommend watching them (and all the various cool stuff he does).

Introducing a Y coordinate to the generated mesh wasn't the big deal, but aligning it with the neighbor chunks as well as generating proper normals is not a trivial challenge. I went with flat shading for the terrain because I knew the assets I would use for the plants, enemies and the player model would be low poly so a smooth shaded terrain seemed off to me and I preferred the "retro" look of pointy vertices.

Having a vertical dimension in the game of course created lots of new problems: handling the camera so it wouldn't clip through the terrain or stay behind a hill hiding the player, generating proper position and orientation of tree trunks, plants and props, better player controls to make walking over terrain more realistic, restricting the camera so it wouldn't show areas which the player should not see (such as the forest canopy), just to name a few. I will likely write additional posts detailing some of these problems and the solutions I came up with.

The map generator I had been using already returned height values but those were used only in determining whether a tile is corrupted or clear. Now I'm using the actual values which are approximately within the range of 0-1.0 and I multiply that with an arbitrary number to scale those heights to realistic dimensions. The **Tile** objects used within map chunks store all the data necessary for rendering the 3D mesh. I found that while the first time creation of a chunk is more costly because of caching all the values such as the height, elevation (height mapped to a curve of "height zones" determining the actual tile texture), surface normal etc. during the creation of the chunk it still saves a lot of computation time later on.

In the current version there are two layers to the dynamic chunk handling:

A **ChunkManager** class maintains a collection of all the chunks and based on the player position starts the activation/creation or deactivation of chunks. The nine chunks around the player (including the chunk the player is standing on) are always kept alive. When the player approaches the edge of a chunk leaving it via a side or corner the relevant neighbors are also activated. Deactivation on the other hand is only called if a chunk is further away from the player i.e. it has a Chebyshev distance of at least 3 from the player chunk. This is included to avoid switching chunks on/off multiple times when the player goes back and forth on a border line between chunks.
By checking the dictionary of current chunks it is decided whether a new chunk needs instantiation or is was created before and only reactivation is required.

The **Chunk** class itself handles initialization taking different steps depending on whether it's a first time creation or reactivation of the chunk. Creation steps with a bigger workload are organized into async Tasks to avoid stalling the game and causing frame drops. These tasks are: generation of terrain data and tree positions, creation of an array containing all the Tile objects of the chunk, chunk mesh generation, texture generation. Props like rocks, plants etc. and powerups are pooled but also their activation is throttled to a certain amount/update loop otherwise this would be another cause of FPS drops. When first time chunk setup is finished or a chunk is deactivated all the chunk state is serialized and stored via the ZeroFormatter plugin. This ensures quick restoration of the state when a chunk is reactivated, also it can easily be written into a save file to be loaded next time. The chunk mesh itself is only deactivated when far away from the player but all props, items, enemies are despawned and returned to their pools.

Finally, a screenshot of the rather funny (and temporary) solution I came up with to prevent the player from leaving the map or falling off: the generated height values are tweaked based on the distance from the map center to create a big crater with unclimbable edge walls.

![Balance crater map screenshot]({{site.url}}/assets/images/balance/balance-crater-map.jpg){: .align-center}
