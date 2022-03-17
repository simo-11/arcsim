# timing in results directory
timing is a dump of the wall-clock compute time spent per time step
  in each routine of the simulation (README)
Each line contains entry for each module

[runphysics.cpp](../../src/runphysics.cpp)
```
static void save_timings () {
    static double old_totals[Simulation::nModules] = {};
    if (!::timingfile)
        return; // printing timing data to stdout is getting annoying
    ostream &out = ::timingfile ? ::timingfile : cout;
    for (int i = 0; i < Simulation::nModules; i++) {
        out << sim.timers[i].total - old_totals[i] << " ";
        old_totals[i] = sim.timers[i].total;
    }
    out << endl;
}
```
[simulation.hpp](../../src/simulation.hpp)
```
enum {Proximity, Physics, StrainLimiting, Collision, Remeshing, Separation,
          PopFilter, Plasticity, Fracture, nModules};
```
## Sample output from sphere-break.json
```
4.31331 5.96738 0 2.54031 16.1944 0.363076 14.6288 0 0
4.3319 6.05657 0 2.25023 3.54577 0.326712 13.7956 0 0
4.29006 5.98368 0 2.24725 3.51193 0.336261 13.7219 0 0
```
 in table ` cat results/timing | sed -e 's/^/|/' -e 's/ / | /g'`
 |Proximity|Physics|StrainLimiting| Collision| Remeshing| Separation|PopFilter| Plasticity| Fracture|
 --- | ---- | --- | ---- | --- | ---- | --- | ---- | --- |
|4.31331 | 5.96738 | 0 | 2.54031 | 16.1944 | 0.363076 | 14.6288 | 0 | 0 |
|4.3319 | 6.05657 | 0 | 2.25023 | 3.54577 | 0.326712 | 13.7956 | 0 | 0 |
|4.29006 | 5.98368 | 0 | 2.24725 | 3.51193 | 0.336261 | 13.7219 | 0 | 0 |

```
/github/arcsim$ bin/arcsimd simulate conf/sphere-break.json results/sphere-break
```
Run stalls at Sim frame 248 time 0.012 at time of collision
Before starting simulation
![image](https://user-images.githubusercontent.com/1210784/158882392-030e103b-cd4b-4a59-8424-99f63cfe8412.png)
```
Sim frame 249 [249]
Collision resolution failed to converge!
```
![image](https://user-images.githubusercontent.com/1210784/156644315-41ebbb34-a306-45b3-a4ed-57f50dc6a7a7.png)

