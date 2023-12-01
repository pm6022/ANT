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
    class       volVectorField;
    location    "0";
    object      U;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 1 -1 0 0 0 0];

internalField   uniform (0 0 0);

boundaryField
{
    top
    {
        type        zeroGradient;
    }
    
    bottom
    {
        type		noSlip;
    }
    
    front
    {
        type        wedge;
    }
    
    back
    {
        type        wedge;
    }
    
    outer
    {
        type        noSlip; // using slip you can simulate the unconfined case, by slipping perfectly there is no effect of the wall presence on the system!
    }

}


// ************************************************************************* //
```