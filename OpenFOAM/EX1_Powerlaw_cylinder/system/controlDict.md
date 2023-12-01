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
    location    "system";
    object      controlDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

application     pisoFoam;

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         0.01;

deltaT          0.00001; // 0.001 unstable, 0.0001 stable but rubbish, 0.00001 good
```
This is important for restarting the simulation at a certain time, the dt is fixed and we have to make a convergence on that. This file is well documented [here](https://doc.cfd.direct/openfoam/user-guide-v6/controldict). As you see in the snippet below, we can choose which time to actually save once the simulation is over, just like in COMSOL.
```cpp
writeControl    runTime;

writeInterval   0.001;

purgeWrite      0;

writeFormat     ascii; // you could choose binary, it takes less space

writePrecision  6;

writeCompression off;

timeFormat      general;

timePrecision   6;

runTimeModifiable true;

// ************************************************************************* //
```