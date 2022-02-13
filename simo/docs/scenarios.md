# Introduction

# Common parameters

## Geometries are based on [plate.obj](../mesh/plate.obj) and its variatios
plate defines unit rectangle having left lower corner at origo and is in xy-plane
```
v 0 0 0
v 1 0 0
v 1 1 0
v 0 1 0
f 1 4 3 2
```
[beam](../mesh/beam.obj) has non-zero x-coordinates of 10 and non-zero y-coordinates of 0.1

# [Elastic deformation](../conf/elastic.json)
 * plate-100 is used to model geometry
 * handles are used to fix vertices at x=0
 * thickness is set to 1
 * According to beam theory which applies well for these measures tip displacement is 0.546 [m]


# Ductile plastic deformation due to bending

# Fracture after plastic deformation due to bending

# Fracture due to cutting
