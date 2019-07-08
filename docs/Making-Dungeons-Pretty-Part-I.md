# Making Dungeons Pretty Part I
If your goals are more pragmatic than lofty, digging a random dungeon is relatively simple and straightforward. You can get something passable with only a few lines of code. However, if you want to take that map and use graphics to display it, just replacing the ASCII with equal tile graphics ends up creating a flat and boring map that likely looked better in ASCII, truth be told. 

| ![Two pictures of the same thing](https://phoebe-g.github.io/mapmaker-musings/images/map01-ascii.png) | ![Two pictures of the same thing](https://phoebe-g.github.io/mapmaker-musings/images/map01-tiles.png) |

Below is a relatively simple technique to go from illustration A to illustration B procedurally - as in, it uses a series of rules based on neighboring cells to determine which tile graphic should go there. With it, all your graphics can feature smooth transitions, shadows, and pretty pictures. Incidentally, this technique has been tried and proven using the tiles from the Free Pixel Project dungeon tiles. 

Though this "system" is mainly aimed at 2D map developers, which a strong focus on Roguelikes, it can be modified for other uses. 

## Model-View-Controller and You!
At the heart of this concept is the old computer science standby, the Model-View-Controller. If you have ever programmed for a computer more modern than DOS, chances are that you've met the old MVC a dozen times over. It is literally the foundation of modern GUIs. 

If you aren't familiar with MVC, here's a quick overview. Basically it describes the decoupling of data (the model) with how that data is viewed (the view), using an intermediary (the controller) to convert one to the other. The classic example is a list of numbers which can be viewed as a spreadsheet and/or a pie graph. Same data, different views. With GUIs, most controls are considered views and it is usually up to you to write the controller to link data of your format into something the controls can see and manipulate. 

This system is essentially a controller, but to make sense of it, I have to describe the view and the model. 

The view is simple enough. It is just a giant two-dimensional array of integers which are indexes into the array of tile images. It is the most classic of views and can the entire thing can be drawn onscreen with a few lines of code. 


```js
  for( int row = 0; row < maxRows; row++ )
  {
    for( int col = 0; col < maxCols; col++ )
    {
      drawTile( col * tileWidth,
                row * tileHeight,
                tilemap[ row * maxCols + col ] );	
    }
  }
```

The model is equally simple. It is just a giant two-dimensional array of map cells, equal in size to the view map, where each map cell represents something like floor, wall, lava, and so on. 

Though you CAN represent the model as an ASCII map, I recommend against it. It's too hard to maintain and you have to constantly look up what the data is for a '#' tile. Instead, consider using objects to represent things like floor and wall. You can query the objects for information easily, and using a data type separate from ASCII allows you to use ASCII as its own view type (as in, being able to play in both ASCII or tiles at the flip of a switch). 

For the purposes of this article, let's refer to the tile type object's class as RLTileType (the RL is for Roguelike - that's how I roll). 


## RLTileType
Probably the most gameplay-centric features of the RLTileType class are the permissions. For instance, can you walk on this tile type, does it block projectiles, can you drop items on it, can you see through it, how much damage do I take each turn I stand there, etc. Since this tutorial isn't about the gameplay aspects, I'm just going to gloss over this part and assume that you can figure out what to do with it.

More importantly, each RLTileType requires a unique identifier. I, personally, chose a string with a unique name in it, like "FLOOR" or "WALL". You could use an ascii character for this, but at least use it as a string since you will be able to use it as an identifier in a hashtable or associative array (why this is important, I'll talk about soon). Personally, I kept the ascii character and a default graphic tile index as additional variables, mainly for quickie testing.

You don't need to use unique instances of RLTileType for every cell. It is enough that all floor cells point to the same, immutable floor object. 

## One Rule To Ring Them All
The basic idea behind this system is that it is a series of matching rules. If the eight neighboring cells match a particular rule, then that rule returns an index into the tile graphic array. Here's an example: 

![](https://phoebe-g.github.io/mapmaker-musings/images/map01-rule.png)

Here, I've used ASCII to represent the RLTileTypes in the rule in order to distinguish between the result of the rule, which is a graphical tile (in this case, a T-shaped ceiling tile).

The center square represents the RLTileType that we have found, as in we are trying to figure out which ceiling graphic to select. This is also used to split the rules into different rulesets by src tile type.

The eight surrounding cells represent the expected tile type to be found in each neighboring cell. If even one of these doesn't match, the rule fails. Also note that there is a wildcard type (the ANY type) which will always match, regardless of what is in that cell, and the NOT Ceiling type, which will match every cell EXCEPT a ceiling. 

## RLTypeViewRule and RLTypeViewRuleSet
The RLTypeViewRule is extremely simple. In fact, it's just data. Doesn't even need to be a class, technically. It is just eight strings which represent the unique identifiers of neighboring RLTileTypes, along with ANY and NOT types. ANY can be considered to be the default value if unspecified (NULL string). Then it has the single integer which is the index into the tile graphics array.

When a map cell is checked against a rule, the surrounding eight cells are checked against the the rule's identifiers for those cells. ANY always matches. NOT TYPEs match everything except the specified TYPE. Otherwise, the rule matches if the cell and the specified type are the same. If any one of the eight neighbors does not match, the whole rule fails.

All these RLTypeViewRules of a particular tile type are then collected into an ordered array, the RLTypeViewRuleSet. All floor rules are kept in a set, all ceiling rules in a set, and so on. Thus, when you want to match a cell, you simply select the ruleset for that particular type.

When trying to match, the ruleset will go through through the rules, in order, until it finds the first match. Then it returns the tile graphic index given to it by the matching rule. If no rules match, a default tile graphic index is returned. And that's pretty much it, except that you might want to wrap all of this up into one clean little class that hold all the rulesets togetherâ€¦ 

## RLEnvironment
The RLEnvironment is sort of the glue that holds everything together. It is basically everything you need to take a type map and turn it into a view map.

At the most basic level, it is a dictionary of rulesets (organized by type) and the tileset in question. It also has the function for loading the rulesets from an XML file.

For additional ASCII support, you can add a dictionary that will convert from ASCII symbols to RLTileTypes, as well as support functions for creating TypeMaps from AsciiMaps and vice versa.

Finally, you have the functions which will actually enact the rules, creating a new ViewMap from an inputted TypeMap. I also recommend an update function, where you can pass an position (x,y) and a typeMap and get the tile graphics index of that location - for when a TypeMap changes and you need to update the ViewMap accordingly. 

----

Copyright 2009 Sean Howard. All rights reserved.

Rescued from internet archive and google image cache in 2019.
