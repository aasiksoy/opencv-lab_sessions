# Lab 3, Section 1: from a mask to objects (contours + rotated boxes)

## Theory

**Binary mask:** an image with only two values, 255 (white) = "this pixel belongs to the object" and 0 (black) = "it doesn't". It's a yes/no map, made for example by color thresholding in HSV (Lab 1).

**Contour:** the outline of one white blob in the mask, stored as a list of border points. One blob = one contour, so **counting objects = counting contours**.
- Only the **outer** outline is kept (holes inside a blob are ignored).
- On straight edges only the **end points** are stored, to save memory.

**Noise filtering by area:** tiny specks in the mask also become contours. Each contour has an **area** (number of pixels inside). Keep only contours above a threshold.
- How to choose the threshold: list all areas from big to small, find the **big gap** between real objects and noise, and put the threshold in the **middle** of the gap.
- Strawberries: objects 388 to 2072 px, noise at most 9 px, so anything between about 10 and 380 works (we used 100).

**Rotated (minimum-area) bounding box:** the tightest rectangle around an object, allowed to turn with it.
- A **straight** box (horizontal/vertical sides) wastes space around tilted objects and gives the wrong length/width.
- The rotated box is found by: (1) wrapping the outline in its **convex hull** (a "rubber band" that bridges the dents), (2) trying orientations and keeping the one with the **smallest area**.
- Result: **center** (where), **width and height** (how big), **angle** (which way it points). Just 5 numbers per object.
- To **draw** it, convert it to its **4 corner points** and connect them.
- For **round** objects the angle is meaningless (every rotation gives the same square).

**Object area:** the area **inside the outline** is the object's real size. The box's area (width × height) is bigger, because it includes the empty corners.

## The flow (what to do, in order)

1. **Make a binary mask** of the objects (e.g. HSV color thresholding; for red, use two hue ranges and combine them with OR).
2. **Clean the mask if needed:** opening removes white specks, closing fills gaps and joins split pieces (Lab 1 morphology).
3. **Find the contours:** one outline per white blob.
4. **Filter by area:** throw away contours smaller than the threshold (noise).
5. **Count:** number of objects = number of remaining contours.
6. **Measure:** area of each contour = object size; largest area = biggest object.
7. **Rotated box per contour:** gives center, length, width and angle.
8. **Draw:** convert the boxes to 4 corners and draw them on a **copy** of the original image.

## What can go wrong

- **Touching objects** merge into one blob, so they're counted as one and the area is too big. Fix: a better mask, opening, or watershed.
- **One object split into pieces** (shadow, thin part) is counted twice. Fix: closing.
- **No clear area gap** between noise and objects: improve the segmentation first.
- **Drawing on the original image** changes it. Always draw on a copy.

## Example case

*"Count and measure the bananas on a conveyor belt, and find which way each one points."*
HSV threshold on yellow → clean the mask (opening/closing) → contours → filter by area → count → rotated box per banana gives its center (for picking), its length (size) and its angle (orientation).


# Lab 3, Section 2: finding circles (Hough Circle Transform)

## Theory

**Why not contours for round objects?** Glare, shadows and touching objects make the mask messy, so the outline isn't round anymore. If you **know** the objects are circles, it's more robust to search for circles directly.

**A circle = 3 numbers:** center (x, y) and radius r. Finding circles = finding the (x, y, r) values that fit the edges. ("Parametric" shape detection: search for the parameters of a known shape.)

**Voting idea:**
1. Find the **edge pixels** (where brightness changes sharply).
2. At each edge pixel, the **gradient** points straight across the edge. On a circle that is along the **radius**, so it points toward the center.
3. Each edge pixel **votes** along its gradient line. A grid of vote counters (the **accumulator**) counts how many lines pass through each point.
4. At a real center, the lines from the whole border **cross**, so it gets **many votes**. Points with enough votes become circle centers.
5. The **radius** = the most common distance from that center to the edge pixels that voted for it.

**Why it's robust:** you don't need the whole circle. A coin that's half-covered or has glare still gets votes in the right place from its visible border (fewer votes, but the same center). Like RANSAC: the answer with the most support wins.

**Limits:** only finds **circles**. On a square, the gradients along one side are parallel, so they never cross in one point, and there's no single radius.

**Grayscale first:** edges are brightness changes, so color isn't needed. (Rare trap: two colors with the same brightness give no edge.)

**Blur first (most important step):** textures inside or around objects (coin faces, letters, scratches, wood grain) create tiny edges that also vote, so you get **fake circles** and wrong radii ("ghost votes"). Blurring erases small details and keeps the big outer borders. Use a **big** kernel (e.g. 11×11 to 21×21).
- too little blur → fake circles; too much → real borders fade, circles get missed
- coins experiment: no blur gave 23 circles (15 fake); with blur, the correct 8. With strong blur the result stays correct even when other settings change.

**The parameters, in words:**
- **resolution of the vote grid (dp):** almost always 1
- **minimum distance between centers:** too small → several circles on one object; too big → neighbors get missed. Set it to about the object radius if objects don't overlap.
- **edge strength threshold (param1):** decides **which pixels may vote**. Higher → only strong edges.
- **vote threshold (param2), the main knob:** decides **how many votes make a circle**. Lower → finds more (and more fakes); higher → only clear circles.
- **min / max radius:** the size range to search. If you know the object size, use it: faster and fewer mistakes.

**Output:** a list of circles, each [x, y, r], with decimal precision (round before drawing). If nothing is found, you get **nothing (None)**, so always check before using it.

## The flow (what to do, in order)

1. **Load** the image (keep a color copy for drawing).
2. **Convert to grayscale.**
3. **Blur** with a big kernel, to remove texture.
4. **Run the circle voting** with sensible settings: radius range from what you know about the objects, minimum distance about one radius, then tune the vote threshold.
5. **Check that circles were found** (handle the "nothing found" case).
6. **Use the results:** count = number of circles, size = radius, position = center.
7. **Draw** the circles on a copy and **check by eye**: fake circles → more blur or a stricter vote threshold; missed circles → a lower vote threshold or less blur; doubles on one object → a bigger minimum distance.

## What can go wrong → fix

- **fake circles in the background texture** → more blur, higher vote threshold, higher edge threshold, tighter radius range
- **several circles on one object** → bigger minimum distance between centers
- **real circles missed** → lower vote threshold, less blur, check the radius range
- **objects seen at an angle** look like **ellipses**, not circles → this method struggles
- **non-round objects** → use contours instead (Section 1)

## Example case

*"Count the coins on a table and estimate their value from their size."*
Grayscale → strong blur (the table texture would create fake circles) → circle voting with a radius range around the expected coin sizes and a minimum distance of about one radius → count = number of circles → the radius tells you which type of coin it is (bigger radius = bigger coin).
