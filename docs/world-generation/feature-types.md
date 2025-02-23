---
title: Feature Types
---

::: warning
Some links designed to reference external documents do not function, and will be updated at a later time to point to the correct resource.

Screenshots and other resources may be provided for many feature types given here at a later time.
:::

## Content Features

### Single Block Features

### Ore Features

### Structure Template Features

### Growing Plant Features

### Tree Features

### Multiface Features

## Proxy Features

Proxy features group, arrange, or gate features, including other proxy features. Proxy features themselves are incapable of having a direct effect on world generation.

All proxy features must therefore point to one or multiple **target features**: the features that are placed, rearranged, or selected by proxy features. Target features are represented as string references to the identifier of the intended feature.

### Scatter Features

```json
{
	"format_version": "1.13.0",

	"minecraft:scatter_feature": {
		"description": {
			"identifier": "arboreal:flower_patch"
		},

		"places_feature": "arboreal:flowers",

		"scatter_chance": {
			"numerator": 2,
			"denominator": 3
		},
		"iterations": "v.flower_patch.size = math.random_integer(6, 14); return math.random_integer(math.pow(v.flower_patch.size, 2) / 4, math.pow(v.flower_patch.size, 2) / 3);",

		"project_input_to_floor": true,
		"x": {
			"distribution": "gaussian",
			"extent": [0, "v.flower_patch.size"]
		},
		"z": {
			"distribution": "gaussian",
			"extent": [0, "v.flower_patch.size"]
		},
		"y": 0
	}
}
```

**Scatter features** are the most flexible and useful feature type available. Scatter features can:

-   Distribute or reposition a feature any number of times within a [chunk’s feature domain](#)
-   Act as a gate to conditionally enable a feature to be placed
-   Execute MoLang within the current [feature context](#)

Scatter features attempt to place a [target feature](#proxy-features) with each iteration:

```json
"places_feature": "lostlands:shimmerfields_spire"
```

If, where, and how an instance of the target is placed depend on the [generation potential](#generation-potential), [distribution](#distribution), and the [evaluation order](#evaluation-order)

#### Generation Potential

```json
"scatter_chance": 25,
"iterations": 12
```

A scatter feature will determine placement attempts of its target using the `"scatter_chance"` and `"iterations"` properties.

**Scatter chance** represents the potential for the scatter feature to succeed. It can be represented as…

A numeric literal:

```json
"scatter_chance": 12.5
```

::: warning
The numeric literal form is considered against 100, not 1. A scatter chance of `50`, therefore, has half a chance of success.
:::

A MoLang expression:

```json
"scatter_chance": "1 / 8"
```

A fraction object:

```json
"scatter_chance": {
	"numerator": 1,
	"denominator": 8
}
```

All 3 examples have a 12.5% chance for success. Use whichever form feels most appropriate for your case. If scatter chance is omitted, it defaults to a 100% chance for the scatter feature to attempt to place its target.

**Iterations** are the number of attempts a scatter feature will try to place its target. If an instance of a scatter feature would succeed (in other words, if its scatter chance check were successful), _all_ of the iterations given by `"iterations"` will be attempted. Iterations may be represented as integer literals or MoLang expressions. Unlike scatter chance, iterations are required.

#### Distribution

```json
"x": {
	"distribution": "fixed_grid",
	"extent": [0, 15]
},
"z": {
	"distribution": "fixed_grid",
	"extent": [0, 15]
},
"y": 0
```

Distribution is predominantly handled using **coordinate properties**: `"x"`, `"z"`, and `"y"`. All these properties may be represented using…

An integer literal:

```json
"x": 0
```

A MoLang Expression:

```json
"x": "math.random_integer(0, v.surface_grass.spread - 1)"
```

Or a number of object forms for conveniently distributing a coordinate:

```json
"x": {
	"distribution": "uniform",
	"extent": [0, 16]
}
```

Literals and MoLang expressions are relative to the [feature origin](#). See [Distribution Types](#distribution-types) for the available pre-constructed distribution systems.

Because placement of features is so often relative to the heightmap, the incoming _y_-origin for the scatter feature may be **projected into the heightmap**:

```json
"project_input_to_floor": true
```

This means that the specified _y_-origin from the scatter feature’s parent is ignored in favor of the _y_-coordinate of the heightmap at an iteration’s _x_-_z_ location ([assuming the _y_-coordinate would be evaluated after the lateral coordinates](#evaluation-order)). The `"y"` property may still be given a value that will represent the offset from the heightmap.

::: tip Note
Functionally, this is the same as using the MoLang expression `"query.heightmap(v.worldx, v.worldz) + *offset*"`.
:::

##### Distribution Types

Custom distribution systems can be constructed using MoLang expressions, but scatter features come pre-equipped with a few common **distribution types** for convenient authoring:

-   Uniform
-   Gaussian
-   Inverse Gaussian
-   Fixed grid
-   Jittered grid

Each distribution type requires an **extent**, which represents the range of values on which that distribution operates, from minimum to maximum. Extents, like the basic forms of coordinate declarations, are relative to the [feature origin](#).

###### Uniform Distribution

```json
"z": {
	"distribution": "uniform",
	"extent": ["v.boulder_spread.start", "v.boulder_spread.end"]
}
```

**Uniform distribution** is uniformly random distribution on a half-open interval between two values. It is known as “uniform” because every value within the range has an equal chance of being selected and “half-open” because the extent minimum is a member of the range, while the extent maximum is not:

<i>minimum extent</i> <= <i>x</i> < <i>maximum extent</i>

Therefore, if an extent of `[0, 16]` were given for a uniform distribution, blocks may be placed in a range of size 16: from 0 to 15. The 1st possible position starts at 0 while the 15th possible position ends at 16, matching the extent.

###### Gaussian Distributions

```json
"y": {
	"distribution": "gaussian",
	"extent": [0, "2 * v.vine_cluster.radius"]
}
```

**Gaussian distribution** (`"gaussian"`) and its **inverse** (`"inverse_gaussian"`) are useful for grouping features together: toward or away from the center of the extent respectively. Gaussian distribution is so extreme that values will almost never be selected away from the center with normal Gaussian distribution or toward the center with inverse Gaussian distribution. The extents for Gaussian distribution behave the as with [uniform distribution](#uniform-distribution).

###### Grid Distributions

```json
"x": {
	"distribution": "jittered_grid",
	"extent": [0, 15],
	"step_size": 2,
	"grid_offset": 4
}
```

**Grid distributions** are powerful systems for placing blocks either directly on (`"fixed_grid"`) or randomly within (`"jittered_grid"`) evenly spaced intervals along a coordinate. Unlike the other distribution types, the extent of grids forms an interval that includes the maximum extent:

<i>minimum extent</i> <= <i>x</i> <= <i>maximum extent</i>

Two grid distribution-only properties are available for finer control over the grids used by these systems. The interval size, which defaults to 1, can be customized with the `"step_size"` property. An initial offset, defaulting to 0, can also be provided via the `"grid_offset"` property.

If the iteration count in conjunction with the step size and offset would push a coordinate beyond the maximum extent, the coordinate will be wrapped back to the minimum extent and continue from there.

While grids are useful on a coordinate independently, their true power shows when in combination with grid distributions on other coordinates. Placements prioritize incrementing the earliest evaluated grid system before later systems; the later layouts are only considered when wrapping occurs in the previous grid system. When a placement in an earlier evaluated coordinate would wrap, the next evaluated grid-powered coordinate will be offset by the number of wraps that occurred.

As a simple example:

```json
"iterations": 21,

"x": {
	"distribution": "fixed_grid",
	"extent": [0, 15]
},
"z": {
	"distribution": "fixed_grid",
	"extent": [0, 15]
}
```

Placements will first begin along _x_: (0, 0), (1, 0), etc., until reaching the end of the extent at (15, 0). However, only 16 of the 21 iterations have occurred; 5 remain. Now, the _x_-coordinate wraps back around to 0, while the _z_-coordinate increments to 1: (0, 1).

This wrapping occurs in three-dimensions, too, so when a plane along the earliest evaluated coordinates would wrap (assuming a high enough iteration count), another plane will begin formation based on the final coordinate’s step size.

::: warning
Grid extents behave in unexpected ways when 0 is not within the extent range. In particular, the extent will be projected into 0 by the closest bound. For example, the given extent `[-7, -2]` will be remapped as though it were `[-5, 0]`. `[13, 21]` would get remapped to `[0, 8]`. For this reason, it's recommended to just specify the extent as a length, such as using `[0, 8]` in the latter example, and proxying that scatter feature with another scatter feature that would position that coordinate using a literal `13`.

When using multiple grid distributions to form a surface or volume, extents for any coordinate that would extend below `0` will only work on the first pass of that coordinate. Later passes will be restricted to non-negative values.
:::

#### Evaluation Order

When a scatter chance is included, it is evaluated before any other properties. If the check against scatter chance fails for that instance of the scatter feature, nothing downstream within that instance is evaluated. No further MoLang is interpreted; no variables within the [feature context](#) are updated. The target feature is entirely disregarded.

Next, the iteration count is evaluated. Similarly to scatter chance, if the iteration count were not to resolve to a positive number of placement attempts, nothing further is evaluated.

Next, every iteration is attempted regardless of whether an early iteration would for some reason fail. For each iteration, each coordinate is evaluated using the same ordering across all iterations.

::: warning
The order of coordinate evaluation is _not_ dependent upon the order the coordinate properties are declared in JSON.
:::

By default, the coordinate ordering is _x_ then _z_ then _y_. This covers the majority of use-cases: if the coordinates aren’t independent, more than likely the vertical position depends on the lateral coordinates. However, scatter features may declare an atypical **coordinate evaluation order** for full control over coordinate dependence:

```json
"coordinate_eval_order": "zyx"
```

After the coordinates for an iteration have been determined, world generation moves its focus within the target: acknowledging its restrictions, attempting its placement, evaluating its MoLang, and (if possible and relevant) continuing by placing its children and their children and so forth.

When finished with the target’s feature tree, if more iterations have yet to be run from the scatter feature, focus returns to the scatter feature beginning with the first-evaluated coordinate and execution is resumed.

### Conditional Lists

```json
{
	"format_version": "1.13.0",

	"minecraft:conditional_list": {
		"description": {
			"identifier": "olympus:columns_selection"
		},

		"conditional_features": [
			{
				"places_feature": "olympus:columns_unweathered",
				"condition": "query.noise(v.originx, v.originz) < 0"
			},
			{
				"places_feature": "olympus:columns_weathered",
				"condition": "query.noise(v.originx, v.originz) >= 0"
			}
		],

		"early_out_scheme": "placement_success"
	}
}
```

**Conditional lists** pick a single feature from a collection based on conditions; they are akin to if-else if blocks in programming languages. Once a condition has been evaluated as successful (as determined via [success determination](#success-determination), the conditional list will select _only that one feature_ for placement.

::: tip Note
Instead, if _every_ success should place a feature in the same location, use an [aggregate feature](#aggregate-features) pointing to [scatter features](#scatter-features) that proxy the target features.
:::

#### Conditions List

```json
"conditional_features": [
	{
		"places_feature": "summer_fun:beachadjustment_water",
		"condition": "query.heightmap(v.originx, v.originz) < 63 && query.noise(v.originx, v.originz) < 0"
	},
	{
		"places_feature": "summer_fun:beachadjustment_coral",
		"condition": "query.heightmap(v.originx, v.originz) < 63 && query.noise(v.originx, v.originz) >= 0"
	},
	{
		"places_feature": "summer_fun:beachadjustment_air",
		"condition": "query.noise(v.originx, v.originz) >= 63"
	}
]
```

The **conditions list**, `"conditional_features"`, is an ordered array comprised of **feature entries** objects. Feature entries bind [target features](#proxy-features) to their **conditions**:

```json
{
	"places_feature": "verona:evergreen_trees_stumps",
	"condition": "v.evergreen_forest.type == v.evergreen_forest.types.lumberjack_ruined"
}
```

The condition of each feature entry is evaluated by entry order in the conditions list. Once a feature entry [would succeed](#success-determination), no later-listed conditions will be evaluated.

#### Success Determination

```json
"early_out_scheme": "placement_success"
```

Condition success is considered in light of the optional **early out scheme**. Two mechanisms are provided for controlling if a feature entry would succeed. `"condition_success"` — the default if no `"early_out_scheme"` is provided — considers a success to occur when a condition evaluates to true. `"placement_success"` goes further: a condition must evaluate to true, and its target feature’s placement must succeed.

### Aggregate Features

### Sequence Features

### Snap-to-Surface Features

### Search Features

### Rect Layouts

### Scan Surface Features

### Weighted Random Features

## Scene Features

### Geode Features

### Beards and Shavers

### Vegetation Patches

## Carver Features

### Cave Carver Features

### Hell Cave Carver Features

### Underwater Cave Carver Features
