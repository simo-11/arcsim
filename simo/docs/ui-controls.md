# ui-controls

## main actions documented in [README](../..README) section  Using the OpenGL interface

Space:          start simulation/playback
Left drag:      rotate
Middle drag:    translate
Scroll wheel:   scale
Esc:            exit

## keyboard actions in [displayphysics.cpp:keyboard](../../src/displayphysics.cpp) 
  * s - single step simulation 

## keyboard actions in [display.cpp:keyboard_handler](../../src/display.cpp)
  * z - zooms in 
  * x - zooms out 
  * n - sets stepDebug to false 
    * causes simulation to continue after debug interactions
  * <TAB> - switch display mode which is shown in window title after debug: 
    * sigma
    * sigma_bend
    * sep strength
    * Sp_str
    * Sp_bend
  * ] and [ - modify scale of current display mode
    * ] (increase by factor of 2) 
    * [ (decrease, multiply by factor of 0.5)  
  
## other mouse actions [display.cpp:select_element](../../src/display.cpp)
Code does not provide expected results and could be improved.  
  * click on face using left button - provides information on selected face 
```
f:0af(v:082-v:084-v:085) index 0
curvw1 0
curvw2 0
vel 0
comp 0
obs 0
frac 0
total 0
max sigma 0 (toughness 1.2e+06)
aspect 0.891519
Sp_str (
    (1, 0, 0),
    (0, 1, 0),
    (0, 0, 1)
)
theta ideal 0 0 0  
```
  * click on vertex using left button - provides information on selected vertex
```
v:082 index 0
preserve 1 flags 0 label 0
sep 0
sizing (
    (0, 0, 0),
    (0, 0, 0),
    (0, 0, 0)
)
x (0, 0, 0)
u (0, 0, 0)
vel (0, 0, 0)
acc (0, 0, 0)
f:0af(v:082-v:084-v:085) f:0b0(v:082-v:085-v:087)   
```
    
