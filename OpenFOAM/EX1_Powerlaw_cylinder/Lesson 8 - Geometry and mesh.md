
At this point it should be clear the steps we follow to run a simulation:
1) drawing the computational domain
2) building the mesh
3) start the solver

OpenFOAM uses Volume Of Fluid numerical method, whereas COMSOL exploits the Finite Element Method, but at the end of day the steps are the same.

OpenFOAM owns two Mesh Generation Utilities:
1) blockMesh
2) snappyHexMesh

At first we'll see blockMesh, it generates meshes made out of hexahedral blocks, it is similar to the mapped method we saw in COMSOL, in the same way it can't be used to directly discretize certain geometries like a pyramid or a cylinder having a spherical cavity, we first have to divide the domain in subdomains.

The good news is that finite volume method works quite good when hexahedral are used.

Below there are depicted several way to use blockMesh when there is curvature or you want to fine tune the mesh, exploiting multiple domains, we will see how to it. 
![[Pasted image 20230426124152.png|200]]
To subdivide the second image in domains there's a keyword to make faces having curvature.The third image has two different element widths and are merged thanks to a non-hexahedral transition zone.
![[Pasted image 20230426070227.png]]

On the other hand snappyHexMesh is able to generate an higher number of elements and makes use of different geometries to fill up the parts of the domain where hexahedral blocks can't be drawn.

There are other utilities called wrappers needed to convert mesh done with the help of other programs, one of these is fluentToFoam, that is able to translate meshes built with Fluent.

Unfortunately there is no comsolToFoam utility but there's a workflow we can exploit to make the translation happen. Starting from COMSOL, we just have to export into NASTRAN and eventually import the output in gmesh, once there it is possible to run the gmshToFoam utility. Of course the mesh will be tetrahedric so different numerical methods are needed to be implemented.

The configuration file blockMeshDic, inside the "system" folder, provides all the informations blockMesh requires to build up the mesh.

Now go on and open it up clicking [here](obsidian://open?vault=ANT&file=OpenFOAM%2FEX1_Powerlaw_cylinder%2Fsystem%2FblockMeshDic)

## 2D problems
OpenFOAM was born to solve 3D problems, for 2D problems we have to follow a procedure:
- build a 3D mesh
- i put just one element along z, width is arbitrary
- you have to change something into the boundary conditions files (we will see it)
![[Pasted image 20230426071111.png |200]]
## 2D axisymmetric problems

- 3D wedge of the geometry
- the side along x must be the symmetry axis!
- i put just one element along z
- the angle is arbitrary, must be small, the guide suggests 5 degrees
- the y axis must slice the geometry in half!
![[Pasted image 20230426071216.png|400]]

Once blockMesh is run successfully, it has created a folder inside the `constant` folder called  `polyMesh` containing all the info about the mesh splitted among several files.

## checkMesh
Once the blockMeshDic configuration file is written, the quality of the mesh and its statistics can be checked using `checkMesh`
```output
/*---------------------------------------------------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  10
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
Build  : 10-c4cf895ad8fa
Exec   : checkMesh
Date   : Apr 26 2023
Time   : 15:27:11
Host   : "bsm"
PID    : 3433
I/O    : uncollated
Case   : /home/bsm/Desktop/EX1_Powerlaw_cylinder
nProcs : 1
sigFpe : Enabling floating point exception trapping (FOAM_SIGFPE).
fileModificationChecking : Monitoring run-time modified files using timeStampMaster (fileModificationSkew 10)
allowSystemOperations : Allowing user-supplied system call operations

// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //
Create time

Create polyMesh for time = 0

Time = 0s

Mesh stats
    points:           3131
    internal points:  0
    faces:            6015
    internal faces:   2885
    cells:            1500
    faces per cell:   5.93333
    boundary patches: 6
    point zones:      0
    face zones:       0
    cell zones:       0

Overall number of cells of each type:
    hexahedra:     1400
    prisms:        100
    wedges:        0
    pyramids:      0
    tet wedges:    0
    tetrahedra:    0
    polyhedra:     0

Checking topology...
    Boundary definition OK.
    Cell to face addressing OK.
    Point usage OK.
    Upper triangular ordering OK.
    Face vertices OK.
    Number of regions: 1 (OK).

Checking patch topology for multiply connected surfaces...
    Patch               Faces    Points   Surface topology                  
    inflow              15       31       ok (non-closed singly connected)  
    outflow             15       31       ok (non-closed singly connected)  
    wall                100      202      ok (non-closed singly connected)  
    wedge1              1500     1616     ok (non-closed singly connected)  
    wedge2              1500     1616     ok (non-closed singly connected)  
    defaultFaces        0        0        ok (empty)                        

Checking geometry...
    Overall domain bounding box (0 0 -0.000436) (0.1 0.00999 0.000436)
    Mesh has 2 geometric (non-empty/wedge) directions (1 1 0)
    Mesh has 3 solution (non-empty) directions (1 1 1)
    Wedge wedge1 with angle 2.49901 degrees
    Wedge wedge2 with angle 2.49901 degrees
    All edges aligned with or perpendicular to non-empty directions.
    Boundary openness (4.54674e-19 -1.32946e-16 -8.40642e-17) OK.
    Max cell openness = 1.82132e-16 OK.
    Max aspect ratio = 3.003 OK.
    Minimum face area = 1.93584e-08. Maximum face area = 8.72e-07.  Face area magnitudes OK.
    Min volume = 1.93584e-11. Max volume = 5.61394e-10.  Total volume = 4.35564e-07.  Cell volumes OK.
    Mesh non-orthogonality Max: 0 average: 0
    Non-orthogonality check OK.
    Face pyramids OK.
    Max skewness = 0.330798 OK.
    Coupled point location match (average 0) OK.

Mesh OK.

End
``` 

It is worth noticing it checks the number of hexahedral elements, having only this kind of element is the best case scenario for the simulation's accuracy. Or the aspect ratio (e.g. 0.01 can be problematic so you can refine that if you read the `checkMesh` output)

### Centroids
Actually an important thing it prints out is the following
![[Pasted image 20230426132320.png]]
This check refers to the orthogonality (or the degree of, depending on the distance from 90 degrees) between the common side of two adjacent elements and the line connecting their centroids (the center of the element), where the fields are actually computed when the finite volume method is applied. 
![[Pasted image 20230426133021.png]]

![[Pasted image 20230426133045.png]]

![[Pasted image 20230426133116.png]]



