# REQUIRES GEOMETRIC FUNCTIONS (cross, perp, sq) and STRAIGHT LINES (in circleLineInter)

```c++
// circumcircle of a triangle abc is the circle that passes through all 3 points a, b and c
// returns the circumcenter of the circumcircle of triangle abc
// radius of circumcircle = dist(circumcenter, any_point [a, b or c])
// [[NOT] INTEGER FRIENDLY]
point circumCenter(point a, point b, point c) {
    b = b - a, c = c - a;
    assert(cross(b,c) != 0);
    return a + perp(b * sq(c) - c * sq(b)) / cross(b,c) / 2.0; // a + radius
}

// returns 1 [POSITIVE], 0 [ZERO], -1 [NEGATIVE]
template<typename T> int sgn(T x) {
    return (T(0) < x) - (x < T(0));
}
// returns the number of inter. points [and places them in (out) IF THEY EXIST] between line l and a circle
// number of inter. points = 0, 1(tangent line) or 2
int circleLine(point o, double r, line l, pair<point, point>& out) {
    double h2 = r * r - l.sqDist(o);
    if (h2 >= 0) { // the line touches the circle
        point p = l.proj(o); // point P (midpoint)
        point h = l.v * sqrt(h2) / abs(l.v); // vector parallel to line l with magnitude h
        out = {p - h, p + h};
    }
    return 1 + sgn(h2);
}

// returns the number of inter. points between [and places them in (out) IF THEY EXIST] two circles
// number of inter. points = 0, 1, 2 or infinity
// aborts if the two circles are concentric and identical
int circleCircleInter(point o1, double r1, point o2, double r2, pair<point, point>& out) {
    point d = o2 - o1;
    double d2 = sq(d);
    if (d2 == 0) {assert(r1 != r2); return 0;} // concentric circles
    double pd = (d2 + r1 * r1 - r2 * r2) / 2; // = |O_1P| * d
    double h2 = r1 * r1 - pd * pd / d2; // = h^2
    if (h2 >= 0) {
        point p = o1 + d * pd / d2, h = perp(d) * sqrt(h2 / d2);
        out = {p - h, p + h};
    }
    return 1 + sgn(h2);
}

// returns the number of tangents [and places them as a pair inside vector (out)]
// number of tangents = 0, 1(the two circles are tangent to each other at point p) or 2
// in case of 1 tangent, we can find the tangent line by [line(o1, p).perpThrough(p)]
// aborts if the two circles are concentric and identical

// the same code can be used to find the tangent to a circle passing through a point
// by setting r2 to 0 (in this case the value of inner doesn't matter)
int tangents(point o1, double r1, point o2, double r2, bool inner, vector<pair<point, point > > &out) {
    if (inner) r2 = -r2;
    point d = o2 - o1;
    double dr = r1 - r2, d2 = sq(d), h2 = d2 - dr * dr;
    if (d2 == 0 || h2 < 0) {assert(h2 != 0); return 0;}
    for (double sign : {-1, 1}) {
        point v = (d * dr + perp(d) * sqrt(h2) * sign) / d2;
        out.push_back({o1 + v * r1, o2 + v * r2});
    }
    return 1 + (h2 > 0);
}
```