```cpp
viscosityModel  constant;

nu              [0 2 -1 0 0 0 0] 0.1; // kinematic viscosity eta/rho
				[kg m sec K altro]
```

First it is specified that the model for the viscosity for a newtonian fluid, the solver we will use solves the incompressible fluid equation, where nu is use during adimensionalization.

This parameter will be considered unless a different rheological model is otherwise specified as seen in lesson 10. 