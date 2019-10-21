# Basic Blocks

Let's see how you can add some new blocks to the game.

> This is presuming you followed these previous tutorials (in this order):
> - [New Project](../getting_started/new_project.md)
> - [Code Structure](../getting_started/code_structure.md)
> - [Resource Structure](../getting_started/resource_structure.md)
>
> Reading these tutorials is optional but recommended:
> - [Basic Items](../items/basic_items.md)

Let's start with a `ModBlocks` class.

```java
package com.example.examplemod.blocks;

import com.example.examplemod.ExampleMod;
import net.minecraft.block.Block;
import net.minecraft.block.Block.Properties;
import net.minecraft.block.material.Material;
import net.minecraft.item.BlockItem;
import net.minecraft.item.DyeColor;
import net.minecraft.item.Item;
import net.minecraftforge.event.RegistryEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.registries.IForgeRegistry;
import net.minecraftforge.registries.ObjectHolder;

import java.util.ArrayList;

@Mod.EventBusSubscriber(bus=Mod.EventBusSubscriber.Bus.MOD)
@ObjectHolder(ExampleMod.MODID)
public class ModBlocks {

    @ObjectHolder("gem_block")
    public static Block GEM_BLOCK = null;

    @ObjectHolder("example_block")
    public static Block EXAMPLE_BLOCK = null;

    private static ArrayList<Block> blocks = new ArrayList<>();
    private static ArrayList<BlockItem> items = new ArrayList<>();

    static {
        registerBlock(new Block(Properties.create(Material.IRON, DyeColor.MAGENTA)).setRegistryName("gem_block"));
        registerBlock(new Block(Properties.create(Material.IRON)).setRegistryName("example_block"));
    }

    @SubscribeEvent
    public static void onRegisterBlocks(RegistryEvent.Register<Block> e) {
        IForgeRegistry<Block> reg = e.getRegistry();

        reg.registerAll((Block[]) blocks.toArray());

        blocks.clear();
    }

    @SubscribeEvent
    public static void onRegisterItems(RegistryEvent.Register<Item> e) {
        IForgeRegistry<Item> reg = e.getRegistry();

        reg.registerAll((Item[]) items.toArray());

        items.clear();
    }

    public static void registerBlock(Block block) {
        registerBlock(block, new Item.Properties());
    }

    public static void registerBlock(Block block, Item.Properties itemBlockProperties) {
        blocks.add(block);

        if (itemBlockProperties != null) {
            items.add((BlockItem) new BlockItem(block, itemBlockProperties).setRegistryName(block.getRegistryName()));
        }
    }

}
```

## `@OjectHolder`, `@Mod.EventBusSubscriber` and `@SubscribeEvent`

Same as with the items,
the object holder annotation tells forge to inject an instance
of your item into the field it is placed on.
If used on a class it tells forge that any later
`@ObjectHolder`s are from this mod.

The `@Mod.EventBusSubscriber` annotation tells forge that this class
wants to handle events. Because `RegistryEvent`s are fired on the `Bus.MOD` Bus and the annotation by default subscribes to the `Bus.FORGE` bus we have to specify it.

`@SubscribeEvent` points forge at the correct functions to call
for every event type specified by the function's arguments.

## Static
This time we instantiate our `Block`s and `BlockItem`s as soon as
this class is loaded in by the game. This is just to avoid having

## onRegisterBlocks and onRegisterItems
In these function we instantiate and register our blocks.
But blocks require you to have a item "version" of the block
used when the player holds your block in their inventory.

The `Block.Properties` object holds info about the blocks material
which controls its appearance on a map, the sounds it makes,
hardness, harvest tool, and so on.

As shown above the color used can be overriden with one of the values
from the `DyeColors` class or the `MaterialColor` class.

Diamond blocks use the `Material.IRON` material and their own (`MaterialColor.DIAMOND`)
material color so i did something similar here with our
gem block, but using a standard dye color.

You can also need to create an `BlockItem` and
assign a `Item.Properties` object to it for each block which is exactly
what the `registerBlock` method does (the name is a bit misleading).

You can read this section of the item tutorial for some info on what
does this object contain (but i recommend just using your IDE to explore
it yourself because the method names are very straight forward).

The default `regsterBlock(block)` call just supplies you with a
default `Item.Properties` object but you can use the other version
`registerBlock(block, prop)` to supply your own like in the item
tutorial. Passing in `null` instead in the properties parameter
(`registerBlock(block, null)`) will cause no `BlockItem` to be created
(which will make your block unobtainable for players with the 
exception of `/setblock` and `/fill`)


### Testing in-game

If you launch your game now (using the `runClient` task in your IDE), 
you should be able to give yourself your blocks using 
the `/give` command (`/give @p modid:registry_name`).
When you do that you will surely notice that your item is a purple
blocky mess. To fix this you have to add a model and a texture.

## Basic Blockstate and Model
The first thing you need is a blockstate. They allow you to
change your models texture, model, rotation etc. depending
on in-game conditions. This is the simplest blockstate you can 
have

```json
{
    "variants": {
        "": { "model": "modid:block/model_name" }
    }
}
```

The same as with textures in the item tutorial this path can 
be pretty much anything as long as it points to a valid model.

The most basic block model you can have is this:

```json
{
    "parent": "block/cube_all",
    "textures": {
        "all": "modid:block/texture_name"
    }
}
```

All this does is "extend" the default block model
using the `parent` option and specify a texture.

You will want to place your blostate file named the same way
as your block (e.g. if your registry name is `gem_block` then your file will be `gem_block.json`)
in the `src/main/resources/assets/modid/blockstates/` directory.

Then you will want to place a model file in any directory, with any name
under the `src/main/resources/assets/modid/models` directory.

You just have to make sure the path in your blockstate file matches.
(e.g. if you place your file in `models/whatever/xd.json`, then your 
blockstate must point to `modid:whatever/xd`, 
but in these tutorials i will use the typical convention of placing 
block models under `models/block`).

Then the rest is identical to item textures, just under 
the `textures/block` directory.

All you have to do is place a texture in `textures/block/`
and reference it in the block model `modid:block/texture_name`

Now you can go back to the game and press `F3 + T` to reload assets
without having to close it.

> In Intellij idea you might need to use the `Run -> Reload Changed Classes` option before `F3 + T` 
> succesfully reloads your resources 
> (If you are not running in Debug Mode this will be grayed out).
> Alternatively you can press `CTRL + F9`.

## Localization
The last thing that you need to do is give your new item a localized name.
This allows you to create translations of your mod into different languages.
All you need to do for now though is open your `src/main/resources/assets/modid/lang/en_us.json` file

All you need to do is add a new entry to the json file.
```json
{
    "item.modid.registry_name": "Item Name" // For each of your items, remember a comma at the end of each entry except the last one
}
```
Which in our case will be:
```json
{
    "item.example.gem": "Gem",
    "item.example.example": "Example Item"
}
```

Now you can just reload resources again and in the game your
item should have a proper name now.

## Advanced models
TODO
