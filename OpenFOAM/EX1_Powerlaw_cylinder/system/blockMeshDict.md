
## FoamFile
```cpp
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  8
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      blockMeshDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

convertToMeters 1;
```
In OpenFOAM, the code is organized into "blocks," which are sections of code that define a particular aspect of the simulation.

In this code snippet, the first block is the `FoamFile` block, which specifies the version, format, class, and object of the input file.

The second block is the `convertToMeters` keyword, which specifies the scale of the mesh.

## vertices
And the third block is the `vertices` block, which defines the coordinates of the vertices that define the boundary of the fluid domain.
```cpp
vertices
(
    (0 0 0)
    (0.1 0 0)
    (0.1 0.00999 -0.000436)
    (0 0.00999 -0.000436)
    (0 0 0)
    (0.1 0 0)
    (0.1 0.00999 0.000436)
    (0 0.00999 0.000436)
);
```

These vertices can be arranged in any order but you actually have to mind the convention represented in the figure below, the actual order will be specified in the following block. 

Starting from the 0 you have to enumerate until 3 in a counterclockwise direction with respect to the z axis (just like the right-hand rule, rotating the x axis on the y axis). Then in the z direction do the same starting from 4.

![[Pasted image 20230422170906.png | center ]]


## blocks
```cpp
blocks
(
    hex (0 1 2 3 4 5 6 7) (100 15 1) simpleGrading (1 1 1)
);
```
Inside this block, the `hex` keyword defines the type of element to use, the sequence of 8 number is refered to the vertices defined in the `vertices` block, it has to been such as it follows the convention so the order is important this time. The triplet nearby represents the number of segments in which the 3 sides (x y z) of the cell are divided, once defined, the others are attained because the mesh costruction is similar to the mapped one's.

The keyword `simpleGrading` is used to define a trend on the segments' size, in particular the triplet are the ratio between the last and the first segment and trend between is linear, in the documentation there are other keywords to use different trends or even a list for a custom sequence.   

## boundary
```cpp
edges
(
);
// per ora non serve a niente
boundary
(
    inflow
    {
        type patch;
        faces
        (
            (0 4 7 3)
        );
    }
    outflow
    {
        type patch;
        faces
        (
            (1 5 6 2)
        );
    }
    wall
    {
        type wall;
        faces
        (
            (3 7 6 2)
        );
    }
    wedge1
    {
        type wedge;
        faces
        (
            (4 5 6 7)
        );
    }
    wedge2
    {
        type wedge;
        faces
        (
            (0 1 2 3)
        );
    }
);
```
This block somewhat resembles clicking the boundaries on comsol's drawing, names of the sub-blocks are arbitrary but must be the same as the files we will write the boundaries in.

The `type` keyword points out the kind of boundary condition to be applied for the surfaces included in the following faces block.

In the `faces` block you have to mind avoiding entanglements between the vertices constructing the faces! The sequence must form a convex area.

## mergePatchPairs
```cpp
mergePatchPairs
(
);
```

This block is compulsory, it takes care of  surfaces where the elements are in contact but have different sizes. In this example is blank because there is no need for it.