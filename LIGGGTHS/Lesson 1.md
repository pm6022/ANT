DEM stands for Discrete Element Method, it is similar to Method used in Molecular Dynamics because we are also dealing with particles but are larger and are not point-like, on the contrary they have a finite volume, even a complex one, and the collisions are among surfaces.

$N_p=\frac{6\epsilon_p V}{\pi d_p^3}=(\frac{L}{d_p})^3$ with random close packing ε_(p,RCP)≈0.64 (spheres randomly poured in a volume)

We have many particles, so computation is demanding.

# Mathematical model
Torque and Force Balances on each spherical particle
Newton's Law, considering Range Forces and Contact Forces
- $m_p\frac{du_p}{dt}=F_c+F_r$
Angular momentum balance, where $I_p$ is the inertia tensor (actually
![[Pasted image 20230522155954.png]]
for a sphere the w is an eigenvector of the inertia matrix, therefore $I_p$ is the scalar moment of inertia, due to the symmetry, the tensor is this scalar times the unit tensor, therefore this equation is valid only for the sphere, as the previous one is refered to the velocity of the center of mass)
- $I_p \frac{dw_p}{dt} = T_c+T_r$ 

### Nota bene
When 2 spheres are touching, the contact point is always on the line passing through the two centers. This is called collinear contact.
This point belongs to the contact plane, that is orthogonal to the conjunction line of the centers.
This aforementioned plane is quite useful to define normal and parallel components of the forces acting on contact. For sphere the normal component will always be on the jointing therefore we just need to know the positions of the two centers.

## Hard sphere approach

- The two spheres exchange momentum and energy impulsively, they don't stay in contact for a certain time. A sphere on a surface is a problem, they would exchange momentum and energy for ever.
- It is used in rarefied systems, where there are less interactions
- and the interactions are only rebounding, not of contact for a finite time
- It is easier from a computing point of view

## Soft sphere
![[Pasted image 20230522222211.png]]

- rigid particles can approach one other until the distance between them is equal to the sum of their radii, that is the moment there is a contact.
- meanwhile soft spheres when they collide they deform locally in the contact plane, the more rigid the material of the particles the less important the deformation, this deformation is called Overlapping, indicated as $\delta = R_i+R_j-\Vert x_i-x_j \Vert$  
- the overlapping is generally considered equally distributed between the two particles, actually  the stiffer sphere would deform less than the less stiff one.
- this model is more useful for our purpose, because a sphere can sit on a surface without exchange energy and momentum for ever, they would both deform and stay still in equilibrium
- using this method it is possible to have multiple contacts unlike the Rigid sphere where they exchange momentum and energy one contact at a time.
- the computational problem arising from this model is caused by the timescale of this deformation, considering a constant velocity the small deformation implies a small needed time to occur therefore more rigid materials demand smaller timesteps. The deformation is also a small fraction of the sphere's radius therefore also smaller particles are computational demanding.

## Interaction Type

![[Pasted image 20230522222151.png]]

## Contact model: Hertz-Mindling model
- Hooke is the classical one but we use Hertz-Mindling
- the contact force is decomposed in normal and tangential component, the first one is on the jointing of the centres, the second one belongs to the tangential plane
- each component has two contributions: an elastic one and a viscous one, just like the viscoelastic model we saw: $F_c = (k_n\delta_n+\gamma_n\frac{\delta_n}{dt})n-(k_t\delta_t+\gamma_t\frac{\delta_t}{dt})t$ 
- for the torque we consider only sliding interaction therefore: $T_c=-r_iF_cn\times t$ 
- for the Hooke model we only have to deal with the elastic part and the $k_n$ is a constant, so the force is linear with deformation, in contrast in the Hertz-Mindling $k_n$ depends on deformation therefore is a nonlinear relationship. The other parameters depend on deformation too:
![[Pasted image 20230522234259.png]]

## Coulomb friction limit
$F_c$ can't diverge, its maximum value is $\mu F_n$ 

``` chatGPT
In other words, the static friction force resists the start of motion and adjusts its value to match the applied external force up to a maximum limit, μs * N. Beyond this point, if the applied external force continues to increase, the object will start to slide or move. This transition from static to kinetic friction is often associated with a decrease in the frictional force, as the coefficient of kinetic friction (μk) is typically less than the coefficient of static friction (μs).
```

## Nota bene 
$k_n$ and $k_t$ depend on the material's mechanical properties, the $\epsilon$ is the coefficient of restitution.
We have several properties for the two bodies but we can model the collision as if the point of contact is seeing a collision with a equivalent single body. As depicted in the table:
![[Pasted image 20230523001441.png|200]]
So the problem is a single body with respect to its center of mass 50'15" thanks to this approximation
q: what?

So i can account for multiple contacts but for every pair i can write down a reduced system using the equivalent parameters.

## Restitution Factor

$\epsilon = u_R/u_I$ 
these are relative velocities before and after the collision (impact and rebound)
$\epsilon = 1$  perfectly elastic collisions (elastic energy is totally)
$\epsilon = 0$  inelastic collisions  (all energy is dissipated and/or it is absorbed by the material leading to rupture) 
in the simulation we have 0.99 cause of physical meaning: there cannot be a perfectly elastic collision,you can simulate it but the velocity of the system could diverge neither an anelastic one: 0.05 because of numerical reasons. 
![[Pasted image 20230524223848.png|300]]
The stokes number is useful to determine the restitution factor (the plot is valid for not too high velocities because it doesn't account for plastic deformation and rupture that have no elasticity indeed) 

This plot is valid for low viscous fluid like air, 
in water there are other dissipation phenomena to take into account like the squeezing of the thin film between the particle and a surface before the impact.

## Time step

To evaluate the timestep we take a fraction of the lowest one between the Rayleigh time and the Hertz time

- Rayleigh: the time related to the diffuse of mechanical wave when there is a deformation. Deformation is not instantaneous, it takes time. When you press a incompressible liquid, all the molecules instantaneously start to move, while in reality there is elasticity due to molecules' compression at start then a propagation. The parameters are of the particle. 
- Hertz: parameters are the reduced ones. it is the time of total contact in an elastic bounce. So it is related to the maximum overlapping between them, when the velocity goes to zero. Total contact means is the time to overlap, reach the maximum, zero velocity and starting going backward until the distance is the sum of radii. So i need a small timestep with respect to Hertz time in order to capture the total contact smoothly.
- 10^-8/10^-9/picoseconds is not unusual because of velocities and rigidity so they will require a lot of time.
To accelerate this simulations, there is a trick: reducing the Young modulus therefore the forces are underestimated, restitution and velocities are the same but the forces are lower but the directions are the same. If the scope of the simulation is mechanics-related i must use the real Young modulus. 

## Non spherical particles
- multisphere(jointing between centers is easy)
- superquadric (sphere's equation is modified algebraicly to get cylinders or tablets, to manage contacts is complex, you need to solve equations)
- rocks can be meshed with tetrahedrons and then the interactions between them (rocky not liggghts)
## Smoothed Particle Hydrodynamics
- videogames
- movies
Particles interact with neighbourhood therefore are easily treated with parallel computation, cfd is not that easy to have parallel computation because you have a big matrix with all the elements, you have to close the balance.





