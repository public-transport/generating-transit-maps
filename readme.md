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

@derhuerst wrote [`generate-vbb-graph`](https://github.com/derhuerst/generate-vbb-graph) and [`generate-vbb-graph`](https://github.com/derhuerst/generate-vbb-graph) for this. Both of them generate data in the [JSON Graph Format](https://github.com/jsongraph/json-graph-specification/blob/master/README.rst#json-graph-specification).

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

TODO

## rendering the map

[`generate-vbb-transit-map`](https://github.com/derhuerst/generate-vbb-transit-map) will take the optimized graph as JSON, add line colors and generate an [SVG file](https://developer.mozilla.org/en-US/docs/Web/SVG).

TODO: map picture

## improvements & enhancements to be implemented

- handle very large graphs (Germany would be ~500k million edges)
- find a way to avoid intersecting lines
- interactive visualisations
	- hide less important lines to reduce noise, show them on zoom
	- route planning, similar to [vbb-map-routing](https://github.com/derhuerst/vbb-map-routing)
