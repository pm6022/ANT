```cpp
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  10
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
FoamFile
{
    format      ascii;
    class       dictionary;
    object      blockMeshDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

convertToMeters 0.08;

nx_ 300;
ny_ 150;

vertices
(
    (0 0 0)
    (2 0 0)
    (2 0.999390827 -0.0348994967)
    (0 0.999390827 -0.0348994967)
    (2 0.999390827 0.0348994967)
    (0 0.999390827 0.0348994967)
);

blocks
(
    hex (0 1 2 3 0 1 4 5) ($nx_ $ny_ 1) simpleGrading (1 1 1)
);

edges
(
);

boundary
(
    top
     {
        type patch;
        faces
        (
            (0 3 5 0)
        );
    }
    bottom
    {
        type patch;
        faces
        (
            (1 2 4 1)
        );
    }
    outer
    {
        type wall;
        faces
        (            
            (3 2 4 5)           
        );
    }
    axis
    {
        type empty; // used to neglect the third dimension and solve in 2D??
        faces
        (
            (0 1 1 0)           
        );
    }
    front
    {
        type wedge;
        faces
        (
            (0 1 4 5)           
        );
    }
    
    back
    {
        type wedge;
        faces
        (
            (0 1 2 3)          
        );
    }
);

mergePatchPairs
(
);

// *********************************************************************** //
```