REQUIRES GEOMETRIC FUNCTIONS (`sq`, `perp`, `dot`, `cross`)
```c++
struct line {
    // Fields
    // Directional vector v = (b, -a)
    // b = delta[x] = x2 - x1, -a = delta[y] = y2 - y1 -> +a = -delta[y] = y1 - y2
    point v;
    // Position Constant c = crossProduct(v, p) : p is a point on the line
    T c;

    // Constructors
    // From direction vector v and offset c
    line(point v, T c) : v(v), c(c) {}
    // From standard form equation ax + by = c
    // If the standard form equation is ax + by + c = 0, then we will assign the Position Constant c as -c
    line(T a, T b, T c) : v(b,-a), c(c) {}
    // From points p and q that belongs to the line
    line(point p, point q) : v(q-p), c(cross(v,p)) {}

    //! List of Methods
    //! these work with T = int
    // T side(point p);
    // double dist(point p);
    // line perpThrough(point p);
    // bool cmpProj(point p, point q);
    // line translate(point t);
    //! these require T = double
    // void shiftLeft(double dist);
    // point proj(point p);
    // point refl(point p);
    // line bisector(line l1, line l2, bool interior);

    // returns the orientation of a point with respect to the line [integer friendly]
    // +ve -> point p is on the [left] side of the line
    // -ve -> point p is one the [right] side of the line
    // zero -> point p is on the the line
    T side(point p) {return cross(v,p)-c;}

    // returns the distance between point p and the line
    // side(p) = cross(v, p) - c = cross(v, p) - cross(v, l) such that l is a point on the line
    // = ||v|| ||p|| sin(theta1) - ||v|| ||l|| sin(theta2) [take ||v|| as a common factor]
    // = ||v|| (||p|| sin(theta1) - ||l|| sin(theta2)) <- this is the distance
    T dist(point p) {return abs(side(p)) / abs(v);}

    // same as dist(p) * dist(p)
    T sqDist(point p) {return side(p) * side(p) / (T)sq(v);}

    // returns a line perpendicular to this line and passes through the point p
    // p + prep(v) can be considered as translate(p, perp(v))
    line perpThrough(point p) {return {p, p + perp(v)};}

    // sorting comparator for points [or their projections] on a line
    // as point p become far from vector v, the value of abs(||p||cos(theta)) will increase
    // returns (true) if p is before q with respect to Direction Vector v, (false) otherwise
    bool cmpProj(point p, point q) {
        return dot(v,p) < dot(v,q);
    }

    // returns a translated line by vector t [magnitude ||t||, in t direction]
    // line before translation: c = cross(v, p) : p is a point on the line
    // after translation, point p becomes p + t so, c' = cross(v, p + t)
    // c' = cross(v, p) + cross(v, t) = c + cross(v, t)
    // the Directional Vector v will remain the same
    line translate(point t) {return {v, c + cross(v,t)};}

    // return a line translated orthogonally by distance +d[left]/-d[right]
    // we can use the same logic of line translation, we want to translate the line by vector t
    // t = d/||perp(v)|| * perp(v) [scalar multiplication]
    // c' = c + cross(v, t) = c + d/||perp(v)|| * cross(v, perp(v)) [||perp(v)|| = ||v||]
    // c' = c + d/||v|| * ||v||^2 * sin(90) -> c' = c + d * ||v||, and v will remain the same
    line shiftLeft(double distance) {return {v, c + distance * abs(v)};}

    // to find the projection of point p on straight line l, we have to move the point orthogonally [in dir of perp(v)]
    // with magnitude k * ||perp(v)|| so that side(p') = 0 : p' is the projection of point p
    // side(p') = side(p + k * perp(v)) = 0 -> cross(v, p + k * perp(v)) - c
    // -> (cross(v, p) - c) + k * cross(v, perp(v)) = 0 -> side(p) + k * ||v||^2 = 0
    // -> factor (k) = -side(p) / ||v||^2 [we can substitute with k in p + k * perp(v)]
    point proj(point p) {return p - perp(v) * side(p) / sq(v);}

    // to find the reflection of p by line l, we need to move point p in the same dir but twice the distance
    // same logic as function proj(..) but we move p twice the distance
    point refl(point p) {
        T s = 2;
        return p - s * perp(v) * side(p) / sq(v);
    }    
};

// the Directional Vector (v) of a bisector = l1.v / ||l1.v|| (sign) l2.v / ||l2.v||
// the Position Constant (c) of a bisector = l1.c / ||l1.v|| (sign) l2.c / ||l2.v||
// (sign) -> +ve in case of Internal Bisector, -ve in case of External Bisector
// [[NOT] INTEGER FRIENDLY]
line bisector(line l1, line l2, bool interior) {
    assert(cross(l1.v, l2.v) != 0); // l1 and l2 cannot be parallel
    T sign = interior ? 1 : -1;
    return {l2.v / abs(l2.v) + l1.v / abs(l1.v) * sign,
    l2.c / abs(l2.v) + l1.c / abs(l1.v) * sign};
}

// returns (true, assign the inter. pt. to [out]) if there is a proper inter. between lines l1 and l2
// returns (false) in case of no inter. between l1 and l2 or improper inter.
// in case of improper inter., l1 and l2 will be parallel (crossProduct = 0) and l1.c = l2.c
bool inter(line l1, line l2, point &out) {
    T d = cross(l1.v, l2.v);
    if (d == 0) return false;
    out = (l2.v * l1.c - l1.v * l2.c) / d; // requires floating-point coordinates
    return true;
}
```

A straight line can be represented as a standard algebraic equation $ax + by = c$, where $a = -\Delta y = y_1 - y_2$ and $b = \Delta x = x_2 - x_1$. Therefore, we can represent any straight line using two fields: a directional vector $\vec{v} = (b, -a)$ and a positional constant $c$. Also,  a straight line can be represented by two points $p$ and $q$ that belong to that line ($\vec{v} = \vec{pq} = q - p$ and $c = \vec{v} \times p = (b, -a) \times (x, y) = by - (-ax) = ax + by$).

`T side(Point p)` returns the orientation of point $p$ relative to the directional vector of the straight line $\vec{v}$.
* If $p$ is on the line, $\vec{v} \times p = c$, so it returns $0$.
* If the result is **positive**, $p$ is on the **left** of the line (relative to the direction of $\vec{v}$).
* If the result is **negative**, $p$ is on the **right** side of the line.

___

`T dist(Point p)` returns the shortest distance $l_p$ from point $p$ to the line.

let $x$ be a point on the line, then:
$$
\begin{aligned}
\text{side}(p) = \vec{v} \times p - c = \vec{v} \times p - \vec{v} \times x
\\
= |\vec{v}| \ |p| \ \text{sin}(\theta_1) - |\vec{v}| \ |x| \ \text{sin}(\theta_2)
\\
= |\vec{v}| \cdot (|p| \ \text{sin}(\theta_1) - |x| \ \text{sin}(\theta_2))
\\
= |\vec{v}| \cdot l_p
\end{aligned}
$$

___

`T sqDist(Point p)` returns the squared distance $(l_p)^2$ (same as $(\text{dist}(p))^2$).

___

`Point proj(Point p)` Finds the projection of point $p$ onto the line.

To find the projection of point $p$ onto the line, we have to move that point orthogonally (in the opposite direction of $\text{prep}(\vec{v})$, that why $k$ has a negative value) with magnitude equals to $k \cdot |\text{prep}(\vec{v})|$ so that $\text{side}(p') = 0$, where $p'$ is the projection of point $p$ onto the line.
$$
\begin{aligned}
\text{side}(p') = side(p + k \cdot \text{prep}(\vec{v})) = 0
\\
\vec{v} \times (p + k \cdot \text{prep}(\vec{v})) - c = 0
\\
(\vec{v} \times p - c) + k \cdot (\vec{v} \times \text{prep}(\vec{v})) = 0
\\
\text{side}(p) + k \cdot |\vec{v}|^2 = 0
\\
k = -\frac{\text{side}(p)}{|\vec{v}|^2}
\end{aligned}
$$
Note: $\vec{v} \times \text{prep}(\vec{v}) = |\vec{v}| \ |\vec{v}| \ \text{sin}(90^{\circ}) = |\vec{v}|^2$.

___

`Point refl(Point p)` reflects point $p$ across the line. It uses the same logic as `proj(p)`, but multiplies $k$ by $2$.

___

`bool cmpProj(Point p, Point q)` A comparator used for sorting points (or their orthogonal projection) on the line.

We can use the dot product to figure out the order of two points: a point $p$ (or its orthogonal projection) comes before a point $q$ (or its orthogonal projection) if $\vec{v} \cdot p < \vec{v} \cdot q$.
$$
\begin{aligned}
\vec{v} \cdot p < \vec{v} \cdot q
\\
|\vec{v}| \ |p| \ \text{sin}(\theta_1) < |\vec{v}| \ |q| \ \text{sin}(\theta_2)
\\
|p| \ \text{sin}(\theta_1) < |q| \ \text{sin}(\theta_2)
\\
\text{adj}_p < \text{adj}_q
\end{aligned}
$$

___

`Line translate(Point t)` shifts the entire line in the 2D plane by distance $|\vec{t}|$ in the direction of $\vec{t}$.

If we want to translate a line by vector $\vec{t}$, the direction vector $\vec{v}$ remains the same but we have to update the positional constant $c$:
$$
c' = \vec{v} \times (p + \vec{t}) = \vec{v} \times p + \vec{v} \times \vec{t} = c + \vec{v} \times \vec{t}
$$
where $p$ denotes a point on the original line.

___

