# REQUIRES GEOMETRIC FUNCTIONS (cross, dot functions)

```c++
struct ray {
    point o, v;
    T c;

    // Constructors
    // From direction vector v and offset c
    ray(point v, T c, point o) : v(v), c(c), o(o) {}
    // From standard form equation ax + by = c
    // If the standard form equation is ax + by + c = 0, then we will assign the Position Constant c as -c
    ray(T a, T b, T c, point o) : v(b,-a), c(c), o(o) {}
    // From points p and q that belongs to the line : p is the ray start point
    ray(point p, point q) : v(q-p), c(cross(v,p)), o(p) {}

    // returns the orientation of a point with respect to the ray [integer friendly]
    // +ve -> point p is on the [left] side of the ray
    // -ve -> p is on the [right] side of the ray
    // zero -> point p is MAYBE on the ray
    T side(point p) {return cross(v,p) - c;}


    // sorting comparator for points [or their projections] on a line
    // as point p become far from vector v, the value of abs(||p||cos(theta)) will increase
    // returns (true) if p is before q with respect to Direction Vector v, (false) otherwise
    bool cmpProj(point p, point q) {
        return dot(v,p) < dot(v,q);
    }

    // returns the mn_distance between point p and the ray
    // if the projection of p is before the starting point o -> mn_dist = dist between p and o
    // otherwise, mn_dist = orthogonal distance from p to the ray
    // [[NOT] INTEGER FRIENDLY]
    T dist(point p) {
        if (p == o) return (T)0;
        return (cmpProj(p, o) ? abs(p - o) : abs(side(p)) / abs(v));
    }

    // returns (true) if there is an proper inter. between ray1 and ray2, false otherwise.
    // [integer friendly]
    bool sign(T number) {
        return (number > 0);
    }
    bool inter(ray r2, point &out) {
        T d = cross(v, r2.v);
        if (d == 0) return false;

        T t = cross(r2.o - o, r2.v); // parametric equation
        T u = cross(r2.o - o, v);
        if ((sign(t) ^ sign(d)) || (sign(u) ^ sign(d))) return false;
        out = (r2.v * c - v * r2.c) / d; // requires floating-point coordinates
        return true;
    }

    // returns the mn_distance between two rays r1, r2
    // if there is a proper intersection between r1 and r2 -> mn_dist = 0
    // otherwise mn_dist between r1.o and r2, r2.o, r1
    T distRay(ray r2) {
        point dummy;
        if (inter(r2,dummy)) return 0;
        return min(dist(r2.o), r2.dist(o));
    }

    // point p is on ray r if side(p) equals to zero and p is not before r.o
    bool onRay(point p) {
        return side(p) == 0 && !cmpProj(p,o);
    }
};
```