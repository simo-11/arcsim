# scene-file documentation
Based on code and sample files [sphere-annotated.json](../../conf/sphere-annotated.json) 
## Toplevel parsing in parse method in [conf.cpp](../../src/conf.cpp)
Contains storing of configuration part into [Simulation sim](../../src/Simulation.hpp)
  * simulation timing - see code below for details
    * frame_time [s] primary way to control how often image is created 
    * timestep [s] alternative way to control simulation timestep
    * end_time [s] defines end time
  * cloths - deforming objects. Naming is probably due to historical reasons. Eachs cloth has defitions for
    * [reference](reference.md) - reference for mesh "linear" or "sphere". Other values will cause error "Reference mesh lookup not implemented here" from referenceshape.cpp as there is no implementation.  
    * [mesh](mesh.md) - initial mesh in obj format
    * transform - Initial pose of mesh (optional), object contains optional entries
      * "translate": [x,y,z]
      * "rotate": [angle,axis_x,axis_y,axis_z]
      * "scale": scale
    * [materials](materials.md) -  material properties for each cloth
    * remeshing - Remeshing parameters
  * motions - List of motions for handles and/or obstacles (optional)
  * handles - List of handles to predescribe motion for specific items
    * node (default) [parse_node_handle in conf.cpp](../../src/conf.cpp)
      * cloth - default 0
      * label - mesh nodes having same label are selected
      * nodes - mesh nodes are selected using mesh indexes 
    * circle [parse_circe_handle in conf.cpp](../../src/conf.cpp)
    * glue [parse_glue_handle in conf.cpp](../../src/conf.cpp)
    * soft  [parse_soft_handle in conf.cpp](../../src/conf.cpp)
    * motion <index> - Optional: Index of motion to attach to, hf omitted, handle does not move
    * start_time <time> - Optional, default 0: When handle activates
    * end_time <time> - Optional, default infinity: When handle deactivates
    * fade_time <time>
  * obstacles - List of static or scripted obstacles
  * gravity - Acceleration due to gravity (optional)
  * wind - (optional)
  * friction - Coeff. of cloth-cloth friction (optional)
  * obs_friction - Cloth-obstacle friction (optional)
  * disable - Names of moduless to disable. strainlimiting, plasticity and fracture are disabled if corresponding material properties are not defined
  * [magic](magic.md) - numbers to make the simulation behave
```
    if (!json["frame_time"].empty()) {
        parse(sim.frame_time, json["frame_time"]);
        parse(sim.frame_steps, json["frame_steps"], 1);
        sim.step_time = sim.frame_time/sim.frame_steps;
        parse(sim.end_time, json["end_time"], infinity);
        parse(sim.end_frame, json["end_frame"], infinity);
    } else if (!json["timestep"].empty()) {
        parse(sim.step_time, json["timestep"]);
        parse(sim.frame_steps, json["save_frames"], 1);
        parse(sim.save_every, json["save_every"], 1);
        sim.frame_time = sim.step_time*sim.frame_steps;
        parse(sim.end_time, json["duration"], infinity);
        sim.end_frame = infinity;
    }
    parse(sim.passive_time, json["passive_time"], infinity);
    sim.time = 0;
    parse(sim.cloths, json["cloths"]);
    parse_motions(sim.motions, json["motions"]);
    parse_handles(sim.handles, json["handles"], sim.cloths, sim.motions);
    parse_obstacles(sim.obstacles, json["obstacles"], sim.motions);
    parse_morphs(sim.morphs, json["morphs"], sim.cloths);
    parse(sim.gravity, json["gravity"], Vec3(0));
    parse(sim.wind, json["wind"]);
    parse(sim.friction, json["friction"], 0.6);
    parse(sim.obs_friction, json["obs_friction"], 0.3);
    string module_names[] = {"proximity", "physics", "strainlimiting",
                             "collision", "remeshing", "separation",
                             "popfilter", "plasticity", "fracture"};
    for (int i = 0; i < Simulation::nModules; i++) {
        sim.enabled[i] = true;
        for (int j = 0; j < (int)json["disable"].size(); j++)
            if (json["disable"][j] == module_names[i])
                sim.enabled[i] = false;
    }
    parse(::magic, json["magic"]);
    // disable strain limiting and plasticity if not needed
    bool has_strain_limits = false, has_plasticity = false, has_fracture = false;
    for (int c = 0; c < (int)sim.cloths.size(); c++)
        for (int m = 0; m < (int)sim.cloths[c].materials.size(); m++) {
            Material *mat = sim.cloths[c].materials[m];
            if (is_finite(mat->strain_min) || is_finite(mat->strain_max))
                has_strain_limits = true;
            if (is_finite(mat->yield_curv))
                has_plasticity = true;
            if (mat->toughness > 0)
            	has_fracture = true;
        }
    if (!has_strain_limits)
        sim.enabled[Simulation::StrainLimiting] = false;
    if (!has_plasticity)
        sim.enabled[Simulation::Plasticity] = false;
    if (!has_fracture)
    	sim.enabled[Simulation::Fracture] = false;    
 ```
 
