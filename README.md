simplicial-complex
==================

This CommonJS module implements basic topological operations and indexing for [abstract simplicial complexes](http://en.wikipedia.org/wiki/Abstract_simplicial_complex) (ie graphs, triangular and tetrahedral meshes, etc.) in JavaScript.

A [simplicial complex](http://en.wikipedia.org/wiki/Abstract_simplicial_complex) is a higher dimensional generalization of the usual concept of a [(directed) graph](http://en.wikipedia.org/wiki/Graph_\(mathematics\)).  Recall that a graph is represented by a pair of sets called:

* [vertices](http://en.wikipedia.org/wiki/Vertex_\(graph_theory\)) : A finite collection of labels/indices.
* [edges](http://en.wikipedia.org/wiki/Edge_\(graph_theory\)) : A finite collection of ordered pairs of vertices.

In an oriented abstract simplicial complex, the edges of a graph are replaced by a higher dimensional concept general concept called the cells.  Again:

* [vertices](http://en.wikipedia.org/wiki/Vertex_\(graph_theory\)) : A finite collection of labels/indices.
* [cells](http://en.wikipedia.org/wiki/Simplex) : A finite collection of ordered [tuples](http://en.wikipedia.org/wiki/Tuple) of vertices.

We say that the dimension of each cell is just its length - 1.  For example:

* -1 - cells: The empty cell
* 0 - cells: Vertices
* 1 - cells: Edges
* 2 - cells: Faces/triangles
* 3 - cells: Volumes/tetrahedra
*  ...
* n-cells: n-simplices

You probably already know of many examples of simplicial complexes.  Triangular meshes (as commonly used in computer graphics) are an instance of simplicial complexes where each cell has exactly 3 elements.  A more restricted example of a simplicial complex is the notion of a [hypergraph](http://en.wikipedia.org/wiki/Hypergraph), which is basically what you get when you forget the ordering of each cell.

There are many ways to represent abstract simplicial complexes, but the two most basic forms are:

* (Adjacency List) A flat list of cells
* (Adjacency Polynomial) As a collection of tensors, K0, K1, ..., Kn, where Kn(v0, v2, ..., vn) = 1 if the simplex (v0, v1, ..., vn) is in the complex and 0 otherwise.

The first approach generalizes the [adjacency list](http://en.wikipedia.org/wiki/Adjacency_list) storage format for a graph, while the second form generalizes the [adjacency matrix](http://en.wikipedia.org/wiki/Adjacency_matrix).  In the first case, the storage scales linearly as O(v + d * c), while in the later case the storage scales as O(v^d).  When the total number of cells is very large and d is very small, adjacency matrix representations may be acceptable.  On the other hand, for large values of d adjacency lists scale far better.  As a result, we categorically adopt the first form as our representation.

Usage
=====

First, you need to install the library using [npm](https://npmjs.org/):

    npm install simplical-complex
    
And then in your scripts, you can just require it like usual:

    var top = require("simplicial-complex");

`simplicial-complex` represents cell complexes as arrays of arrays of vertex indices.  For example, here is a triangular mesh:

    var tris = [
        [0,1,2],
        [1,2,3],
        [2,3,4]
      ];
      
And here is how you would compute its edges:

    var edges = top.skeleton(tris, 1);
    
    //Result:
    //  edges = [ [0,1],
    //            [0,2],
    //            [1,2],
    //            [1,3],
    //            [2,3],
    //            [2,4],
    //            [3,4] ]

The functionality in this library can be broadly grouped into the following categories:

Generic
-------

### `dimension(cells)`
**Returns:** The dimension of the cell complex.

**Time complexity:** `O(cells.length)`

### `countVertices(cells)`
An optimized way to get the number of 0-cells in a cell complex with dense, sequentially indexed vertices.  If `cells` has these properties, then:

    top.countVertices(cells)
    
Is equivalent to:

    top.skeleton(cells, 0).length

* `cells` is a cell complex

**Returns:** The number of vertices in the cell complex

**Time complexity:**  `O(cells.length * d)`, where `d = dimension(cells)`

### `cloneCells(cells)`
Makes a copy of a cell complex

* `cells` is an array of cells

**Returns:** A deep copy of the cell complex

**Time complexity:** `O(cells.length * d)`

Indexing and Incidence
----------------------

### `compareCells(a, b)`
Ranks a pair of cells relative to one another up to permutation.

* `a` is a cell
* `b` is a cell

**Returns** a signed integer representing the relative rank:
* < 0 : `a` comes before `b`
* = 0 : `a = b` up to permutation
* > 0 : `b` comes before `a`

**Time complexity:** `O( a.length * log(a.length) )`

### `normalize(cells[, attr])`
Canonicalizes a cell complex so that it is possible to compute `findCell` queries.  Note that this function is done **in place**.  `cells` will be mutated.  If this is not acceptable, you should make a copy first using `cloneCells`.

* `cells` is a complex.
* `attr` is an optional array of per-cell properties which is permuted alongside `cells`

**Returns** `cells`

**Time complexity:** `O(d * log(cells.length) * cells.length )`

### `findCell(cells, c)`
Finds a lower bound on the first index of cell `c` in a `normalize`d array of cells.

* `cells` is a `normalize`'d array of cells
* `c` is a cell represented by an array of vertex indices

**Returns:** The index of `c` in the array if it exists, otherwise -1

**Time complexity:** `O(d * log(d) * log(cells.length))`, where `d` is the max of the dimension of `cells` and `c`

### `buildIndex(from_cells, to_cells)`
Builds an index for [neighborhood queries](http://en.wikipedia.org/wiki/Polygon_mesh#Summary_of_mesh_representation).  This allows you to quickly find cells the in `to_cells` which are incident to cells in `from_cells`.

* `from_cells` a `normalize`d array of cells
* `to_cells` a list of cells which we are going to query against

**Returns:** An array with the same length as `from_cells`, the `i`th entry of which is an array of indices into `to_cells` which are incident to `from_cells[i]`.

**Time complexity:** `O(from_cells.length + d * 2^d * log(from_cells.length) * to_cells.length)`, where `d = max(dimension(from_cells), dimension(to_cells))`.

Basic Topology
--------------

### `dual(cells[, vertex_count])`
Computes the [dual](http://en.wikipedia.org/wiki/Hypergraph#Incidence_matrix) of the complex.  An important application of this is that it gives a more optimized way to build an index for vertices for cell complexes with sequentially enumerated vertices.  For example,

    top.dual(cells)

Is equivalent to doing:

    top.buildIndex(cells, top.skeleton(cells, 0), 0)
    
* `cells` is a cell complex
* `vertex_count` is an optional parameter giving the number of vertices in the cell complex.  If not specified, then it calls `buildIndex(cells, skeleton(cells,0), 0))`

**Returns:** An array of elements with the same length as `vertex_count` (if specified) or `skeleton(cells,0)` otherwise giving the [vertex stars of the mesh](http://en.wikipedia.org/wiki/Star_(graph_theory\)) as indexed arrays of cells.

**Time complexity:** `O(d * cells.length)`

### `subcells(cells, n)`
Enumerates all n cells in the complex.

* `cells` is an array of cells
* `n` is the dimension of the cycles to compute

**Returns:**  A list of all cycles

**Time complexity:** `O(d^n * cells.length)`

### `skeleton(cells, n)`
Computes the [n-skeleton](http://en.wikipedia.org/wiki/N-skeleton) of an unoriented simplicial complex.  This is the set of all unique n-cells up to permutation.

* `cells` is a cell complex
* `n` is the dimension of the skeleton to compute.

**Returns:** A `normalize` array of all n-cells which are unique up to permutation.

**Example:**

* `skeleton(tris, 1)` returns all the edge in a triangular mesh
* `skeleton(tets, 2)` returns all the faces of a tetrahedral mesh
* `skeleton(cells, 0)` returns all the vertices of a mesh

**Time complexity:**  `O( n^d * cells.length )`, where d is the `dimension` of the cell complex

### `boundary(cells, n)`
Computes the <a href="http://en.wikipedia.org/wiki/Boundary_(topology)">d-dimensional boundary</a> of a cell complex.  For example, in a triangular mesh `boundary(tris, 1)` gives an array of all the boundary edges of the mesh; or `boundary(tets, 2)` gives an array of all boundary faces.  Algebraically, this is the same as evaluating the boundary operator in the Z/2 homology.

* `cells` is a cell complex.
* `n` is the dimension of the boundary we are computing.

**Returns:** A `normalize`d array of `n`-dimensional cells representing the boundary of the cell complex.

**Time complexity:** `O((d^n + log(cells.length)) * cells.length)`

### `connectedComponents(cells[, vertex_count])`
Splits a simplicial complex into its <a href="http://en.wikipedia.org/wiki/Connected_component_(topology)">connected components</a>.  If `vertex_count` is specified, we assume that the cell complex is dense -- or in other words the vertices of the cell complex is the set of integers [0, vertex_count).  This allows for a slightly more efficient implementation.  If unspecified, a more general but less efficient sparse algorithm is used.

* `cells` is an array of cells
* `vertex_count` (optional) is the result of calling `countVertices(cells)` or in other words is the total number of vertices.

**Returns:** An array of cell complexes, one per each connected component.  Note that these complexes are not normalized.

**Time complexity:**

* If `vertex_count` is specified:  `O(vertex_count + d^2 * cells.length)`
* If `vertex_count` is not specified: `O(d^3 * log(cells.length) * cells.length)`

Credits
=======
(c) 2013 Mikola Lysenko.  MIT License

