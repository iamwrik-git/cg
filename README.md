# Computer Graphics Lab (MJC-10) — All 7 Programs

## 1. Bresenham's Line Drawing Algorithm
```python
import matplotlib.pyplot as plt

def bresenham_line(x0, y0, x1, y1):
    x, y = x0, y0
    xs, ys = [x], [y]

    dx = abs(x1 - x0)
    dy = abs(y1 - y0)

    if dx > dy:
        p = 2 * dy - dx
        for _ in range(dx):
            x += 1
            if p < 0:
                p = p + 2 * dy
            else:
                y += 1
                p = p + 2 * (dy - dx)
            xs.append(x)
            ys.append(y)
    else:
        p = 2 * dx - dy
        for _ in range(dy):
            y += 1
            if p < 0:
                p = p + 2 * dx
            else:
                x += 1
                p = p + 2 * (dx - dy)
            xs.append(x)
            ys.append(y)

    pixels = list(zip(xs, ys))
    print("Plotted Pixels:", pixels)

    plt.scatter(xs, ys, marker='s', s=150)
    plt.gca().set_aspect('equal')
    plt.title(f"Bresenham Line ({x0}, {y0}) to ({x1}, {y1})")
    plt.grid(True)
    plt.show()

bresenham_line(2, 2, 18, 10)
```

---

## 2. Mid-Point Circle Drawing Algorithm
```python
import matplotlib.pyplot as plt

def plot_octants(xc, yc, x, y, x_coords, y_coords):
    octants = [
        (xc + x, yc + y), (xc - x, yc + y),
        (xc + x, yc - y), (xc - x, yc - y),
        (xc + y, yc + x), (xc - y, yc + x),
        (xc + y, yc - x), (xc - y, yc - x)
    ]
    for px, py in octants:
        x_coords.append(px)
        y_coords.append(py)

def midpoint_circle(xc, yc, r):
    x, y = 0, r
    decision = 1 - r
    x_coords, y_coords = [], []

    while x <= y:
        plot_octants(xc, yc, x, y, x_coords, y_coords)

        if decision < 0:
            decision += 2 * x + 3
        else:
            decision += 2 * (x - y) + 5
            y -= 1

        x += 1

    pixels = list(zip(x_coords, y_coords))
    print("Circle Pixels:", pixels)

    plt.scatter(x_coords, y_coords)
    plt.gca().set_aspect('equal')
    plt.title("Midpoint Circle Algorithm")
    plt.grid(True)
    plt.show()

midpoint_circle(0, 0, 10)
```

---

## 3. Cohen–Sutherland Line Clipping Algorithm
```python
import matplotlib.pyplot as plt

INSIDE, LEFT, RIGHT, BOTTOM, TOP = 0, 1, 2, 4, 8

def get_code(x, y, xmin, ymin, xmax, ymax):
    code = INSIDE
    if x < xmin:   code |= LEFT
    elif x > xmax: code |= RIGHT
    if y < ymin:   code |= BOTTOM
    elif y > ymax: code |= TOP
    return code

def cohen_sutherland(x1, y1, x2, y2, xmin, ymin, xmax, ymax):
    c1 = get_code(x1, y1, xmin, ymin, xmax, ymax)
    c2 = get_code(x2, y2, xmin, ymin, xmax, ymax)

    while True:
        if c1 == 0 and c2 == 0:
            return (round(x1, 2), round(y1, 2)), (round(x2, 2), round(y2, 2))
        elif (c1 & c2) != 0:
            return None
        else:
            code = c1 if c1 != 0 else c2
            if code & TOP:
                x = x1 + (x2 - x1) * (ymax - y1) / (y2 - y1)
                y = ymax
            elif code & BOTTOM:
                x = x1 + (x2 - x1) * (ymin - y1) / (y2 - y1)
                y = ymin
            elif code & RIGHT:
                y = y1 + (y2 - y1) * (xmax - x1) / (x2 - x1)
                x = xmax
            elif code & LEFT:
                y = y1 + (y2 - y1) * (xmin - x1) / (x2 - x1)
                x = xmin

            if code == c1:
                x1, y1 = x, y
                c1 = get_code(x1, y1, xmin, ymin, xmax, ymax)
            else:
                x2, y2 = x, y
                c2 = get_code(x2, y2, xmin, ymin, xmax, ymax)

win = (0, 0, 10, 10)
p1, p2 = (-2, 2), (12, 9)
clipped = cohen_sutherland(*p1, *p2, *win)

print("Clipped Line:", clipped)

plt.plot([0, 10, 10, 0, 0], [0, 0, 10, 10, 0], 'k-', label="Clip Window")
plt.plot([p1[0], p2[0]], [p1[1], p2[1]], 'r--', label="Original Line")
if clipped:
    plt.plot([clipped[0][0], clipped[1][0]], [clipped[0][1], clipped[1][1]], 'g-', linewidth=2.5, label="Clipped Line")
plt.legend()
plt.grid(True)
plt.show()
```

---

## 4. Sutherland–Hodgman Polygon Clipping Algorithm
```python
import matplotlib.pyplot as plt

def clip_poly(poly, xmin, ymin, xmax, ymax):
    edges = [
        (lambda p: p[0] >= xmin, lambda p1, p2: (xmin, p1[1] + (p2[1] - p1[1]) * (xmin - p1[0]) / (p2[0] - p1[0]))),
        (lambda p: p[0] <= xmax, lambda p1, p2: (xmax, p1[1] + (p2[1] - p1[1]) * (xmax - p1[0]) / (p2[0] - p1[0]))),
        (lambda p: p[1] >= ymin, lambda p1, p2: (p1[0] + (p2[0] - p1[0]) * (ymin - p1[1]) / (p2[1] - p1[1]), ymin)),
        (lambda p: p[1] <= ymax, lambda p1, p2: (p1[0] + (p2[0] - p1[0]) * (ymax - p1[1]) / (p2[1] - p1[1]), ymax)),
    ]
    for inside, intersect in edges:
        out = []
        for i in range(len(poly)):
            curr, prev = poly[i], poly[i - 1]
            if inside(curr):
                if not inside(prev): out.append(intersect(prev, curr))
                out.append(curr)
            elif inside(prev):
                out.append(intersect(prev, curr))
        poly = out
    return [(round(x, 2), round(y, 2)) for x, y in poly]

poly = [(-2, 5), (5, 12), (12, 5), (5, -2)]
clipped = clip_poly(poly, 0, 0, 10, 10)
print("Clipped Polygon:", clipped)

plt.plot([0, 10, 10, 0, 0], [0, 0, 10, 10, 0], 'k-', label="Window")
plt.plot(*zip(*(poly + [poly[0]])), 'r--', label="Original")
plt.plot(*zip(*(clipped + [clipped[0]])), 'g-', linewidth=2, label="Clipped")
plt.legend()
plt.grid(True)
plt.show()
```

---

## 5. 2D Homogeneous Transformations
```python
import numpy as np
import matplotlib.pyplot as plt

tri = np.array([[1, 4, 2],
                [1, 1, 4],
                [1, 1, 1]])
a = np.radians(45)

T = np.array([[1, 0, 3], [0, 1, 2], [0, 0, 1]])
S = np.array([[2, 0, 0], [0, 2, 0], [0, 0, 1]])
R = np.array([[np.cos(a), -np.sin(a), 0],
              [np.sin(a),  np.cos(a), 0], [0, 0, 1]])

shapes = {"Original": tri, "Translated": T @ tri, "Scaled": S @ tri, "Rotated": R @ tri}

for name, obj in shapes.items():
    print(name, ":\n", np.round(obj[:2].T, 2))
    closed = obj[:, [0, 1, 2, 0]]
    plt.plot(closed[0], closed[1], marker='o', label=name)

plt.legend()
plt.grid(True)
plt.show()
```

---

## 6. 3D Transformations & Perspective Projection
```python
import numpy as np
import matplotlib.pyplot as plt

pyr = np.array([[0, 2, 0, 1],
                [0, 0, 2, 1],
                [1, 1, 1, 4],
                [1, 1, 1, 1]])

T = np.array([[1, 0, 0, 2], [0, 1, 0, 1], [0, 0, 1, 2], [0, 0, 0, 1]])
S = np.array([[2, 0, 0, 0], [0, 2, 0, 0], [0, 0, 2, 0], [0, 0, 0, 1]])

d = 5
P = np.array([[1, 0, 0, 0], [0, 1, 0, 0], [0, 0, 1, 0], [0, 0, 1/d, 0]])

orig_2d = P @ pyr
orig_2d = orig_2d / orig_2d[3]

obj_3d  = T @ S @ pyr
proj_2d = P @ obj_3d
proj_2d = proj_2d / proj_2d[3]

print("Transformed 3D:\n", np.round(obj_3d[:3].T, 2))
print("Projected 2D:\n",   np.round(proj_2d[:2].T, 2))

idx = [0, 1, 2, 0, 3, 1, 3, 2]
plt.plot(orig_2d[0, idx], orig_2d[1, idx], 'b--', label="Original (Projected)")
plt.plot(proj_2d[0, idx], proj_2d[1, idx], 'ro-', label="Transformed (Projected)")
plt.legend()
plt.grid(True)
plt.show()
```

---

## 7. Cubic Bézier Curve
```python
import matplotlib.pyplot as plt

def bezier_curve(p0, p1, p2, p3):
    xs, ys = [], []

    for step in range(101):
        t = step / 100.0

        b0 = (1 - t)**3
        b1 = 3 * t * (1 - t)**2
        b2 = 3 * t**2 * (1 - t)
        b3 = t**3

        x = b0 * p0[0] + b1 * p1[0] + b2 * p2[0] + b3 * p3[0]
        y = b0 * p0[1] + b1 * p1[1] + b2 * p2[1] + b3 * p3[1]

        xs.append(round(x, 2))
        ys.append(round(y, 2))

    curve_points = list(zip(xs, ys))
    print("Bezier Curve Points (First 10):", curve_points[:10])
    print("Total Points Generated:", len(curve_points))

    plt.plot(xs, ys, color='red', linewidth=2, label="Bézier Curve")
    plt.scatter(*zip(p0, p1, p2, p3), color='blue', label="Control Points")
    plt.title("Cubic Bézier Curve")
    plt.legend()
    plt.grid(True)
    plt.show()

bezier_curve((0, 0), (1, 3), (3, 3), (4, 0))