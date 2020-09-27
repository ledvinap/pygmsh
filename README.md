<p align="center">
  <a href="https://github.com/nschloe/pygmsh"><img alt="pygmsh" src="https://nschloe.github.io/pygmsh/logo-with-text.svg" width="60%"></a>
  <p align="center">Gmsh for Python.</p>
</p>

[![PyPi Version](https://img.shields.io/pypi/v/pygmsh.svg?style=flat-square)](https://pypi.org/project/pygmsh)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/pygmsh.svg?style=flat-square)](https://pypi.org/pypi/pygmsh/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1173105.svg?style=flat-square)](https://doi.org/10.5281/zenodo.1173105)
[![GitHub stars](https://img.shields.io/github/stars/nschloe/pygmsh.svg?style=flat-square&logo=github&label=Stars&logoColor=white)](https://github.com/nschloe/pygmsh)
[![PyPi downloads](https://img.shields.io/pypi/dm/pygmsh.svg?style=flat-square)](https://pypistats.org/packages/pygmsh)

[![Documentation Status](https://readthedocs.org/projects/pygmsh/badge/?version=latest&style=flat-square)](https://pygmsh.readthedocs.org/en/latest/?badge=latest)
[![Slack](https://img.shields.io/static/v1?logo=slack&label=chat&message=on%20slack&color=4a154b&style=flat-square)](https://join.slack.com/t/nschloe/shared_invite/zt-cofhrwm8-BgdrXAtVkOjnDmADROKD7A
)

[![gh-actions](https://img.shields.io/github/workflow/status/nschloe/pygmsh/ci?style=flat-square)](https://github.com/nschloe/pygmsh/actions?query=workflow%3Aci)
[![codecov](https://img.shields.io/codecov/c/github/nschloe/pygmsh.svg?style=flat-square)](https://codecov.io/gh/nschloe/pygmsh)
[![LGTM](https://img.shields.io/lgtm/grade/python/github/nschloe/pygmsh.svg?style=flat-square)](https://lgtm.com/projects/g/nschloe/pygmsh)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square)](https://github.com/psf/black)

pygmsh combines the power of [Gmsh](https://gmsh.info/) with the versatility of Python.
It provides useful abstractions from Gmsh's own Python interface so you can create
complex geometries more easily.

This document and the directory
[`test/`](https://github.com/nschloe/pygmsh/tree/master/test/) contain many small
examples. See [here](https://pygmsh.readthedocs.io/en/latest/index.html) for the full
documentation.

#### Flat shapes

<img src="https://nschloe.github.io/pygmsh/polygon.svg" width="100%"> | <img src="https://nschloe.github.io/pygmsh/circle.svg" width="100%"> | <img src="https://nschloe.github.io/pygmsh/splines.svg" width="100%">
:------------------:|:-------------:|:-------------:|
Polygon             |  Circle       |  (B-)Splines  |

Codes:
```python
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    geom.add_polygon(
        [
            [0.0, 0.0],
            [1.0, -0.2],
            [1.1, 1.2],
            [0.1, 0.7],
        ],
        mesh_size=0.1,
    )
    mesh = pygmsh.generate_mesh(geom)

# mesh.points, mesh.cells, ...
# mesh.write("out.vtk")
```
```python
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    geom.add_circle([0.0, 0.0], 1.0, mesh_size=0.2)
    mesh = pygmsh.generate_mesh(geom)
```
```python
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    lcar = 0.1
    p1 = geom.add_point([0.0, 0.0, 0.0], lcar)
    p2 = geom.add_point([1.0, 0.0, 0.0], lcar)
    p3 = geom.add_point([1.0, 0.5, 0.0], lcar)
    p4 = geom.add_point([1.0, 1.0, 0.0], lcar)
    s1 = geom.add_bspline([p1, p2, p3, p4])

    p2 = geom.add_point([0.0, 1.0, 0.0], lcar)
    p3 = geom.add_point([0.5, 1.0, 0.0], lcar)
    s2 = geom.add_spline([p4, p3, p2, p1])

    ll = geom.add_curve_loop([s1, s2])
    pl = geom.add_plane_surface(ll)

    mesh = pygmsh.generate_mesh(geom)
```

The return value is always a [meshio](https://pypi.org/project/meshio) mesh, so to store
it to a file you can
<!--exdown-skip-->
```python
mesh.write("test.vtk")
```
The output file can be visualized with various tools, e.g.,
[ParaView](https://www.paraview.org/).

#### Extrusions

<img src="https://nschloe.github.io/pygmsh/extrude.png" width="100%"> | <img src="https://nschloe.github.io/pygmsh/revolve.png" width="100%"> | <img src="https://nschloe.github.io/pygmsh/twist.png" width="100%">
:------------------:|:-------------:|:--------:|
`extrude`           |  `revolve`    |  `twist` |

```python
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    poly = geom.add_polygon(
        [
            [0.0, 0.0],
            [1.0, -0.2],
            [1.1, 1.2],
            [0.1, 0.7],
        ],
        mesh_size=0.1,
    )
    geom.extrude(poly, [0.0, 0.3, 1.0], num_layers=5)
    mesh = pygmsh.generate_mesh(geom)
```
```python
from math import pi
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    poly = geom.add_polygon(
        [
            [0.0, 0.2, 0.0],
            [0.0, 1.2, 0.0],
            [0.0, 1.2, 1.0],
        ],
        mesh_size=0.1,
    )
    geom.revolve(poly, [0.0, 0.0, 1.0], [0.0, 0.0, 0.0], 0.8 * pi)
    mesh = pygmsh.generate_mesh(geom)
```
```python
from math import pi
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    poly = geom.add_polygon(
        [
            [+0.0, +0.5],
            [-0.1, +0.1],
            [-0.5, +0.0],
            [-0.1, -0.1],
            [+0.0, -0.5],
            [+0.1, -0.1],
            [+0.5, +0.0],
            [+0.1, +0.1],
        ],
        mesh_size=0.05,
    )

    geom.twist(
        poly,
        translation_axis=[0, 0, 1],
        rotation_axis=[0, 0, 1],
        point_on_axis=[0, 0, 0],
        angle=pi / 3,
    )

    mesh = pygmsh.generate_mesh(geom)
```

#### Mesh refinement/boundary layers
<img src="https://nschloe.github.io/pygmsh/boundary0.svg" width="100%"> | <img src="https://nschloe.github.io/pygmsh/revolve.png" width="100%"> |
:------------------:|:-------------:|:--------:|

```python
import pygmsh

with pygmsh.built_in.Geometry() as geom:
    poly = geom.add_polygon(
        [
            [0.0, 0.0, 0.0],
            [2.0, 0.0, 0.0],
            [3.0, 1.0, 0.0],
            [1.0, 2.0, 0.0],
            [0.0, 1.0, 0.0],
        ],
        mesh_size=0.3,
    )

    field0 = geom.add_boundary_layer(
        edges_list=[poly.curves[0]],
        lcmin=0.05,
        lcmax=0.2,
        distmin=0.0,
        distmax=0.2,
    )
    field1 = geom.add_boundary_layer(
        nodes_list=[poly.points[2]],
        lcmin=0.05,
        lcmax=0.2,
        distmin=0.1,
        distmax=0.4,
    )
    geom.set_background_mesh([field0, field1], operator="Min")

    mesh = pygmsh.generate_mesh(geom)
```

#### OpenCASCADE
![](https://nschloe.github.io/pygmsh/puzzle.png)

As of version 3.0, Gmsh supports OpenCASCADE, allowing for a CAD-style geometry
specification.

Example:
```python
import pygmsh

with pygmsh.opencascade.Geometry() as geom:
    geom.characteristic_length_min = 0.1
    geom.characteristic_length_max = 0.1

    rectangle = geom.add_rectangle([-1.0, -1.0, 0.0], 2.0, 2.0)
    disk1 = geom.add_disk([-1.2, 0.0, 0.0], 0.5)
    disk2 = geom.add_disk([+1.2, 0.0, 0.0], 0.5)

    disk3 = geom.add_disk([0.0, -0.9, 0.0], 0.5)
    disk4 = geom.add_disk([0.0, +0.9, 0.0], 0.5)
    flat = geom.boolean_difference(
        geom.boolean_union([rectangle, disk1, disk2]),
        geom.boolean_union([disk3, disk4]),
    )

    geom.extrude(flat, [0, 0, 0.3])

    mesh = pygmsh.generate_mesh(geom)
```

### Installation

pygmsh is [available from the Python Package Index](https://pypi.org/project/pygmsh/),
so simply do
```
pip install pygmsh
```
to install. Also, make sure to have [gmsh](http://gmsh.info/) installed.

### Usage
Just
```
import pygmsh
```
and make use of all the goodies the module provides. The
[documentation](https://pygmsh.readthedocs.org/) and the examples under
[`test/`](https://github.com/nschloe/pygmsh/tree/master/test/) might inspire you.


### Testing

To run the pygmsh unit tests, check out this repository and type
```
pytest
```

### Building Documentation

Docs are built using [Sphinx](http://www.sphinx-doc.org/en/stable/).

To build run
```
sphinx-build -b html doc doc/_build
```

### License

This software is published under the [GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.en.html).
