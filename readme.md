# generating transit maps

> A transit map is a topological map in the form of a schematic diagram used to illustrate the routes and stations within a public transport system—whether this be bus lines, tramways, rapid transit, commuter rail or ferry routes. […]

> Their primary function is to help users to efficiently use the public transport system, including which stations function as interchange between lines. Unlike conventional maps, transit maps are usually not geographically accurate—instead they use straight lines and fixed angles, and often illustrate a fixed distance between stations, compressing those in the outer area of the system and expanding those close to the center.

[*Transit map* on Wikipedia](https://en.wikipedia.org/wiki/Transit_map)

To automatically generate a transit map of a public transport network, the following information is necessary:

- a list of all stations, their geographical position and their names
- a list of all lines, their order of stations and their colors
- a distance metric between stations, e.g. travel time or physical distance

One can build a [graph structure](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)) from this data and feed it into optimization tools adjusted to this specific problem.

@dirkschumacher, @juliuste and @derhuerst set out to do this with [Berlin & Brandenburg public transport](https://en.wikipedia.org/wiki/Verkehrsverbund_Berlin-Brandenburg) (VBB) and [German national railways *Deutsche Bahn*](https://en.wikipedia.org/wiki/Deutsche_Bahn) (DB).

## generating a graph

@derhuerst wrote [`generate-vbb-graph`](https://github.com/derhuerst/generate-vbb-graph) and [`generate-db-graph`](https://github.com/derhuerst/generate-db-graph) for this. Both of them generate data in the [JSON Graph Format](https://github.com/jsongraph/json-graph-specification/blob/master/README.rst#json-graph-specification).

They will create two files, `nodes.ndjson` and `edges.ndjson`, which [ndjson](http://ndjson.org)-encoded lists of all nodes and edges, respectively. A node from `nodes.ndjson` looks like this:

```json
{
	"id": "900000029101",
	"label": "S Spandau",
	"metadata": {
		"x": 536.66,
		"y": 326.25
	}
}
```

An edge from `edges.ndjson` looks like this:

```json
{
	"source": "900000100001",
	"target": "900000003201",
	"relation": "regional",
	"metadata": {
		"line": "RB22",
		"time": 180
	}
}
```

## optimizing the graph

Generating a transit map based on the geographical representation of the network is a computationally [(NP-)hard task](http://www1.pub.informatik.uni-wuerzburg.de/pub/wolff/pub/nw-dlhqm-10.pdf). Given the input graph we would like to find the best representation of it as a transit map, so it is natural to consider this an optimization problem. Some methods have been developed over the years, two of which we implemented / started to investigate. 

One method that uses mixed-integer linear programming by [Martin Nöllenburg and Alexander Wolff](http://www1.pub.informatik.uni-wuerzburg.de/pub/wolff/pub/nw-dlhqm-10.pdf) to solve this task was implemented by us in [julia](https://github.com/dirkschumacher/TransitmapSolver.jl). It is still work in progress, as automated labeling is not yet supported, but already yields good results using the open source solver [COIN-CBC](https://github.com/JuliaOpt/Cbc.jl) for average transit size subway-networks. Also this paper gives a nice overview of the problem in general. Note that mixed-integer linear programming aims at finding a (provable) optimal solution: however an optimal solution in the mathematical sense might not be the most pleasent looking transit map. 

Related to the problem of embedding a graph as a transit map is to decide which order parallel edges should have, so that line crossings at intersections are minimized. This is something we have only started to look into. However it seems that this problem can also be formulated as a mixed-integer program. Some [first ideas](https://github.com/dirkschumacher/LineFlowSolver.jl) have been implemented in julia as well.

## rendering the map

[`generate-vbb-transit-map`](https://github.com/derhuerst/generate-vbb-transit-map) will take the optimized graph as JSON, add line colors and generate an [SVG file](https://developer.mozilla.org/en-US/docs/Web/SVG).

Here's a preliminary map for Berlin subway:

![Berlin subway generated transit map](https://cdn.rawgit.com/public-transport/generating-transit-maps/master/berlin-subway.svg)

## improvements & enhancements to be implemented

- handle very large graphs (Germany would be ~500k edges)
- find a way to avoid intersecting lines
- add support for node labeling
- interactive visualisations
	- hide less important lines to reduce noise, show them on zoom
	- route planning, similar to [vbb-map-routing](https://github.com/derhuerst/vbb-map-routing)
