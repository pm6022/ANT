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
    version     2.0;
    format      ascii;
    class       volScalarField;
    object      p;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 2 -2 0 0 0 0];
```
dimensions here are $m^2/s^2$  because the pressure has been divided by the density:
![[Pasted image 20230430173025.png]]
```cpp
internalField   uniform 0;

boundaryField
{
    inflow
    {
        type            zeroGradient;
    }

    outflow
    {
        type            fixedValue;
        value           uniform 0;
    }
```
qui non  ho capito se è assoluto o meno, penso di si
```cpp
    wall
    {
        type            zeroGradient;
    }

    wedge1
    {
        type            wedge;
    }

    wedge2
    {
        type            wedge;
    }

    defaultFaces
    {
        type            empty;
    }
}

// ************************************************************************* //
```

Here there are boundary conditions on the pressure, that's because pisoFoam uses different equations in which the NS equations are subdivided, doing that there's the need of bc in this case.
![[Pasted image 20230430173949.png]]
here it is seen the reference to a pressure equation that need bc.

The zeroGradient  bc at the inlet is used to have compatibility with profile we assigned to the velocity field.
On the wall it means that the isobaric line is orthogonal to the wall, like in the first example we saw in comsol where such lines were parallel to each other and orthogonal to the walls.
![[Pasted image 20230430175041.png]]
On a forum he found a demostration: in certain conditions, you always get isobaric lines orthogonal to the wall where you apply the no slip condition
(in comsol you get this as a result, in this you use it as a compatibility condition and it was proved by someone)
