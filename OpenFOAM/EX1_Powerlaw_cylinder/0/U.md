```cpp
dimensions      [0 1 -1 0 0 0 0];

internalField   uniform (0 0 0);

boundaryField
{
    inflow
    {
        type            fixedValue;
        value           uniform (0.1 0 0);
    }
```
- dimensions keyword is for the units as we saw likewise in [constant/physicalProperties](obsidian://open?vault=ANT&file=OpenFOAM%2FEX1_Powerlaw_cylinder%2Fconstant%2FphysicalProperties)
- internalField keyword is the initial values of the field inside the domain, hence the fluid is stagnant everywhere at the initial timestep or as a first try for a steady state simulation
- inflow is the name we gave to a face in [system/blockMeshDict](obsidian://open?vault=ANT&file=OpenFOAM%2FEX1_Powerlaw_cylinder%2Fsystem%2FblockMeshDict)  , if you went back to that file you'll the type "patch", that is the one used for inlet and outlet, there are 4 types used in blockMeshDict:
	- patch
	- wall
	- wedge
	- symmetryPlane
- here the type is referring to the kind of boundary condition, these names are partially documented in the guide, some bc not documented are fans.
- value is uniform on the face, if you want a parabola you'll need find how to do that
```cpp
    outflow
    {
        type            zeroGradient;
    }
```
zeroGradient for the velocity literally means the following: $$\nabla v \cdot n = 0$$
that basicly means that the stream is parallel to the face (**nota bene: per me è sbagliato vuole solo dire che la velocità non cambia in prossimità della faccia sarebbe tipo una derivata direzionale rispetto alla normale della faccia**) 
```cpp
    wall
    {
        type            noSlip;
    }
```
the classic noslip condition, several years ago there was no noslip keyword so it was used the fixedValue one in the folowing way:
```cpp
wall
{
type fixedValue;
value uniform(0 0 0);
}
```

that's one openfoam understands we are dealing with a axisymmetric problem
```cpp
    wedge1
    {
        type            wedge;
    }

    wedge2
    {
        type            wedge;
    }
// do not remove the block below, it is not important for us but it is needed.
    defaultFaces
    {
        type            empty;
    }
}
```
