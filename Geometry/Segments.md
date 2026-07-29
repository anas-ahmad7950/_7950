# Requires Geometric Functions (and Straight Lines in case of Segment-Point distance)

```c++
//! LINE SEGMENTS
struct Segment {
    point s,e;
    Segment(point a,point b) : s(a),e(b) {}

    // determines whether point p lies inside a disk(circle) whose diameter equals to ab
    // if p lies inside the circle, then the angle between vectors pa and pb >= PI / 2 -> cos(angle) <= 0
    // otherwise, the angle between pa and pb < PI / 2 -> cos(angle) > 0
    // therefore we will use dot product because cos(theta) distinguish between angles < PI / 2 and >= PI / 2
    bool inDisk(point p) { // [integer friendly]
        return dot(s - p, e - p) <= 0;
    }

    // returns (true) if point p lies on segment ab, (false) otherwise
    // point p will lie on segment ab if and only if [a, b and p are collinear -> cross(p-a, b-a) = 0]
    // and [p lies inside a circle with diameter ab -> inDisk(..)]
    bool onSegment(point p) { // [integer friendly]
        return orient(s, e, p) == 0 && inDisk(p);
    }

    // in segments, an inter. is called proper if the inter. point out is [different] from
    // segments endpoints a, b, c and d (i.e. out != [a, b, c or d]), in order for this to happen:-
    // orient[a] with respect to seg. cd must have [different sign] than orient[b] with respect to cd
    // orient[a] * orient[b] = -ve_number, same logic applies to points c and d (with respect to seg. ab)
    bool properInter(Segment s2, point &out) {
        T oa = orient(s2.s, s2.e, s),
        ob = orient(s2.s, s2.e, e),
        oc = orient(s, e, s2.s),
        od = orient(s, e, s2.e);
        // Proper intersection exists iff opposite signs
        if (oa * ob < 0 && oc * od < 0) {
            out = (s * ob - e * oa) / (ob - oa); // weighted average
            return true;
        }
        return false;
    }

    // To create sets of points we need a comparison function
    struct cmpX {
        bool operator() (const point &a,const point &b) const {
            return make_pair(a.X, a.Y) < make_pair(b.X, b.Y);
        }
    };
    // returns a set(distinct values) of intersection points
    // set s will contain 0, 1 or 2 distinct points, describing an empty inter., a single inter. or a seg. inter.
    // if there is no PROPER inter. between ab and cd, we will check each endpoint if it lies on the other segment.
    set<point, cmpX> inters(Segment s2) { // [integer friendly]
        point out;
        if (properInter(s2,out)) return {out};
        set<point, cmpX> st;
        if (s2.onSegment(s)) st.insert(s);
        if (s2.onSegment(e)) st.insert(e);
        if (onSegment(s2.s)) st.insert(s2.s);
        if (onSegment(s2.e)) st.insert(s2.e);
        return st;
    }

    // returns the min. distance from point p to segment ab
    // min. distance = orthogonal distance from point p to ab iff projection of p lies on seg. ab
    // else min.distance = min(distance from p to endpoint a, distance from p to b)
    double segPoint(point p) {
        if (s != e) {
            line l(s, e);
            if (l.cmpProj(s, p) && l.cmpProj(p, e)) // if closest to projection
                return l.dist(p); // return distance to line
            }
        return min(abs(p - s), abs(p - e)); // otherwise return min. distance to a or b
    }

    // returns the min. distance between two segments ab and cd
    // if the two seg. are properly intersected -> min. distance = 0
    // else distance = min(the distance from each endpoint to the other segment)
    double segSeg(Segment s2) {
        point dummy;
        if (properInter(s2, dummy))
            return 0;
        return min({segPoint(s2.s), segPoint(s2.e),
        s2.segPoint(s), s2.segPoint(e)});
    }
    
    // moves the starting point across the line segment with t representing the percentage
    // t = distToMove / abs(Direction Vector)
    point linearInterpolation(double t) {
        return s + t*(e - s);
    }
};
```