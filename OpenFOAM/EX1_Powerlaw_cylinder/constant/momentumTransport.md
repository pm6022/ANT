```cpp
simulationType laminar;
```

You just need to specify the type of regime. This is the newtonian case we first sa in lessons 8 and 9


Once we reached the lesson 10, we uncommented the rest of this file, to unlock the generalised newtonian model for Carreau/Yasuda coded as follows:
```cpp
simulationType laminar;

laminar
{
    model        generalisedNewtonian;

    viscosityModel powerLaw;

    nuMax    1; 
    nuMin    1e-02; 
    k        1e-01; 
    n        0.5; 
}
```