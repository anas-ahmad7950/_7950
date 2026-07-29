THIS TEMPLATE OVERFLOWS FOR $10^9$ NUMBERS
```c++
typedef double T;
using Point = complex<T>;
#define X real()
#define Y imag()
#define PI acos(-1)

T sq(Point p) { return p.X * p.X + p.Y * p.Y; }

Point translate(Point p, Point v) { return p + v; }

Point scale(Point p, T factor, Point o) {
    return o + (p - o) * factor;
}

Point rotate(Point p, T angle, Point o) {
    return o + (p - o) * polar((T)1.0, angle);
}

Point perp(Point p) { return {-p.Y, p.X}; }

T dot(Point v, Point w) { return (conj(v) * w).X; }

T cross(Point v, Point w) { return (conj(v) * w).Y; }

bool isPerp(Point v, Point w) { return dot(v, w) == 0; }

T angle(Point v, Point w) {
    return acos(clamp(dot(v, w) / abs(v) / abs(w), (T)-1.0, (T)1.0));
}

T dist(Point a, Point b) {
    return sqrt((b.X - a.X) * (b.X - a.X) + (b.Y - a.Y) * (b.Y - a.Y));
}

T orient(Point a, Point b, Point c) { return cross(b - a, c - a); }

bool inAngle(Point a, Point b, Point c, Point p) {
    return (orient(a, b, p) * orient(a, c, p) <= 0);
}

T orientedAngle(Point a, Point b, Point c) {
    if (orient(a, b, c) >= 0)
        return angle(b - a, c - a);
    return 2 * PI - angle(b - a, c - a);
}
```

`T sq(Point v)` returns the squared magnitude of a vector $\vec{v}$, more formally, if vector $\vec{v}$ is represented as $(x, y)$, then $|\vec{v}|^2 = x^2 + y^2$.

___

`Point translate(Point p, Point v)` moves a point $p$ by a distance of $|\vec{v}|$ in the direction of $\vec{v}$ (performs vector addition).

___

`Point scale(Point p, T factor, Point o)` scales the distance of point $p$ from a point $o$ by a specific factor.

It shifts the coordinate system by subtracting $o$ from $p$ (i.e. $p_n = p - o$) so that $o$ is at the origin $(0, 0)$, then multiplies $p_n$ by the given factor and finally it adds $o$ back to shift $p_n$ to its original location.

___

`Point rotate(Point p, T angle, Point o)` rotates a point $p$ counter-clockwise around a point $o$ by a specific angle $\theta$ (in radians). If you pass a negative angle ($-\theta$), point $p$ will rotate by $\theta$ clockwise.

Every complex point can be written in polar form using two values: magnitude $r$ and angle $\theta$. let $p_n = r \cdot e^{i \theta}$ (where $p_n = p - o$). By multiplying $p_n$ by another complex number with magnitude of $1$ and angle $\phi$, $p_n$ will rotate by angle $\phi$ counter-clockwise.
$$
(r \cdot e^{i \theta}) \times (1 \cdot e^{i \phi}) = (r \times 1) \cdot e^{i(\theta + \phi)}
$$

___

`Point prep(Point p)` returns a new vector that is perpendicular (rotated $90^{\circ}$ counter-clockwise) to $p$.

___

`T dot(Point v, Point w)` computes the dot product of two vectors $\vec{v}$ and $\vec{w}$.

let $\vec{v} = (x_1, y_1) = x_1 + i y_1$ and $\vec{w} = (x_2, y_2) = x_2 + i y_2$. Then by multiplying the conjugate of $\vec{v}$ by $\vec{w}$, the real part of the resulting complex number will equal to $x_1 x_2 + y_1 y_2$, which is the formula for the dot product.

___

`T cross(Point v, Point w)` computes the 2D cross product of $\vec{v}$ and $\vec{w}$. It represents the signed area of the parallelogram formed by the two vectors.

By multiplying the conjugate of $\vec{v}$ by $\vec{w}$, the imaginary part of the resulting complex number will equal to $x_1 y_2 - y_1 x_2$, which is the formula for the cross product.

___

`bool isPrep(Point v, Point w)` checks of two vectors are exactly perpendicular to one another.

By definition, the dot product of two vectors $\vec{v}$ and $\vec{w}$ is $|\vec{v}| |\vec{w}| \text{cos}(\theta)$. If the angle $\theta$ is $90^{\circ}$ or $270^{\circ}$, $\text{cos}(\theta) = 0$. Therefore, if the dot product is $0$, $\vec{v}$ and $\vec{w}$ are perpendicular.

___

`double angle(Point v, Point w)` returns the shortest angle in radians (between $0$ and $\pi$ inclusive) between two vectors.

It uses the rearranged dot product formula:
$$
\text{cos}(\theta) = \frac{\vec{v} \cdot \vec{w}}{|\vec{v}| |\vec{w}|}
$$

___

`double dist(Point a, Point b)` calculates the euclidean distance between two points $a$ and $b$.

___

`T orient(Point a, Point b, Point c)` determines the side of point $c$ with respect to vector $\vec{ab}$.

It takes the cross product of the vector $\vec{ab} = b - a$ and the vector $\vec{ac} = c - a$.
* If the result is **positive**, then $c$ is to the **left** of the $\vec{ab}$ (counter-clockwise turn, $0^{\circ} < \theta < 180^{\circ}$).
* If the result is **negative**, then $c$ is to the **right** of the $\vec{ab}$ (clockwise turn, $180^{\circ} < \theta < 360^{\circ}$).
* If the result is $0$, then the three points are **collinear** (on the same straight line, $\theta = \{0^{\circ}, 180^{\circ}\}$).

___

`bool inAngle(Point a, Point b, Point c, Point p)` checks if point $p$ falls within the space between lines extending from $a$ to $b$ and $a$ to $c$.

In order for point $p$ to fall between lines $ab$ and $ac$, the orientation of $p$ relative to $ab$ multiplied by its orientation relative to $ac$ must yields a non-positive number ($\le 0$).

___

`T orientedAngle(Point a, Point b, Point c)` returns the counter-clockwise angle (from $0$ to $2 \pi$) going from vector $\vec{ab}$ to vector $\vec{ac}$.

It checks the orientation of $c$ relative to $ab$:
* If $c$ is on the left ($\ge 0$), the shortest angle is already the counter-clockwise angle, so it returns `angle(b - a, c - a)`.
* If $c$ is on the right ($< 0$), the shortest angle goes clockwise. To get the counter-clockwise angle, it subtracts that shortest angle from $2 \pi$.

### Notes:

`polar(r, theta)` returns a complex point (vector) $(x, y)$ with magnitude $r$ and angle $\theta$ with the positive direction of x-axis.

`atan2(opp, adj)` returns the angle $\theta$ made by vector $\vec{v}$ with the positive direction of x-axis between $(-\pi, \pi]$.

`arg(complex<T>)` returns the angle made by vector $\vec{v}$ with the positive direction of x-axis between $(-\pi, \pi]$.

`abs(complex<T>)` return magnitude of vector $\vec{v}$.

If `T` is `int`, functions `arg(complex<T>)` and `abs(complex<T>)` will return `int` values.