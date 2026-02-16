---
title: SGRID Conventions (v0.3)
---

!!! WARNING

    This is a fork of the SGRID conventions. See the [original website](https://sgrid.github.io/sgrid/) to read the conventions.

Following the success of the [UGRID conventions](https://github.com/ugrid-conventions/ugrid-conventions),
Bert Jagers created conventions for staggered data on structured grids that are consistent with the UGRID conventions.
We refer to these as the SGRID conventions, described below.

## Introduction

The [CF-conventions](http://cfconventions.org/) are widely used for storing and distributing environmental / earth sciences / climate data. The CF-conventions use a data perspective: every data value points to the latitude and longitude at which that value has been defined; the combination of latitude and longitude bounds and cell methods attributes can be used to define spatially averaged rather than point values.

This is all great for the distribution of (interpolated) data for general visualization and spatial data processing,
but it doesn't capture the relationship of the variables as computed by a numerical model (such as [Arakawa staggering](http://en.wikipedia.org/wiki/Arakawa_grids)). Many models use staggered grids (using finite differences, or finite volume approach) or use a finite element approach of which the correct meaning may not be captured easily by simple cell methods descriptors.
This becomes a problem if you don't want to just look at the big picture of the model results,
but also at the details at the grid resolution:

- What is the exact meaning of a flux on the output file in discrete terms?
- Can we verify the mass balance?
- Can the data be used for restarting the model?

Correctly handling the staggered data has always been a crucial element of model post-processing tools. In the UGRID conventions, we have defined the (unstructured) grid as a separate entity on the file which consists of nodes and connections of nodes defining edges, faces, and volumes.
For a structured (staggered) grid we are currently lacking a consistent convention. Although one could store structured grid data using UGRID conventions,
some fundamental aspects such as distinction between grid directions would be lost.

In this context we have created these lightweight SGRID conventions to define the core aspects of a structured staggered grid without trying to capture the details of finite element formulations.
This is an attempt to bring conventions for structured grids on par with those for unstructured grids.

## Conventions

Consistent with the UGRID conventions we use the following terms for points,
lines, and cells that make up a grid.

| Dimensionality | Name   | Comments                                                                                                                 |
| -------------- | ------ | ------------------------------------------------------------------------------------------------------------------------ |
| 0              | node   | A point, a coordinate pair or triplet: the most basic element of the topology (also known as "vertex").                  |
| 1              | edge   | A line or curve bounded by two nodes.                                                                                    |
| 2              | face   | A plane or surface enclosed by a set of edges. In a Cartesian 2D model, you might refer to this as a "cell" or "square". |
| 3              | volume | A volume enclosed by a set of faces.                                                                                     |

In the UGRID conventions the focus is on describing the topology of the mesh (connectivity of the nodes, edges, faces, and volumes as appropriate).
The topology of a structured grid is inherently defined;
the focus for the SGRID conventions is therefore on the numbering used.
Still we need to distinguish between 2D and 3D grids (1D conventions may be defined consistently).

### 2D grid

| Required topology attributes | Value                                                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------------------------------- |
| cf_role                      | grid_topology                                                                                         |
| topology_dimension           | 2                                                                                                     |
| node_dimensions              | node_dimension1 node_dimension2                                                                       |
| face_dimensions              | face*dimension1:node_dimension1 (padding:\_type1*) face*dimension2:node_dimension2 (padding:\_type2*) |

| Optional attributes | Default value                                                      |
| ------------------- | ------------------------------------------------------------------ |
| edge1_dimensions    | node*dimension1 face_dimension2:node_dimension2 (padding:\_type2*) |
| edge2_dimensions    | face*dimension1:node_dimension1 (padding:\_type1*) node_dimension2 |
| node_coordinates    |                                                                    |
| edge1_coordinates   |
| edge2_coordinates   |                                                                    |
| face_coordinate     |                                                                    |
| vertical_dimensions |                                                                    |
|                     |

where the padding type may be one of the four literal strings:
"none", "low", "high", or "both" depending on whether the face*dimension is one shorter than the corresponding node_dimension (padding:none),
one longer than the corresponding node_dimension (padding:both),
or of equal length with one extra value stored on the low or high end of the dimension (see the figure below).
The edge1_dimensions and edge2_dimensions attributes may be used to define separate dimensions for the edges (see the ROMS example below),
but by default the edge dimensions are assumed to be consistent with the dimensions used by the edges and faces respectively.
The optional vertical_dimensions attribute may be used to specify the names of the dimensions for the layers and layer interfaces respectively using the same syntax:
layer_dimension:layer_interface_dimension (padding:\_type*).

Note:

- The numbering of the edges corresponds to the order of the dimensions in the dimensions attributes.
  The order of the dimensions listed here does not change since these are specified as a string.
  The actual order of dimensions in the netCDF API (and hence in the dump generated by different tools) depends on the programming language used.
  Hence, the order of the dimensions listed here may differ from the order of the dimensions in the data definition.

![SGRID faces and nodes](images/sgrid_faces_nodes.png)
Figure 1: illustrating the formulations used for expressing the relationship between face/edge and node dimensions. Please note that the numbering of the faces and nodes can be adjusted using face(face) and node(node) coordinate variables.

Example:

```java
--8<-- "examples/2d-grid.cdl"
```

### 3D grid

| Required topology attributes | Value                                                                                                                                                    |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| cf_role                      | grid_topology                                                                                                                                            |
| topology_dimension           | 3                                                                                                                                                        |
| node_dimensions              | node_dimension1 node_dimension2 node_dimension3                                                                                                          |
| volume_dimensions            | face*dimension1:node_dimension1 (padding:\_type1*) face*dimension2:node_dimension2 (padding:\_type2*) face*dimension3:node_dimension3 (padding:\_type3*) |

| Optional attributes   | Default value                                                                                                         |
| --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| edge1_dimensions      | face*dimension1:node_dimension1 (padding:\_type1*) node_dimension2 node_dimension3                                    |
| edge2_dimensions      | node*dimension1 face_dimension2:node_dimension2 (padding:\_type2*) node_dimension3                                    |
| edge3_dimensions      | node*dimension1 node_dimension2 face_dimension3:node_dimension3 (padding:\_type3*)                                    |
| face1_dimensions      | node*dimension1 face_dimension2:node_dimension2 (padding:\_type2*) face*dimension3:node_dimension3 (padding:\_type3*) |
| face2_dimensions      | face*dimension1:node_dimension1 (padding:\_type1*) node*dimension2 face_dimension3:node_dimension3 (padding:\_type3*) |
| face3_dimensions      | face*dimension1:node_dimension1 (padding:\_type1*) face*dimension2:node_dimension2 (padding:\_type2*) node_dimension3 |
| node_coordinates      |                                                                                                                       |
| edge _i_\_coordinates |                                                                                                                       |
| face _i_\_coordinates |                                                                                                                       |
| volume_coordinates    |                                                                                                                       |

Notes:

- The edge1, edge2, and edge3 are in a 3D grid aligned to the dimensions1, 2, and 3 respectively,
  whereas the edge1 and edge2 are in a 2D grid perpendicular to the dimensions 1 and 2 respectively.
  The face1, face2, and face3 play that role in the 3D grid.
- The 3d grid option should not be used be used for layered grids,
  such as typical ocean and atmosphere models.
  Use the 2d grid with vertical dimensions instead.
  This allows 2D quantities (such as surface quantities) and 3D quantities to be linked to the same mesh.

Example:

```java
--8<-- "examples/3d-grid.cdl"
```

## Data variables

The use of the attributes to associate a data variable with a specific grid and stagger location is copied from the UGRID conventions:
To map the variable onto the topology of the underlying grid,
two new attributes have been introduced.
First, the attribute `grid` points to the grid_topology variable containing the meta-data attributes of the grid on which the variable has been defined.
Second, the attribute `location` points to the (stagger) location within the grid at which the variable is defined.

Example:

```java
double waterlevel(time,j,i) ;
        waterlevel:standard_name = "sea_surface_height_above_geoid" ;
        waterlevel:units = "m" ;
        waterlevel:grid = "MyGrid"
        waterlevel:location = "face" ;
        waterlevel:coordinates = "lat_face_MyGrid lon_face_MyGrid" ;
```

## Examples

### Delft3D

Delft3D uses an Arakawa C-grid with the water level (pressure) computed in the cell centres,
and the normal velocities at the cell edges.
This example shows the use of asymmetric padding (at the low end of the horizontal coordinate indices there is an extra line of face/mid-point values).
In the vertical there is no padding, so the number of layer interfaces is one more than the number of layers.
The integer coodinate variables KMAX and KMAX1 are used to indicate that layer interfaces are numbered 0 to KMAX whereas all other indices use the default numbering from 1 to the maximum value.

```java
--8<-- "examples/delft3d.cdl"
```

The edge_dimension attributes are not needed.

### ROMS

ROMS uses also a C-grid, but it uses on the output file different dimensions for each staggered location.
In this case we need all attributes defined above including the edge*i*\_dimension attributes.

```java
--8<-- "examples/roms.cdl"
```

### WRF (ARW version)

The WRF-ARW also uses a C-grid.
Again the model results can best be captured by a 2D grid.
It might be interesting to verify the result for WRF-NMM since that model uses an E-grid,
but I couldn't find an example file.

```java
--8<-- "examples/wrf.cdl"
```
