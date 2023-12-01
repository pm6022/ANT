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
    object      setFieldsDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

R_ 0.008;
C_ #calc"5*$R_";

defaultFieldValues
(
    volScalarFieldValue alpha.water 1 //alpha is the volumetric fraction
);

regions
(
    sphereToCell
    {
    	centre ($C_ 0 0);
	radius $R_;

        fieldValues
        (
            volScalarFieldValue alpha.water 0 
        );
    }

);


// ************************************************************************* //
```