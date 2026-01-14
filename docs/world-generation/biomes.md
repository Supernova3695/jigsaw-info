---
title: Biomes
category: General
tags:
    - guide
mentions:
    - SirLich
    - solvedDev
    - stirante
    - Joelant05
    - destruc7ion
    - SmokeyStack
    - MedicalJewel105
    - aexer0e
    - Apex360
    - Lufurrius
    - TheItsNameless
    - ThomasOrs
    - SmokeyStack
    - Supernova3695
description: Biomes guide.
---

_Last updated for 1.21.132_

:::warning
[Nether biome generation](#the-nether) is in preview as of preview 26.0.28
:::

Behavior packs allow for the customization of biomes. A behavior pack can either create entirely new **custom biomes** which can replace vanilla biomes. [**overrides** for previously declared biomes](#inheritance), such as the vanilla biomes. Biomes hook into critical gameplay features, such as mob spawning, data-driven gameplay, and presentation of custom blocks. Biomes also enable a powerful system for adding decorations like flowers and trees, or even structures like towers and houses; these decorations and structures are together known as [features](#/concepts/features/), which are crucial to world generation but (generally) separate in scope and construction from biomes.

Overrides should retain the original biome’s identity and intentions and should only be reserved for:

-   Mild surface, heightmap, or climate adjustments
-   Redistribution of biome rarity in world generation
-   Addition of new features or mobs, but only if thematically appropriate

Custom biomes should be used when _any_ unique gameplay experience is desired or if an adjustment to a previously declared biome would fundamentally change its nature. Examples of situations where custom biomes shine include:

-   A new or radical terrain is required to achieve an aesthetic.
-   Custom features, like a new tree type, need somewhere to generate.
-   An alternate or more challenging gaming experience is desired, potentially using new mobs and structures.

## Biome Definitions

Biomes are declared in a file of the form `*biome_name*.json` or `*biome_name*.biome.json` in the top-level `biomes` directory of a behavior pack. Subdirectories may not be used within the `biomes` directory to group biome definitions; all definitions within sub-directories of `biomes` will be ignored.

Identifiers for biomes must be namespaced such as `wiki:magical_grove`. 

> Invalid JSON files declared in the top-level `biomes` directory are more likely to log errors, but they may cause crashes. Non-JSON files directly placed inside this directory are ignored. If a file exists directly inside `biomes` that begins with a `.`, the game currently always crashes. This can cause problems with files such as those used for project configuration or even the infamous [.DS_Store file](https://en.wikipedia.org/wiki/.DS_Store) on macOS.

### Format

Like all constructed assets in a behavior pack, biome definitions are written in JSON, such as:

<CodeHeader></CodeHeader>

```json
{
    "format_version": "1.21.130",

    "minecraft:biome": {
        "description": {
            "identifier": "wiki:pumpkin_pastures"
        },

        "components": {
            "minecraft:replace_biomes": {
                "replacements": [
                    {
                        "amount": 0.5,
                        "noise_frequency_scale": 50,
                        "dimension": "minecraft:overworld",
                        "targets": [
                            "minecraft:plains"
                        ]
                    }
                ]
            },
            "minecraft:surface_parameters": {
                "foundation_material": "minecraft:stone",

                "top_material": "minecraft:grass",
                "mid_material": "minecraft:dirt",

                "sea_floor_depth": 4,
                "sea_material": "minecraft:water",
                "sea_floor_material": "minecraft:sand"
            },
            "minecraft:overworld_height": {
                "noise_params": [0.125, 0.0625]
            },
            "minecraft:climate": {
                "temperature": 0.375,
                "downfall": 0.25,
                "snow_accumulation": [0, 0.5]
            },
            "minecraft:village_type": "default",
            "minecraft:tags": {
                "tags": [
                    "overworld",
                    "pumpkin_pastures",
                    "animal",
                    "monster"
                ]
            }
        }
    }
}
```

> Invalid JSON — like with all aspects of add-ons — causes a biome definition to fail; that biome will not generate in the world. Unfortunately, no error will be thrown. A JSON validator and/or syntax highlighter easily makes this a non-problem.

#### Format Version

<CodeHeader></CodeHeader>

```json
"format_version": "1.21.130"
```

The top-level property `"format_version"` describes the version specification to which the proceeding schema conforms. The latest issued `"format_version"` for biomes is `"1.21.130"`. Any version lower then this will fail to generate.

#### Biome Specification

<CodeHeader></CodeHeader>

```json
"minecraft:biome": {
	…
}
```

The other top-level property is `"minecraft:biome"`, which establishes the schema for the biome definition.

##### Description

<CodeHeader></CodeHeader>

```json
"description": {
	"identifier": "wiki:pumpkin_pastures"
}
```

The `"description"` property of the `"minecraft:biome"` property is used as the metadata for the biome. It currently contains only one property, `"identifier"`, which is used to uniquely identify a biome. This identifier is used for referencing from a number of biome definition properties.

##### Components

<CodeHeader></CodeHeader>

```json
"components": {
	…
}
```

The `"minecraft:biome"` property also holds the `"components"` property, which is the meat of a biome definition. The components declared here place, shape, and style biomes.

> Component details are scattered throughout the rest of this document; they cannot be as neatly described or organized as these wrapper properties due to intricacies in their interactions.

Components are always object properties, even those that should seemingly act as booleans. For example, the `"minecraft:ignore_automatic_features"` component property is not assigned `true` or `false`, but instead an empty object, `{}`:

```json
"components": {
	…

	"minecraft:ignore_automatic_features": {}
}
```

Although the JSON may be invalid, properties of the same name representing components may be used within the `"components"` object. Although this situation should be avoided, it should be noted that only the last provided instance of a component will be used by the game: the values inside the earlier defined component(s) will be completely ignored. For example:

```json
"components": {
	…

	"minecraft:overworld_height": {
		"noise_type": "ocean"
	},
	"minecraft:overworld_height": {
		"noise_params": [1.25, 0]
	}
}
```

Despite the fact that the `"noise_type"` property would [typically completely override](#heightmap) the `"noise_params"` property, the preset will be ignored in this example due to the order of the components within the `"components”` object.

When a component is used in a biome definition, _all_ of its required properties must be provided; if they aren’t, an error will be thrown and the biome will fail to generate.

> Because of how the inheritance model works, if incomplete components across the inheritance chain for a biome definition would contain the properties needed to complete the schema for that component, the component will work correctly. Interestingly, this situation is not true when both components reside inside the same biome definition file. As described before, only the latter component will be used, even if it is missing required properties.

###### Tags

<CodeHeader></CodeHeader>

```json
"components": {
	…
    "minecraft:tags": {
        "tags": [
            "overworld",
	        "pumpkin_pastures",
	        "animal",
	        "monster"
        ]
    }
}
```

In addition to the components that work to create a biome, the `"components"` property also allows for the addition of arbitrary tags. Tags appear like empty components, e.g., `"animal": {}`. Tag names must conform to the regular expression `[a-z0-9_.:]+`, that is: lowercase Latin letters, Arabic numerals, underscores, periods, and colons.

> For future-proofing, it is not recommended to create a tag with the prefix `minecraft:` to ensure a tag does not clash with a future Mojang-defined component. Furthermore, it is suggested to use a pack-specific namespace with these tags to minimize collisions with other behavior packs, such as `"betterbiomes:arboreal"`.

### Inheritance

Biome definition files can act as initial definitions or overrides depending on behavior pack ordering. The earliest appearance of a biome definition in a behavior pack stack marks the creation of a custom biome; subsequent definitions of the same biome in the behavior pack stack can modify or override earlier definitions through inheritance.

Only components and tags in the `"components"` property are inherited. Properties within an individual component are also usually inherited. Unfortunately, some components or component property objects require complete redeclaration of all their properties to work, meaning it is often better to redeclare an entire component when overriding. Inheritance always occurs unless a new component would interfere with a previously existing component, as is typically the case with [surface builders](#inheritance-considerations).

There is no way to indicate a property should be removed from earlier definitions. This can especially be troublesome with [tags](#tagging) due to their usage in signifying biome placement and how they power other gameplay elements like mob spawning. If conflicts arise due to inheritance issues, it is recommended to extract the desired elements of a biome into a new custom biome and attempt to remove the old biome from [world generation](#generation).

Biome files may uselessly be empty if overriding a previously declared biome. If a biome definition, initial or override, contains any components, it must contain the top-level `"format_version"` property and down to the `"identifier"` property. Initial definitions of a biome must contain at least one component or tag; the required declaration is small because defaults for almost every biome aspect are available for fallback.

## Generation

The `"minecraft:replace_biomes"` component is the only way to generate custom biomes in the nether and the overworld. It allows custom biomes to replace vanilla biomes at a percentage such as 50%. If two addons try to replace the same biome the biome with the lower `noise_frequency_scale` value will win.

### Overworld

<CodeHeader></CodeHeader>

```json
"minecraft:replace_biomes": {
    "replacements": [
        {
            "amount": 0.5, //How much of the target biome will be replaced
            "noise_frequency_scale": 50, //Fequency of replacement attempts. Value between 0 and 100 inclusive.
            "dimension": "minecraft:overworld", //Dimension can be minecraft:overworld or minecraft:nether
            "targets": [
                "minecraft:plains"
            ]
        }
    ]
}
```



Minecraft only allows the player’s first load in a select few biomes:

-   Plains
-   Forest
-   Taiga
-   Dark Forest
-   Savanna
-   Jungle

The variants of these biomes, such as Shattered Savannas and Flower Forests, also allow for player load-in. If none of these biomes are present due to de-weighting (and in the case of the Plains and Forest biomes, additionally being unlisted as [sub-biomes of Deep Oceans](#islands)), the player usually will not be able to load in to the world: the game most often will search for a valid spawn location endlessly.

> In some rare, inexplicable cases, the player will be thrown into a biome not ordained for player loading at the world origin after enough time has passed attempting to find a valid load-in spot.


### The Nether

<CodeHeader></CodeHeader>

```json
"minecraft:replace_biomes": {
    "replacements": [
        {
            "amount": 0.5, //How much of the target biome will be replaced
            "noise_frequency_scale": 50, //Fequency of replacement attempts. Value between 0 and 100 inclusive.
            "dimension": "minecraft:nether", //Dimension can be minecraft:overworld or minecraft:nether
            "targets": [
                "minecraft:hell"
            ]
        }
    ]
}
```


### Surface Builders

Whereas heightmaps are used to control the general shape of a biome, surface builders are used to style biomes. Surface builders provide two key mechanisms for this styling: a schema to which blocks can be assigned for actual terrain generation and optionally a set of large-scale adornments to make a biome stand out.

The optional adornments allow for biome terrain features that would otherwise be impossible using only heightmap adjustments or challenging using biome features; these adornments are either intricately shaped or massive in size. Unfortunately, there is no way to create a surface builder; the provided surface builders exist solely to represent complex vanilla biome surfaces.

> Adornmenrts created by surface builders are unfortunately fixed and not relative to the declared heightmap. This means that if the heightmap at the location of a given surface builder-created decoration is high or low enough, the decoration will not appear to exist, consumed by the land.

##### Surface Types

###### Default

###### Capped

###### Swamp

###### Mesa

###### Frozen Ocean

###### The Nether

###### The End

###### The End

The End surface is the designated surface for The End dimension and its lone biome. [Because the End’s foundation material is not configurable](#dimensional-considerations), the End surface only works to generate a top material

#### Dimensional Considerations

The Nether may be able to take on the actual surface materials of certain Overworld-specific surface builders, but the features of these surfaces, such as badlands spires, will never generate.

The End’s foundation material is not configurable in any way, even if using the only other surface type allowed in The End, [capped surfaces](#capped).

#### Inheritance Considerations

Due to biome inheritance, surface builders declared in later definitions of a biome may conflict with earlier definitions. Because each surface builder type is its own component, the default resolution system for inheritance pits the builders against each other, typically leading to some strange results.

Only the [default surface builder](#default) can be overridden. Overrides will fail if attempted for any other builder, but the inner schema of the failed override may actually yet have an effect on generation. This occurs when the schemas of two different builders share properties of the same name and type. The inheritance system here will still use the initial, un-overridable surface builder, but its block declarations will be overridden with those from matching properties from the latter-declared surface builder.

### Surface Adjustments

<CodeHeader></CodeHeader>

```json
"minecraft:surface_material_adjustments": {
	"adjustments": [
		{
			"materials": {
				"top_material": "minecraft:podzol"
			},

			"noise_range": [0, 0.5],
			"noise_frequency_scale": 0.0625,
			"height_range": [72, 255]
		}
	]
}
```

Surface adjustments allow for fine-tuning a biome's surface blocks. Despite being called "surface" adjustments, these adjustments can actually affect all blocks declared in an eligible surface builder. These adjustments cannot modify blocks outside the scope of a surface builder and therefore cannot be used to alter bedrock or air, whether the air generates in caves, above the heightmap, or between shelves of land (if the heightmap is radical enough). Currently, only the default and swamp surface builders support adjustments.

Surface adjustments declarations are implemented using objects in the `"adjustments"` property of the `"minecraft:surface_material_adjustments"` component. These declarations contain both overrides for blocks declared in the biome’s surface builder and the conditions under which these adjustments should occur.

Surface adjustment conditions can check against a [random noise surface](#noise-intersections) dependent on the _x_ and _z_ coordinates using `"noise_range"` and `"noise_frequency_scale"` or a [simple range](#height-restrictions) of _y_ coordinates via `"height_range"`. If all coordinates should be considered, both conditions can be used. For an adjustment to be applied to a location, every declared condition must succeed; it any fail, the condition check fails, and the game will fall back to the surface builder’s declared block for that location.

No default surface adjustments are forced automatically upon a biome. If no adjustments are listed anywhere along the inheritance chain for a biome, no adjustments will be observed in that biome.

#### Noise Intersections

<CodeHeader></CodeHeader>

```json
"noise_range": [-1, -0.5],
"noise_frequency_scale": 0.125,
```

A noise curve that is dependent upon the seed of a world can be used to restrict the _x_ and _z_ components of a surface adjustment. The origin of this noise curve is centered on the world origin and [can then optionally be scaled via `"noise_frequency_scale"`](#sizing) to map onto the horizontal plane of a dimension. The noise curve can therefore only work on this horizontal plane and not on the _y_ coordinate; for that, use [height restrictions](#height-restrictions). To actually use the noise curve to restrict adjustments, a success interval must be provided using `"noise_range"`.

The exact value generated from the noise curve at a particular location is inconsequential to the resultant surface adjustment. The only consideration is whether the value at that location meets the conditional check.

> Although both curves are formed based on the world seed, the noise curve used for surface adjustments is not equivalent to the noise curve used with `"q.noise"`. Their correspondence cannot be depended upon for generation.

##### Intervals

<CodeHeader></CodeHeader>

```json
"noise_range": [0.5, 1]
```

The noise curve used for surface adjustments generates values lying on the closed interval [-1, 1]. Values on this interval are targeted via a sub-interval using the `"noise_range"` property.

The noise curve itself has some notable properties that should be understood when using surface adjustments. Negative values generated by Perlin noise behave symmetrically with the positives; that is: an intersecting sub-interval to this curve [-0.8, -0.6] would theoretically have approximately the same shape and represent approximately the same ratio of the full range as the sub-interval [0.6, 0.8]. Intervals closer to `0` tend to form continually winding striations in the surface, while those further away tend to form small, discrete, well spread shapes. Intervals closer to `0` also intersect a larger range of the noise curve than intervals of equal length further from `0`; this means that if discrete shapes are desired, like those that are formed at the extremes of the noise curve’s range, a larger relative interval may be required than one targeting the winding paths close to `0`.

##### Sizing

<CodeHeader></CodeHeader>

```json
"noise_frequency_scale": 0.25
```

The surface adjustment noise curve uses a default mapping relative to the dimension coordinates that would form extremely small transformations. The noise intersections are actually made larger by setting the sizing value to be smaller. Smaller values, such as `0.125`, are key to forming large patches of transformations, such as podzol or coarse dirt clusters. Larger values of this property are typically used to create a messier feel to the terrain. Values greater than the default `1` are not recommended unless something akin to a checkerboard pattern is desired.

#### Removal

Surface adjustments from earlier definitions of a biome can be removed by matching the definitions from the biome’s [surface builder](#surface-builders) across the relevant conditions. Take, for example, the vanilla surface adjustments made to the Shattered Savanna:

<CodeHeader>biomes/savanna_mutated.json</CodeHeader>

```json
"minecraft:surface_parameters": {
	"foundation_material": "minecraft:stone",

	"top_material": "minecraft:grass",
	"mid_material": "minecraft:dirt",

	"sea_floor_depth": 7,
	"sea_material": "minecraft:water"
	"sea_floor_material": "minecraft:gravel",
},
"minecraft:surface_material_adjustments": {
	"adjustments": [
		{
			"materials": {
				"top_material": "minecraft:stone",
				"mid_material": "minecraft:stone"
			},

			"noise_range": [0.212, 1.0],
			"noise_frequency_scale": 0.0625
		},
		{
			"materials": {
				"top_material": {
					"name": "minecraft:dirt",

					"states": {
						"dirt_type": "coarse"
					}
				}
			},

			"noise_range": [-0.061, 0.212],
			"noise_frequency_scale": 0.0625
		}
	]
}
```

To revert the surface back to its original, “paint back over” the surface with the original blocks, like:

<CodeHeader></CodeHeader>

```json
"minecraft:surface_material_adjustments": {
	"adjustments": [
		{
			"materials": {
				"top_material": "minecraft:grass",
				"mid_material": "minecraft:dirt"
			},

			"noise_range": [0.212, 1.0],
			"noise_frequency_scale": 0.0625
		},
		{
			"materials": {
				"top_material": "minecraft:grass"
			},

			"noise_range": [-0.061, 0.212],
			"noise_frequency_scale": 0.0625
		}
	]
}
```

### Climate

<CodeHeader></CodeHeader>

```json
"minecraft:climate": {
	"temperature": 1,
	"downfall": 0.25,
	"snow_accumulation": [0.0, 0.125],
	"ash": 1
}
```

A biome’s climate mostly represents its ambient aesthetic. Aspects of a biome’s climate may have an effect on gameplay, but this is rarely used in vanilla. Climate adjustments work everywhere in Minecraft, even The End, but the adjustments may have no effect if a dimension doesn’t support a specific climate feature, such as [precipitation](#precipitation) outside the Overworld.

All aspects of a biome’s climate are optional. Defaults that are sensible for the Overworld are provided as fallbacks.

#### Temperature

<CodeHeader></CodeHeader>

```json
"minecraft:climate": {
	"temperature": 0.5
	…
}
```

The **temperature** of a biome affects various gameplay features like [precipitation type](#precipitation), the formation of ice and [snow layers](#snow-cover) when the appropriate blocks are directly exposed to sunlight, and the survivability of snow golems. It is implemented as the float property `"temperature"` and may be set without limitation: all possible float values may be used. Lower values represent colder temperatures. Freezing temperatures, such as where snow falls, occur below `0.15`. If no temperature is provided for a biome definition, the game will use a value of `0.5`; at this temperature, freezing effects cannot be observed anywhere blocks can be placed in any dimension.

> The effects of temperature are not restricted to the Overworld, but fewer effects may be available in other dimensions: Snow Golems may even survive in the Nether if the temperature at a _y_-height in the dimension is at freezing or below.

> The `"temperature"` property is unrelated to the climate temperatures given in either the `"generate_for_climates"` property in the `"minecraft:overworld_generation_rules"` component or the `"target_temperature"` property in the `"minecraft:nether_generation_rules"` component; the property here affects gameplay rather than biome placement.

The temperature established with this property is not the fixed temperature of a biome, only the basis; the actual temperature at a location also depends on the _y_-height. The temperature at or below sea level is fixed to this basis. Above sea level, however, the temperature decreases by 1 / 600 every block. The formula, therefore, of the temperature, _T_, at a given _y_-height, _y_, from a declared temperature basis, _t_, for _y_-heights above sea level, _s_, is given by:

_T_(_y_) = _t_ - ((_y_ - _s_) / 600)

To establish a _y_-height at which a biome will freeze, use the formula:

_t_(_y_) = 0.15 + ((_y_ - _s_) / 600)

The value of the sea level, _s_, depends on the dimension:

| Dimension  | Sea level |
| :--------- | :-------- |
| Overworld  | 63        |
| The Nether | 32        |
| The End    | 63        |

The constant 0.15 here represents the freezing temperature. Data-driven gameplay can use the temperature at the position of an entity to perform actions upon that entity using the `"is_temperature_value"` damage condition filter in an entity definition, but this is outside the scope of this document.

Vanilla biomes only use temperature values ranging from `-0.5` to `2`. Biomes to be completely covered in snow should use values less than `0.15`; `0` is the recommended value for this case. Warm biomes, such as those that would damage snow golems, should use values greater than `1`. Deserts and Nether biomes use `2`. For biomes that should only be capped with snow layers, a value between about `0.2` and about `0.4` should be used due to [how snow layer generation is affected by temperature](#snow-cover).

#### Precipitation

<CodeHeader></CodeHeader>

```json
"minecraft:climate": {
	"downfall": 0.5
	…
}
```

**Precipitation** is visible as rainfall or snowfall in the Overworld. The float property `"downfall"` controls the extent of precipitation within a biome. Currently, the property only behaves on the closed interval [0, 1]; values outside this interval are clamped to the nearest valid value. At `0` or less, this property disables all precipitation effects within a biome. At `1` or greater, precipitation effects will be maximized. Values between `0` and `1` scale accordingly.

> Due to a bug, adjustments to precipitation are _never_ reflected visually in the particles that fall when raining or snowing. Precipitation particles are never visible in Desert variants, Savanna variants, or Mesa variants, no matter their overrides; particles are _always_ visible in any other biome, including custom ones. Despite this, _most_ gameplay affected by precipitation will behave appropriately.

Examples of precipitation effects in vanilla gameplay include the addition of snow layers when snowing, the filling of exposed cauldrons in rain, and the time required for a fish to spawn when fishing. Some gameplay aspects that should consider this value will instead ignore it, such as entities with the `"in_water_or_rain"` damage filter. Lesser values of `"downfall"` will slow precipitation effects accordingly, i.e., snow layers will form a tenth as fast at a value of `0.1` as they would at `1`.

The type of precipitation occurring at a location depends on its _y_-coordinate. If the value is [freezing, `0.15`, or below](#temperature), snow will fall at that location; otherwise, rain will fall.

#### Snow Cover

<CodeHeader></CodeHeader>

```json
"minecraft:climate": {
	"snow_accumulation": [1, 0.5],
	…
}
```

When a biome is generated, blocks that are [at a freezing temperature](#temperature) typically have **snow cover** above them: snow layers stacked atop eligible blocks. The `"snow_accumulation"` property is used to adjust this snow cover. Snow covering seems to occur as one of the final passes to biome generation, meaning snow will cover features as well as the biome’s surface.

> Snow can also cover a biome post-generation as a part of [snow precipitation](#precipitation), but the `"snow_accumulation"` property has no effect in this situation. Use the [`"precipitation"` property](#precipitation) to adjust snow formation rate.

This property takes a two-valued array of float values, [*a*, *b*], where _a_ affects the average height of snow covering eligible blocks and _b_ seemingly affects the distribution of the snow.

In particular, _a_ represents the maximum block count of snow layers above blocks at a freezing temperature. On average, the snow layer block height will actually be about 40% of this value. Blocks having no snow covering will be uncommon, and blocks having the maximum snow covering will be almost impossibly rare. Because this value represents block height and each actual snow layer is only one-eighth of a block tall, float values must be used for targeting shallower snow covering. For example, to have snow cover that is at most 4 layers tall, use a value of `0.5`. At this value, almost all eligible blocks will have a covering that is either 2 or 3 layers thick.

The second value of the array, _b_, is intended to adjust snow distribution, but many values for _b_ cause erratic behavior, and even well-behaved values have little impact on generation. Values of `0` or less always result in a single layer of snow cover above eligible blocks, entirely disregarding the value set for _a_. Values exclusively between `0` and `0.125` cause a seemingly random spread of entire chunks to be only covered in a single layer of snow, while all other chunks respect the value set for _a_. Values at `0.125` or greater cause mild adjustments in snow cover that are not relative to the value set for _a_; in other words, extreme values of _a_ make alterations caused by _b_ to be virtually unnoticeable. Furthermore, the adjustments caused by _b_ only apply to a small spread of blocks, making this value rather useless at best and buggy at worst. _It is therefore recommended to just set this value to 0.5 wherever `"snow_acucmulation"` is declared._

> There is no way to conveniently avoid snow cover in frozen areas. Snow cover will not effect blocks not suitable for snow layer placement. To avoid snow, choose a warmer [`"temperature"` value](#temperature).

#### Particle Decorations

<CodeHeader></CodeHeader>

```json
"minecraft:climate": {
	…

	"white_ash": 0.5
}
```

**Particle decorations** are storms of ambient particles visible within a biome. These properties are solely decorative; unlike other aspects of a biome’s climate, particle decorations have no effect on gameplay. If not provided, no particle effects will be present in a biome. Custom particles currently may not be used. 4 different particle decorations from vanilla biomes are available to use anywhere:

| Decoration type | Property name   |
| :-------------- | :-------------- |
| Ash             | `"ash"`         |
| White ash       | `"white_ash"`   |
| Red spores      | `"red_spores"`  |
| Blue spores     | `"blue_spores"` |

These decorations will be a constant presence within a biome. If a player is within the bounds of a biome, these particles will be visible, even if underground or within water or lava. If undeclared, no particles will be used in a biome.

As they are represented as float properties in the `"minecraft:climate"` component, their intensities are adjustable. The float value works for any number greater than `0`. The larger the value, the more particles will be used simultaneously; negative values or `0` will disable the effect.

> It is not recommended to set this value too large, as the particle count may cause crashes to occur. By a value of `16`, the screen will be inundated with particles, but the biome will still be barely visible through the storm.

## Gameplay

Biomes are the starting point of much of the configurable gameplay in Minecraft. Features, such as trees and villages, can only generate as part of biomes. Automatic mob spawning can then be configured for biomes and structures. Feature attachments and mob spawning are outside the scope of this document, but there are a few configurations for biome definitions that power how biomes interact with other gameplay systems.

### Structures

<CodeHeader></CodeHeader>

```json
"minecraft:village_type": "default"
```

A specific village type can be set for custom biomes and villages generated in that biome will assume that type of village.

| Village type | Village name   |
| :----------- | :------------- |
| Plains       | `"default"`    |
| Savanna      | `"savanna"`    |
| Desert       | `"desert"`     |
| Taiga        | `"taiga"`      |
| Snowy        | `"ice"`        |

### Features

<CodeHeader></CodeHeader>

```json
"minecraft:forced_features": {
	"surface_pass": {
		"identifier": "wiki:grasslands_caravan_feature",
		"places_feature": "wiki:caravan_feature",

		"scatter_chance": "100 * math.pow(2, -4)",

		"x": {
			"distribution": "uniform",
			"extent": [0, 16]
		},
		"z": {
			"distribution": "uniform",
			"extent": [0, 16]
		},
		"y": "q.heightmap(v.worldx, v.worldz)"
	}
},
"minecraft:ignore_automatic_features": {}
```

If a collection of blocks generating in a world aren’t created by a surface builder, they are created from a feature. Features are fundamental to gameplay in Minecraft: vanilla features range from trees to villages to boulders. The features in a biome may handily determine that biome’s worth to a player.

Features are mostly outside the scope of biomes, but the two components within a biome’s schema affecting feature generation should be noted.

#### Forced Features

<CodeHeader></CodeHeader>

```json
"minecraft:forced_features": {
	"surface_pass": [
		{
			"identifier": "wiki:redwood_tree_feature",
			"places_feature": "wiki:redwood_tree",

			"iterations": "math.random_integer(2, 4)",

			"x": {
				"distribution": "uniform",
				"extent": [0, 16]
			},
			"z": {
				"distribution": "uniform",
				"extent": [0, 16]
			},
			"y": "q.heightmap(v.worldx, v.worldz)"
		}
	]
}
```

The `"minecraft:forced_features"` component can be used to force a feature to generate within a biome without using [feature rules](#/concepts/features/).

Forced features are placed in array order for each pass. When overriding a previously provided array of feature attachments for a placement pass, the previous feature attachments will be completely ignored by the override; only the new attachments will be used. To use the old attachments with the new ones, they must be redeclared within the override.

By default, no forced features are implied in a definition; if they are never declared down the inheritance chain, no additional features will exist. Note that [some features](#immutable-features) are hard-coded and cannot be removed easily.

#### External Features

The empty `"minecraft:ignore_automatic_features"` component is intended to indicate that a declaring biome will ignore all external feature rules attached to it, but this component currently does not work, whether for overrides or initial definitions.

##### Immutable Features

A few features may even generate in a custom biome with no attached tags. These features current cannot be removed conveniently:

-   Springs (water and lava “lakes”)
-   Ruined Portals
-   Mineshafts
-   Dungeons<sup>\*</sup>
-   Nether Fortresses

> The Dungeon structure may generate infrequently due to the air created from the other Overworld immutable features; its generation is typically dependent on cave systems, [which may be disabled](#caves).

These features occur before even the first placement pass of data-driven features and can even cut through the Bedrock layer. The immutable features can be overridden by data-driven features, but this is typically too challenging or expensive to consider.

One typically ubiquitous Overworld structure, Strongholds, cannot be configured to generate in custom biomes no matter what.

### Caves

Carvers generate cave-like features in the world. Unlike the Java Edition of Minecraft, the Bedrock Edition currently provides no way to customize carvers. However, cave generation can be influenced by the types of blocks used in a [biome’s surface](#surface-builders). A whitelist of blocks allow carvers to cut through them to generate caves. At minimum, these blocks include:

-   Stone
-   Dirt
-   Sandstone
-   Grass
-   Podzol
-   Mycelium
-   Sand

All of the variants of these blocks, such as Polished Andesite (a variant of Stone) can be culled by carvers. Custom blocks cannot currently be configured to be culled, often leaving heavily customized biomes without caves. Culling is not stopped by blocks not on the whitelist, so if only the top layer of a surface builder isn’t whitelisted, cave generation may resume underneath it, assuming those blocks below _are_ whitelisted.

Caves are created only after the heightmap of a surface (including its adjustments) has been constructed; decorations created from surface builders, such as icebergs, are not interrupted by cave generation. Caves are always generated in full before even the first placement pass of features occurs.

### Tagging

<CodeHeader></CodeHeader>

```json
"components": {
    "minecraft:tags": {
        "tags": [
            "urban",
            "city",
            "metro",
            "rare"
        ]
    }
}
```

Tags power much of what brings a biome to life in Minecraft, including entity spawns, external feature attachment, and data-driven gameplay. See [Tags](#tags) for the implementation details of tags.

No tags are implied based on the nature of a biome. For example, if a biome is set to generate in the Overworld, the `"overworld"` tag used on such biomes will need to be manually added to opt-in to the consequences of that tag. 

##### Biome Taxonomy

A taxonomical system can be created to bind biomes to other systems by putting the focus on the biome itself. Adding tags that help single out a biome by its generation rules can be useful for features that need to generate under conditions shared by a set of biomes due to their inherent nature. Some examples of a basic taxonomy would include using tags to specify:

-   A name for the basic type of biome
-   The dimension where a biome would generate
-   The [type of climate](#climates) or [strength of a defining characteristic](#separation-of-concerns) for the biome
-   [Variants](#hierarchy) of a biome

Example taxonomical tags for a theoretical Pumpkin Pastures Spooky Hills biome could include:

-   `"overworld"`
-   `"cold"`
-   `"pumpkin_pastures"`
-   `"hills"`
-   `"mutation"` (the “spooky” aspect)
-   `"spooky"`, but only if this mutated aspect were shared with other biomes

Vanilla biomes have historically relied exclusively on taxonomical systems for biome selection, often leading to verbose success conditions for spawning or generation. Here’s a snippet from the `wolf.json` spawn rules:

<CodeHeader></CodeHeader>

```json
"minecraft:biome_filter": {
	"all_of": [
		{"test": "has_biome_tag", "operator":"==", "value": "forest"},
		{"test": "has_biome_tag", "operator":"!=", "value": "mutated"},
		{"test": "has_biome_tag", "operator":"!=", "value": "birch"},
		{"test": "has_biome_tag", "operator":"!=", "value": "roofed"},
		{"test": "has_biome_tag", "operator":"!=", "value": "mountain"}
	]
}
```

In cases such as this, another perspective can be used for tagging that eases authoring pains.

##### Spawning & Generating Perspective

Authoring spawning and generating conditions can be made much easier by shifting the tagging perspective onto feature and spawn rules. [Minecraft 1.16 began using such a system for mob spawning in the Nether.](#other-mobs) In this system, tags are attached to a biome with actual gameplay in mind. If a custom mob were to be created that wasn’t meant to neatly spawn in a convenient classification of biome taxonomy, alluding to spawn rules via tagging can keep condition checks simple.

Imagining bird mobs that could spawn in any area with trees, a tagging system could be developed dependent on how wooded a biome is. From another perspective, these biomes could also be tagged as `"forested"` and even go so far as to provide tags relating the density of trees:

| Forest cover | Feature-focused tag | Biome taxonomy-focused tag |
| :----------- | :------------------ | :------------------------- |
| Light        | `"few_birds"`       | `"lightly_forested"`       |
| Medium       | `"default_birds"`   | `"moderately_forested"`    |
| Heavy        | `"many_birds"`      | `"heavily_forested"`       |

The problem with the latter system is what can happen during development if some other gameplay aspect relies on trees existing in a biome. Continuing the scenario, pretend only the biome taxonomy system were to be used:

An abandoned forest shack structure feature meant to hide under the cover of trees is created; its feature rule will check for any of the 3 forested tags for generation. Later in behavior pack development, a new, heavily forested, toxic waste biome, Toxic Woods, is constructed with the `"heavily_forested"` tag. Abandoned shacks could make sense here to match the environment, but nature should be scarce or even missing; few or no birds should spawn. Unfortunately, the `"heavily_forested"` tag here would cause many birds to spawn, which is undesirable. The spawn rules for the birds would have to be updated to blacklist Toxic Woods.

With dozens or hundreds of biomes, features, and mobs across the game all relying on the same tagging system, such exceptions as this can add up. Consider using feature-focused tagging systems. These systems don’t have to compete with taxonomy; both types can be included and used as needed.

#### Vanilla Tags

The tags used on vanilla biomes should be noted for biome-altering behavior packs that only mildly change or add to vanilla generation. These tags could be used to emulate vanilla gameplay aspects in a custom biome.

##### Location & Variation

Most tags used in vanilla biomes are used to help organize biomes by location and variant, forming a sort of [taxonomy](#biome-taxonomy) for targeting biomes.

> A `"no_legacy_worldgen"` tag exists in the biome definition of Roofed Forests, but its behavior is unknown.

###### Dimensions

<CodeHeader></CodeHeader>

```json
"components": {
	…

	"overworld": {}
}
```

4 tags exists supporting the game’s dimensions:

| Dimension            | Tag                      |
| :------------------- | :----------------------- |
| Overworld            | `"overworld"`            |
| Overworld generation | `"overworld_generation"` |
| The Nether           | `"nether"`               |
| The End              | `"the_end"`              |

> The Overworld generation tag is required by Minecraft to support legacy features and behaviors. Features, entity spawning, and gameplay that needs to target all the Overworld should filter for both `"overworld"` _and_ `"overworld_generation"`, but the construction of this targeting is outside the scope of this document.

These tags have mixed effects on biomes but are predominantly used to further organize or filter biomes in combination with other tags. The most direct effect of any tag here is that the `"overworld"` tag is what enables ores and some other underground features to generate.

###### Biomes

<CodeHeader></CodeHeader>

```json
"components": {
    "minecraft:tags": {
        "tags": [
            "taiga",
            "mega",
            "hills"
        ]
    }
}
```

A number of vanilla biome tags exist to support the specific nature of those biomes.

For the Overworld:

| Biome           | Tag                  |
| :-------------- | :------------------- |
| Plains          | `"plains"`           |
| Forest          | `"forest"`           |
| Mountains       | `"extreme_hills"`    |
| Taiga           | `"taiga"`            |
| Swamp           | `"swamp"`            |
| Flower Forest   | `"flower_forest"`    |
| Jungle          | `"jungle"`           |
| Desert          | `"desert"`           |
| Savanna         | `"savanna"`          |
| Badlands        | `"mesa"`             |
| Snowy Tundra    | `"ice_plains"`       |
| Mushroom Fields | `"mooshroom_island"` |
| Beach           | `"beach"`            |

For the Nether:

| Biome           | Tag                 |
| :-------------- | :------------------ |
| Nether Wastes   | `"nether_wastes"`   |
| Soulsand Valley | `"soulsand_valley"` |
| Basalt Deltas   | `"basalt_deltas"`   |
| Crimson Forest  | `"crimson_forest"`  |
| Warped Forest   | `"warped_forest"`   |

Two tags exists strictly to group related biomes:

| Group                          | Tag                   |
| :----------------------------- | :-------------------- |
| Snowy Tundra & Snowy Mountains | `"ice"`               |
| Crimson Forest & Warped Forest | `"netherwart_forest"` |

A few tags are used to single out unique variants:

| Variant                             | Tag         |
| :---------------------------------- | :---------- |
| Giant Tree Taiga variant            | `"mega"`    |
| Stone Shore variant                 | `"stone"`   |
| Birch Forest variant                | `"birch"`   |
| Dark Forest variant                 | `"roofed"`  |
| Bamboo Jungle variant               | `"bamboo"`  |
| Savanna & Badlands Plateaus variant | `"plateau"` |
| Jungle Edge variant                 | `"edge"`    |

###### Overworld Generation Aspects

<CodeHeader></CodeHeader>

```json
"components": {
	"minecraft:tags": {
        "tags": [
            "ocean",
            "deep",
            "warm"
        ]
    }
}
```

Groups of tags exist for slotting and matching climates and transformations in the Overworld.

##### Mob Spawning

Mob spawning is predominantly controlled by two tags: `"animal"` and `"monster"`. Finer control over the spawning of vanilla mob is provided in some special cases but often necessitates an intervention that involves both adding tags to biome definitions and overriding spawn rules to work with these new tags.

> Overriding spawn rules is outside the scope of this document.

Vanilla mobs not listed in this section are spawned via selection of vanilla biome-specific tags.

###### Animals

<CodeHeader></CodeHeader>

```json
"components": {
	"minecraft:tags": {
        "tags": [
            "animal"
        ]
    }
}
```

Using the `"animal"` tag will allow Cows, Chickens, Sheep, Pigs, and Bats to spawn in a biome.

A `"bee_habitat"` tag exists, presumably to generate beehives, but it is unused.

###### Other Mobs

<CodeHeader></CodeHeader>

```json
"components": {
	"minecraft:tags": {
        "tags": [
            "monster"
        ]
    }
}
```

The `"monster"` tag allows Zombies, Skeletons, Spiders, Creepers, Slime, Endermen, Witches, and Phantoms to spawn in a biome.

Beginning with Minecraft 1.16, a [new tagging strategy](#spawning-generating-perspective) was employed by Mojang in the new Nether biomes to separate biome type and location from functionality. A number of tags now exist for mob spawning in the Nether.

| Mob spawning rule                                                | Tag                             |
| :--------------------------------------------------------------- | :------------------------------ |
| Piglins                                                          | `"spawn_piglin"`                |
| Few Piglins                                                      | `"spawn_few_piglins"`           |
| Zombified Piglins                                                | `"spawn_zombified_piglin"`      |
| Few Zombified Piglins                                            | `"spawn_few_zombified_piglins"` |
| Ghasts                                                           | `"spawn_ghast"`                 |
| Magma Cubes                                                      | `"spawn_magma_cubes"`           |
| Many Magma Cubes                                                 | `"spawn_many_magma_cubes"`      |
| Endermen (only considered in biomes also tagged with `"nether"`) | `"spawn_endermen"`              |

##### Decorations

Currently, features and decorations are applied only through the [targeting of a a particular kind of biome](#location-variation); no dedicated tags exist for decorations.
