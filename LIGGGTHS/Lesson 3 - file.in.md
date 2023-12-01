```liggghts
###############################################################################

### Bottega della Materia Soffice                                           ###

### DICMaPI - Università degli Studi di Napoli "Federico II"                ###

###############################################################################

  

### 1. INITIALIZATION__________________________________________________________________________________________________

  ## Global Parameters

    echo        both            # Sends the input script to screen and log file

    units       si              # Unit reference (SI system: kg,m,s,...)

    atom_style  granular        # Particle style (granular)

    atom_modify map array       # How particle ID are stored

    #hard_particles yes          # Needed to set Young modulus >1e9

    boundary    f f p           # Boundaries of the box (x,y,z): f=fixed p=periodic

    newton      off             # Denies the calculation of pairwise interaction by Newton's 3rd law

    communicate single vel yes  # Sets the style of inter-processor communication (best for granular (no cutoff))

    processors  1 1 1           # Subdomain division for parallelization (x,y,z)

  

  ## Variables

    # Chamber

    variable chamberH       equal 0.05        # Chamber Height

    variable chamberR       equal 0.1         # Chamber Radius

    variable chamberOmega   equal 3.1416      # Roller angular velocity [rad/s]

    variable chamberU       equal $(v_chamberOmega*v_chamberR) # Chamber tangential velocity (vshear > 0 -> counter-clockwise)

  

    # Particles

    variable rp             equal 0.0025       # Particle Radius

    variable dens           equal 1000.       # Particle Density

    variable volp           equal $(4/3*3.1416*v_rp^3)  # Particle Volume

  

    # Simulation parameters

    variable dt             equal 1e-5        # Time step DEM [s]

    variable nsteps         equal 1.5e5         # Number of time steps for initialization (N.B. need to be < 2x10^9)

    variable dumpevery      equal 2e3         # Number of time steps between dumps

    variable thermoevery    equal ${dumpevery} # Number of time steps between thermo prints

    variable restartevery   equal 1e5         # Number of time steps between thermo prints

  

    # Particle Insertion Properties

    variable phi            equal 0.2         # Solid volume fraction in the insertion region

    variable Npart          equal 5000        # Number of particles in the insertion region

    variable maxatt         equal 1000        # Maximum number of insertion attempts per atom

    variable MCnumber       equal 10000       # Number of Monte-Carlo steps for calculating the region volume

    # Material and interaction properties

    variable gforce         equal 9.81        # Gravity Force Modulus

    variable YouMod1        equal 1e8         # Young's Modulus for atom type "1" 0.1GPa

    variable YouMod2        equal 1e8         # Young's Modulus for atom type "2"

  

    variable PoiRat1        equal 0.3         # Poisson's Ratio for atom type "1"

    variable PoiRat2        equal 0.3         # Poisson's Ratio for atom type "2"

    variable cRest11        equal 0.99        # Restitution Coefficient 11 (Relative speed after collision / before collision)

    variable cRest22        equal ${cRest11}  # Restitution Coefficient 22

    variable cRest12        equal ${cRest11}  # Restitution Coefficient 12 or 21

  

    variable cFric11        equal 0.5         # Friction Coefficient 11 (Tangential friction force / Normal force)

    variable cFric22        equal ${cFric11}  # Friction Coefficient 22

    variable cFric12        equal ${cFric11}  # Friction Coefficient 12 or 21

  

  ## Domain definition

    region domain block $(-v_chamberR) ${chamberR} $(-v_chamberR) ${chamberR} 0 ${chamberH} units box

    create_box 2 domain # atom types present

  

### 2. SETUP___________________________________________________________________________________________________________

  

  ## Neighbor Listing

    neighbor $(v_rp/2) bin  # set the skin for the neighbor list (distance < ri+rj+skin)

    neigh_modify delay 0    # Rebuilding of neighbor list at every step

  ## Definition of the physic model to be used for pairwise interactions

    pair_style gran model hertz tangential history # Hertzian with tangential friction

    pair_coeff * *

  

  ## Material and interaction properties (fix property/global is not applied to atoms then not saved in the restart file)

    # peratomtype -> value_i = value for atom type i

    # peratomtypepair -> n_atomtypes value_11 value_12 ... value_ij = value between atom type i and j

    fix m1  all property/global youngsModulus peratomtype ${YouMod1} ${YouMod2}

    fix m2  all property/global poissonsRatio peratomtype ${PoiRat1} ${PoiRat2}

    fix m3  all property/global coefficientRestitution peratomtypepair 2 ${cRest11} ${cRest12} ${cRest12} ${cRest22}

    fix m4  all property/global coefficientFriction peratomtypepair 2 ${cFric11} ${cFric12} ${cFric12} ${cFric22}

  

    fix grav all gravity ${gforce} vector 0. -1. 0.

  

  ## Boundaries (mesh import from cad)

    fix cad1 all mesh/surface file cylinder.stl type 1 scale 1

    fix wall all wall/gran model hertz tangential history mesh n_meshes 1 meshes cad1

    fix move all move/mesh mesh cad1 rotate origin 0. 0. 0. axis 0. 0. 1. period $(2*3.1416/v_chamberOmega)

  

  ## Boundaries (primitive "zcylinder radius c1 c2")

    #fix chamberwall all wall/gran model hertz tangential history primitive type 1 zcylinder ${chamberR} 0.0 0.0 shear x ${chamberU}

  

  ## Particle insertion

    region insreg cylinder z 0. 0. $(0.7*v_chamberR) 0. ${chamberH} units box #cylinder args = dim c1 c2 radius lo hi

    #region insreg sphere 0. 0. $(chamberH/2) $(chamberH/2) units box #sphere args = x y z radius

    #region insreg mesh/tet file mesh_ins.vtk scale 1.0 move 0. 0. 0. rotate 0. 0. 0. units box side in

  

    fix pts all particletemplate/sphere 10007 atom_type 1 density constant ${dens} radius constant ${rp}

    fix pdd all particledistribution/discrete 10037 1 pts 1.

    fix ins all insert/pack seed 10103 distributiontemplate pdd verbose yes insert_every once maxattempt ${maxatt} overlapcheck yes all_in yes vel constant 0. 0. 0. region insreg volumefraction_region ${phi} ntry_mc ${MCnumber}"

    # overlapcheck can be an issue at high volume fraction, hence (check_dist_from_subdomain_border no) can't be used

    # can use (volumefraction_region ${phi}) instead of (particles_in_region ${Npart})

  

### 3. INTEGRATION AND RESULTS_________________________________________________________________________________________

  

  ## Integrator

    fix integrate all nve/sphere

    timestep ${dt}

  

  ## Time step checking and dump files

    fix tscheck all check/timestep/gran ${dumpevery} 0.01 0.01 warn no # Prints a warning if the time step is higher that 1% of Rayleigh or Hertz time

    variable time       equal step*dt

    variable Rayleigh   equal f_tscheck[1]

    variable Hertz      equal f_tscheck[2]

    fix extra all print ${dumpevery} "${time} ${Rayleigh} ${Hertz}" file log/tscheck.out screen no title "# Fraction of the Rayleigh and Hertz time-step sizes"

  

  ## Cluster group management

    compute ke all ke

    compute erotate all erotate/sphere

    variable  kenergy   equal c_ke # needed to update the compute and print its value

    variable  wenergy   equal c_erotate

    fix extra2 all print ${thermoevery} "${time} ${kenergy} ${wenergy}" file log/log.thermo screen no title "# Overall particle translational and rotational kinetic energy"

  

  ## Thermodynamics output settings

    thermo_style custom step atoms ke erotate #cpu #elapsed #erotate

    thermo ${thermoevery}

    thermo_modify norm no lost ignore format float %.4e

  

  ## Results output (dump)

    dump dmp all custom ${dumpevery} post/dump*.out id type x y z vx vy vz fx fy fz omegax omegay omegaz radius

    dump_modify dmp sort id

    dump dumpstl all mesh/stl ${dumpevery} post/dumpcyl*.stl

    #dump dumpvtk all mesh/vtk ${dumpevery} post/wall*.vtk stress cad1

  

  ## Write restart file (A restart file stores: units and atom style, simulation box size and shape and boundary settings, group definitions, per-type atom settings such as mass, per-atom attributes including their group assignments and molecular topology attributes, force field styles and coefficients, and special_bonds settings. !!! The list of fixes used for a simulation is not stored in the restart file. This means the new input script should specify all fixes it will use. Note that some fixes store an internal "state" which is written to the restart file.)

  # Unlike dump/ and vtk/, the folder restart/ need to be already present !

    restart ${restartevery} restart/liggghts*.restart

  

  ## Execution

    run ${nsteps}
```
