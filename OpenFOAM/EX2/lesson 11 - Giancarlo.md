Multiphase systems
1) motivation
The bubble rising is a benchmark problem for mulriphase systems, hence it is a way to validate and test the accuracy of new algorithms, solvers, numerical methods or code.

Thanks to numerical simulation we can estimate, for instance, the centroid's velocity and the bubble's shape. Moreover, starting from the analysis of a single bubble rising it is possible to deduce useful information that could be extended of semidilute or even concentrated systems.

2) problem definition
Assumptions:
- isothermal flow
- negligible gas diffusion inside dispersing phase
- incompressible, newtonian fluids (air and water)

Equations:
$$ \nabla\cdot v=0$$
$$\tau=-pI+\mu(\nabla v+\nabla v^T)$$
$$\rho(\frac{\partial v}{\partial t}+v \cdot \nabla v)=\nabla \cdot \tau +f_b$$
the body forces according to Giancarlo are gravity and  the surface tension (that is not a body force, BUT because we can't use a boundary condition because there is no boundary using vof method so we are dealing with front capturing and not tracking, so there is a term seen as a volume force) $$ \sigma \kappa \vec{n} $$ that is not how it is implemented but it is quite complicated algorithmically. The curvature is evaluate using the partial derivative of $\phi$ , because there's interface where it is changing the indicator function between 0 and 1.

![[Pasted image 20230505214754.png]]
the upper part is the opening of the container.

Dimensionless numbers
![[Pasted image 20230505221029.png|500]]

Morton is useful because it considers only the properties of the two fluids.
When we'll have run the simulation, we'll get the terminal velocity of the bubble and we'll compute Re and We to respectively compare inertia forces with viscous forces and surface tension.

Numerical method
Volume of fluid method, quite useful because it evaluates integrals

Whereas we saw an interface tracking method ALE (Arbitrary Lagrangian Euler), where the interface is a well defined boundary of the system *that moves according to physical laws*.
Therefore has a good accuracy whereas it can't deal with topological changes and relevant mesh deformation needs remeshing and that requires more computational power.

Interface capturing method is what we use, totally Eulerian method, nodes are fixed. This implies an easier algorithm but on the other hand there is the drawback of not having the real interface as a boundary, so we can't apply a boundary condition on that. We need a new equation: 

$$ \frac{\partial \phi}{\partial t}+\nabla \cdot (\phi v)=0$$

Where $\phi$ is the indicator function (volume fraction), in this case study we use a pure advection equation, it is a sort of transport equation for the interface. *Can i have use generation term to model a phase change?*  

We solve using a single fluid formulation, we have a single viscosity and a single density as weighted average of the two fluid where the weights are the volume fractions.


![[Pasted image 20230506135850.png]]

![[Pasted image 20230506140918.png]]

notice the 2 courants and the staibility condition's reference

![[Pasted image 20230506160815.png]]
this integrals are computed to attain the 'boundary' 

thanks to simulation we can observe how shape changes with dimensionless numbers

results may vary with respect to experiments due to unconfinement's degree


nota bene, non stiamo considerando van der walls, la coalescenza in genere potrebbe non avvenire sempre se ci sono surfattanti, nel nostro modello avviene sempre, non serve una forza minima per ottenere coalescenza, non dipende dalla mobilità dell'interfaccia come nei flussi reali:
![[Pasted image 20230506190545.png]]