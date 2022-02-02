# mesh
parsing is in load_obj method in [io.cpp](../../src/io.cpp).
See e.g. [wikipedia](https://en.wikipedia.org/wiki/Wavefront_.obj_file)
for more detailed information.
Vertices are stored in a counter-clockwise order by default, making explicit declaration of face normals unnecessary.

## Mesh objects [mesh.hpp](../../src/mesh.hpp)

### struct Node
```
	enum NodeFlags { FlagNone = 0, FlagActive = 1, FlagMayBreak = 2, 
                     FlagResolveUni = 4, FlagResolveMax = 8 };
    int label;
    int flag;
    std::vector<Vert*> verts;
    Vec3 y; // plastic embedding
    Vec3 x, x0, v; // position, old (collision-free) position, velocity
    bool preserve; // don't remove this node
    // topological data
    int index; // position in mesh.nodes
    std::vector<Edge*> adje; // adjacent edges
    // derived world-space data that changes every frame
    Vec3 n; // local normal, approximate
    // derived material-space data that only changes with remeshing
    double a, m; // area, mass
    Mat3x3 curvature; // filtered curvature for bending fracture
    // pop filter data
    Vec3 acceleration;
explicit Node (const Vec3 &y, const Vec3 &x, const Vec3 &v, int label, int flag, 
    	bool preserve) :
        uuid(uuid_src++), mesh(0), sep(0), label(label), flag(flag), y(y), x(x), x0(x), v(v), preserve(preserve), 
        curvature(0) {}    
```

### struct Vert
```
    Vec3 u; // material space
    Node *node; // world space
    // topological data
    std::vector<Face*> adjf; // adjacent faces
    int index; // position in mesh.verts
    // derived material-space data that only changes with remeshing
    // remeshing data
    Mat3x3 sizing;
```
### struct Edge
```
   Node *n[2]; // nodes
    int preserve;
    // topological data
    Face *adjf[2]; // adjacent faces
    int index; // position in mesh.edges
    // plasticity data
    double theta_ideal, damage; // rest dihedral angle, damage parameter
 ```

### struct Face
```
    Vert* v[3]; // verts
    Material* material;
    int flag;
    // topological data
    Edge *adje[3]; // adjacent edges
    int index; // position in mesh.faces
    // derived world-space data that changes every frame
    Vec3 n; // local normal, exact
    // derived material-space data that only changes with remeshing
    double a, m; // area, mass
    Mat3x3 Dm, invDm; // finite element matrix
    // plasticity data
    Mat3x3 Sp_bend; // plastic bending strain
    Mat3x3 Sp_str; // plastic stretching
    Mat3x3 sigma;
    double damage; // accumulated norm of S_plastic/S_yield
```

### struct Mesh
```
	ReferenceShape *ref;
	Cloth* parent;
  CollisionProxy* proxy;
  std::vector<Vert*> verts;
  std::vector<Node*> nodes;
  std::vector<Edge*> edges;
  std::vector<Face*> faces;
```

## processing of obj lines 
### v - vertex
coordinates are stored as initial values of Node's plastic embedding and old and current position
```
Vec3 x;
linestream >> x[0] >> x[1] >> x[2];
mesh.add(new Node(x, x, Vec3(0), 0, 0, false));
..
void Mesh::add (Node *node) {
    nodes.push_back(node);
    node->index = nodes.size()-1;
    node->adje.clear();
    for (size_t v = 0; v < node->verts.size(); v++) {
        node->verts[v]->node = node;
    }
    node->mesh = this;
}
```

### f - face
Adds Verts and Faces to mesh
```
            vector<Vert*> verts;
            vector<Node*> nodes;
            string w;
            while (linestream >> w) {
                stringstream wstream(w);
                int v, n;
                char c;
                wstream >> n >> c >> v;
                nodes.push_back(mesh.nodes[n-1]);
                if (wstream) {
                    verts.push_back(mesh.verts[v-1]);
                } else if (!nodes.back()->verts.empty()) {
                    verts.push_back(nodes.back()->verts[0]);
                } else {
                	  verts.push_back(new Vert(nodes.back()->x));
                    mesh.add(verts.back());
                }
            }
            for (int v = 0; v < (int)verts.size(); v++)
                connect(verts[v], nodes[v]);
            vector<Face*> faces = triangulate(verts);
            for (int f = 0; f < (int)faces.size(); f++)
                mesh.add(faces[f]);
..
void Mesh::add (Vert *vert) {
    verts.push_back(vert);
    vert->node = NULL;
    vert->adjf.clear();
    vert->index = verts.size()-1;
}
..
void Mesh::add (Face *face) {
    faces.push_back(face);
    face->index = faces.size()-1;
    // adjacency
    add_edges_if_needed(*this, face);
    for (int i = 0; i < 3; i++) {
        Vert *v0 = face->v[NEXT(i)], *v1 = face->v[PREV(i)];
        include(face, v0->adjf);
        Edge *e = get_edge(v0->node, v1->node);
        face->adje[i] = e;
        int side = e->n[0]==v0->node ? 0 : 1;
        e->adjf[side] = face;
    }
}

```

