# REQUIRES GEOMETRIC FUNCTIONS

```c++
//! POLYGONS
// area of triangle abc = 1/2 * base * height
// = 1/2 * ab * (ac * sin(theta)) : theta -> angle between sides ab and ac
// = 1/2 * cross(ab, ac) // [[NOT] INTEGER FRIENDLY]
T areaTriangle(point a, point b, point c) {
    return abs(cross(b-a, c-a)) / 2.0;
}

// to calculate the area of any polygon, we will take the origin point O(0, 0) as a reference point
// then we will consider the vertices of the polygon in order, and for each pair of consecutive points
// U and V, we will add area(OUV) if UV vector goes counter-clockwise around O otherwise,
// we will subtract area(OUV), finally we have to divide the [area] by 2 because the crossProduct computes
// double the area of each triangle
// [[NOT] INTEGER FRIENDLY]
T areaPolygon(vector<point>& p) {
    double area = 0.0;
    for (int i = 0, n = p.size(); i < n; i++) {
        area += cross(p[i], p[(i + 1) % n]); // wrap back to 0 if i == n-1
    }
    return abs(area) / 2.0;
}
// IF THE VALUE OF VARIABLE (area) IS POSITIVE -> points are sorted [counter-clockwise]
// otherwise, points are sorted [clockwise]

// a polygon is considered to be convex iff the (SIGNS) of the orientation for every three consecutive
// points in a polygon are all the same (+ve -> points are sorted counter-clockwise, -ve -> clockwise)
bool isConvex(vector<point> p) {
    bool hasPos = false, hasNeg = false;
    for (int i = 0, n = p.size(); i < n; i++) {
        int o = orient(p[i], p[(i + 1) % n], p[(i + 2) % n]);
        if (o > 0) hasPos = true;
        if (o < 0) hasNeg = true;
    }
    return !(hasPos && hasNeg);
}

// returns true if point p is in the 1st or 2nd quadrants or on the -ve direction of x-axis (0, PI]
// returns false if point p is in the 3rd or 4th quad or on the +ve direction of x-axis (-PI, 0]
bool half(point p) {
    assert(p.X != 0 || p.Y != 0); // the argument of (0,0) is undefined
    return p.Y > 0 || (p.Y == 0 && p.X < 0);
}

// sorts a vector<point> by the following priorities in order:
// FIRST: points with half zero[3rd, 4th quad and +ve x-axis] come earlier than those with half one
// SECOND: if we have two points (v and w), the point that is located on the LEFT will come first
// THIRD: points that are in the exact same dir. are sorted by their dist from the origin (close points comes first)
void polarSort(vector<point> &v) {
    sort(v.begin(), v.end(), [&](point v, point w) {
        return make_tuple(half(v), 0, sq(v)) < make_tuple(half(w), cross(v, w), sq(w));
    });
}
// if we want to sort around some point C other than the origin,
// just subtract that point C from vectors v and w when comparing them


// true if P at least as high as A (blue part)
bool above(point a, point p) {
		return p.Y >= a.Y;
}
 
// check if [PQ] crosses ray from A
bool crossesRay(point a, point p, point q) {
		return (above(a,q)- above(a,p)) * orient(a,p,q) > 0;
}
 
// if strict, returns false when A is on the boundary
bool inPolygon(vector<point> p, point a, bool strict = true) {
		int numCrossings = 0;
		for (int i = 0, n = p.size(); i < n; i++) {
				if (onSegment(p[i], p[(i+1)%n], a))
						return !strict;
				numCrossings += crossesRay(a, p[i], p[(i+1)%n]);
		}
		return numCrossings & 1; // inside if odd number of crossings
}
```