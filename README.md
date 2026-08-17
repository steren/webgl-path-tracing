# WebGL Path Tracing

![Screenshot](image.png)

[Try it live](https://webgl-path-tracing.steren.fr)

Path tracing is a realistic lighting algorithm that simulates light bouncing around a scene. This path tracer uses WebGL for realtime performance and supports diffuse, mirrored, and glossy surfaces. The path tracer is continually rendering, so the scene will start off grainy and become smoother over time.

The entire scene is dynamically compiled into a GLSL shader. Everything can be repositioned using the current shader, but any geometry or material change means a recompilation. To calculate a pixel color, a ray is shot into the scene and allowed to bounce around five times. At each bounce, the direct light incoming at that point (including shadows) is multiplied by all previous material colors and accumulated. Soft shadows come from jittering the light position per pixel at every bounce.

Rather than drawing those jitters independently at random, each pixel walks a
scrambled Halton sequence, which spreads its samples out evenly instead of
letting them clump and leave gaps. That takes roughly two to three times fewer
samples to reach the same amount of grain.

The renderer also accumulates as many samples per animation frame as fit in a
frame's worth of time, instead of exactly one. A scene that costs a fraction of
a frame would otherwise spend the rest of every frame waiting for the next
vsync, capped at the refresh rate no matter how fast the hardware is. The count
is measured and adjusted continuously, and settles back at one sample per frame
on hardware that a single sample already saturates. Pass `samplesPerFrame` in
the config to pin it instead.

Samples accumulate into a floating point buffer where the browser can render to
one, so the image keeps converging instead of settling at the precision of an
8 bit buffer. The canvas can be any size, square or not.

## Shapes

Three shapes are available:

- `new Sphere(center, radius, id, color)`
- `new Cube(minCorner, maxCorner, id, color)`
- `new ExtrudedRectangle(minCorner, maxCorner, borderRadius, id, color, axis)`

An extruded rectangle is a box whose cross section is a rectangle with rounded
corners, like a CSS `border-radius` extruded into a solid: the four edges
running along `axis` (`'x'`, `'y'` or `'z'`, defaults to `'z'`) are rounded off
by `borderRadius` and the two ends stay flat. The radius is clamped to half of
the smaller side of the cross section, and a radius of 0 gives a plain box.

`color` is optional and defaults to a neutral grey. `id` has to be unique within
a scene, since it is used to name the shape's uniforms in the generated shader.

## Use the module

Install the module with `npm install webgl-path-tracing`

Use your favorite bundler or add to your page with:

```html
<script type="importmap">
	{
		"imports": {
			"webgl-path-tracing": "./node_modules/webgl-path-tracing/webgl-path-tracing.js",
			"sylvester": "./node_modules/webgl-path-tracing/sylvester.src.js"
		}
	}
</script>

<script type="module">
	import {makePathTracer, Cube, Sphere, ExtrudedRectangle} from 'webgl-path-tracing';
	import {Vector} from 'sylvester';

	let objects = [];
	let nextObjectId = 0;
	objects.push(new Cube(Vector.create([-0.25, -1, -0.25]), Vector.create([0.25, -0.75, 0.25]), nextObjectId++));
  objects.push(new Sphere(Vector.create([0, -0.75, 0]), 0.25, nextObjectId++)); 
  objects.push(new ExtrudedRectangle(Vector.create([-0.5, -0.5, -0.1]), Vector.create([0.5, 0, 0.1]), 0.15, nextObjectId++));

	window.onload = function() {
		makePathTracer(document.getElementById('canvas'), objects);
	}
</script>
<canvas id="canvas" width="512" height="512"></canvas>
```
