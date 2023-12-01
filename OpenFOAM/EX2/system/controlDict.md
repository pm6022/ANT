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

application     interFoam;

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         1.0;

deltaT          1e-05;

writeControl    adjustableRunTime;

writeInterval   5e-03;

purgeWrite      0;

writeFormat     binary;

writePrecision  6;

writeCompression off;

timeFormat      general;

timePrecision   6;

runTimeModifiable yes;

adjustTimeStep  yes;

maxCo           0.1;
maxAlphaCo      0.1;

maxDeltaT       1e-03;

functions

{
        InfoBubble

    {
        functionObjectLibs ("libutilityFunctionObjects.so");
        enabled         true;
        type            coded;
        name            InfoBubble;
        executeControl  runTime;
        executeInterval 1e-04;
        writeControl    runTime;
        writeInterval   1e-04;
        writeFields     true;

        codeOptions
        #{
            -I$(LIB_SRC)/meshTools/lnInclude
        #};

        codeExecute
        #{

            const volScalarField& alpha1 = mesh().lookupObject<volScalarField>("alpha.water");
            const volVectorField& C = mesh().C();
            const volVectorField& U = mesh().lookupObject<volVectorField>("U");
            const scalarField& volume = mesh().V();
            const uniformDimensionedVectorField& gravity = mesh().lookupObject<uniformDimensionedVectorField>("g");

            scalar alphadV = 0.0; //Integral of alphadV
	    scalar AR      = 0.0;
            scalar xcMpos  = 0.0; // Weighted position x, integral of alpha*x*dV
            scalar ycMpos  = 0.0; // Weighted position y, integral of alpha*x*dV
            scalar vxcMpos = 0.0; // Weighted velocity along x, integral of alpha*vx*dV
            scalar vycMpos = 0.0; // Weighted velocity along y, integral of alpha*vy*dV

            scalar xB 	   = 0.0; // X coordinate of the center of mass
            scalar yB      = 0.0; // Y coordinate of the center of mass
            scalar vxB     = 0.0; // X coordinate of the centroid velocity 
            scalar vyB     = 0.0; // Y coordinate of the centroid velocity 

            volScalarField DD_motion(alpha1);
            DD_motion     *= scalar(0);

            volScalarField DD_perp(alpha1);
            DD_perp       *= scalar(0);

            volScalarField DD_motion_max(alpha1);
            DD_motion_max *= scalar(0);

            volScalarField DD_motion_min(alpha1);
            DD_motion_min *= scalar(1000);

            // Scan all the cells and compute the summation of both the volume and the centroid

                forAll(C, cellI)

                {

                  alphadV += (1-alpha1[cellI]) * volume[cellI] ;
                  xcMpos  += (1-alpha1[cellI]) * volume[cellI] * C[cellI].component(0);
                  ycMpos  += (1-alpha1[cellI]) * volume[cellI] * C[cellI].component(1);
                  vxcMpos += (1-alpha1[cellI]) * volume[cellI] * U[cellI].component(0);
                  vycMpos += (1-alpha1[cellI]) * volume[cellI] * U[cellI].component(1);

                }

                // To parallelize the computation

                reduce(xcMpos,sumOp<scalar>());
                reduce(ycMpos,sumOp<scalar>());
                reduce(vxcMpos,sumOp<scalar>());
                reduce(vycMpos,sumOp<scalar>());
                reduce(alphadV,sumOp<scalar>());

                xB  = xcMpos / max(alphadV, SMALL); //Compute the x center of mass
                yB  = ycMpos / max(alphadV, SMALL); //Compute the y center of mass
                vxB = fabs(vxcMpos) / max(alphadV, SMALL); // Compute the vx of the bubble
                vyB = fabs(vycMpos) / max(alphadV, SMALL); // Compute the vx of the bubble

                forAll(C, cellI)

                {

                        if (alpha1[cellI] < scalar(0.5))

                        {

                            if (gravity.component(0).value() < 0) // if the bubble rises in X

                            {
                                DD_motion_min[cellI] = C[cellI].component(0);
                                DD_motion_max[cellI] = C[cellI].component(0);
                                DD_perp[cellI]       =  C[cellI].component(1);
                            }

                            else // if the bubble rises in Y

                            {

                                DD_motion_min[cellI] = C[cellI].component(1);
                                DD_motion_max[cellI] = C[cellI].component(1);
                                DD_perp[cellI]       =  C[cellI].component(0);
                            }

                        }

                }

                AR = (gMax(DD_motion_max)-gMin(DD_motion_min))/(2*gMax(DD_perp)+VSMALL); //Aspect Ratio

                //Writing section

                if(Pstream::master())

                {
                    std::ofstream comOutput;

                    comOutput.open("DataBubble.dat", std::ofstream::app);
                    comOutput << mesh().time().value() <<"\t" << vxB << "\t" << vyB << "\t" << xB << "\t" << yB << "\t" << AR << "\n";
                    comOutput.close();

                }
          #};

    }
}

// ************************************************************************* //
```