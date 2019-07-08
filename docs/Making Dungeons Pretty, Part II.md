# Making Dungeons Pretty, Part II
This second feature on building pretty maps is just a collection of miscellaneous nuggets meant to expand the functionality of the type-to-view conversion rules. 

## Here There Be Dragons
After you've implemented the system, you may realize that there is a small problem that was initially overlooked - the edges of the map. Depending on how you write your mapping functions, grabbing a tile off the edge of the map may return a default value, a nil value, or just throw an exception. Whatever the case, you may notice that none of your border tiles are matching any rules (or the right rules) and producing the wrong tile values.

There's two simple solutions here. The first is to just make your maps a little bit larger than you can see, so that the border tiles aren't visible - though this is wasteful. The other solution is that off the map cells return a nil value, and then to include NIL as a possible rule in the same way that ANY is a rule. NIL will only match tiles off the map, allowing you to create rules specifically for border tiles.

Problem with that is that it'll require a lot of rules for what are ultimately limited special cases. A more practical solution is to sort of combine the two. Rather than having two extra rows and two extra columns, just mirror the nearest like row or column. For instance, when querying the type at (-1, 14), just return the type at (0, 14). (-1, -1) becomes (0, 0). 99% of the time, you can create perfect border tiles this way.

I still recommend returning a NIL value for off the map because it can be useful for other purposes. The mirroring can be done at the ruleset level. 

## These Are Not the Tiles You are Looking For
Some tile types should be able to pose as other tile types for the purposes of creating the view map. Otherwise, you'd have to make exceptions and write many rules every time you introduce a new, slightly different tile type. For instance, a cracked floor should be treated as a regular floor by neighboring tiles.

This is easy enough to do. Rather than checking the type against the rule, ask the RLTileType object whether or not it matches the rule. That way, a cracked floor can return true for both CRACKED_FLOOR and FLOOR tile types. 

## For Instance, Tile Types
Previously, I said that it was enough for the typeMap to simply point to a single instance of each type. You'd only need one FLOOR RLTileType object that is shared by all floor spaces. However, there are situations where you may not want this.

A cracked floor may contain an instance variable that keeps track of how often it has been stepped on. After a certain number of times, it destroys itself and replaces itself with a pit. You don't want to share the same cracked floor object because the step count needs to be unique for each cell and because the cell needs to know it's own position so that it can replace itself.

The solution is to grab the objects via and instance() function which will either return the same singleton instance, or a brand new instance depending on the function of the type. If you aren't in an environment with garbage collecting or reference counting, be sure to tell the difference before you go deleting objects. 

## Implications of Random Variation
The current approach produces a single tile for each type configuration, but this can lead to large bland areas. You can create some random variation by subclassing the RLTypeViewRule and keeping a list of multiple tile graphic indexes that you choose from randomly (perhaps use the first tile 50% of the time and split the remaining 50% between the remaining values).

Use this to add random flowers to fields or fill your dungeon floors with random debris. 

## Blob, Fence, Rug
There are a few common patterns that will repeatedly crop up that you can save some time and energy by standardizing the rules. There are three patterns that I've noticed, and all three of them rely on each type being insular. That is, the only types it cares about are itself and NOT itself. A ceiling doesn't care whether it is next to a lava pit or a floor, only that it isn't next to another ceiling.

The idea is that each pattern can build a specific set of rules if you pass it the needed tiles in a specific order. So if the Rug pattern requires 16 tiles, you can declare a ruleset a type of Rug and only supply those 16 integers that correspond to corners and whatnot. You can then add additional rules to override them if you choose (for instance, to introduce random variations). 

![The Rug](https://phoebe-g.github.io/mapmaker-musings/images/rug-325%C3%97155.jpg)

The Rug is a pattern which can be used to create any rectangular pattern down to a 1x1 rect using 16 different tiles. Due to the way the tiles are patterned, only rectangles can be properly modeled. 

![The Fence](https://phoebe-g.github.io/mapmaker-musings/images/fence-301%C3%97167.png)

Think of the fence as each tile being a fence post that is connected to one or more of its neighboring tiles. It also uses 16 tiles, like the Rug, but it can make odd shapes because it ignores inner corners. The main drawback of the Fence is that when you get a bunch of center tiles together, it doesn't look very natural (see illustration). 

![The Blob](https://phoebe-g.github.io/mapmaker-musings/images/blob-370%C3%97136.png)

The Blob is the most complex pattern there is, requiring a whopping 47 tiles to fully implement. However, it is the most versatile, capable of producing any connected shape you can imagine. The inner corners are what makes up the bulk of the differences.

As you can imagine, with 47 tiles, a ruleset which creates a blog will come with 47 rules (at least) - some of which need to be in a particular order to work properly. By standardizing the blob pattern, you can create these rules instantly, simply by supplying the indexes of the 47 tiles. And like I said above, you can then add additional override rules to add random variation. 

----

Copyright 2009 Sean Howard. All rights reserved.

Rescued from internet archive and google image cache in 2019.
