![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/mandelbrot/gif.gif)

It is the famous **Mandelbrot set**, a collection of points on the complex plane that form a fractal. It is the most famous fractal in the fractal theory proposed by the mathematician Benoit B. Mandelbrot.

The wonder of this set lies in the fact that when you infinitely magnify the Mandelbrot set, exquisite details emerge within it, all generated by a simple formula. Therefore, some consider the Mandelbrot set to be "the most peculiar and magnificent geometric shape ever made by mankind," often referred to as "**God's fingerprint**”.

Today, we will share what fractal theory is, how to draw Mandelbrot fractals using MoonBit, and discover the beauty of mathematics with MoonBit. [MoonBit](https://www.moonbitlang.com/) is a Rust-like programming language and toolchain optimized for WebAssembly, great for writing high-performance code. Let's take the classic Mandelbrot set algorithm and implement in MoonBit.

# **What is Fractal Theory**

First, let's look at what fractal theory is.

Fractal theory was created by Mandelbrot in 1975, stemming from the Latin word "**fractus**", meaning "broken" or "fractured." **The mathematical foundation of fractal theory** is fractal geometry, which describes and studies objective things from the perspective of fractional dimensions and mathematical methods.

Because of this, fractals transcend the dimensions of our conventional world, allowing for a more concrete and realistic description of complex systems, revealing the complexity and diversity of objective things.

Due to the "infinite complexity" of fractals, you might think that creating fractals is difficult, but it's a very simple process. To create a fractal, you just need to repeat the same process over and over again. In mathematical terms, **a mathematical fractal is an iterative (a form of recursion) equation**.

The most famous fractal is the Mandelbrot set, which comes from the complex number set c. Mathematician Adrien Douady defined the following function:
$$f_{c}(z) = z^2 + c$$

In homage to Mandelbrot, it is named the **Mandelbrot set**. When iterated from z=0, it does not diverge to infinity. Essentially, it is an iterative formula, where the variables in the equation are complex numbers. So when you calculate by substituting according to this formula, **local patterns resemble the overall structure**, and this similarity often concentrates on subtle details, requiring careful observation to discern.

# **How to Draw the Mandelbrot Set with MoonBit**

Next, we will share how to draw the Mandelbrot set using MoonBit.

To determine the area of the graphic we want to draw, we must first introduce the concept of **region coordinates**. A point on the complex plane is represented by a complex number (d=x+yi). Adding width and height determines a rectangular area on the complex plane.

Suppose an image has a width of w pixels and a height of h pixels. We need to calculate the colors of w*h pixels and then draw them.

We use MoonBit to perform the color calculation part, and then pass the calculated colors to JavaScript. We use JavaScript canvas to draw the image.

## **Color Calculation**

```rust
pub func calc_color(col : Int, row : Int, ox : Float64, oy : Float64,
    width : Float64) -> Int {
let pixel_size = width / image_width
let cx = (float_of_int(col) - coffset) * pixel_size + ox
let cy = (float_of_int(row) - roffset) * pixel_size + oy
var r = 0
var g = 0
var b = 0
var i = -1
while i <= 1 {
var j = -1
while j <= 1 {
  let d = iter(
    cx + float_of_int(i) * pixel_size / 3.0,
    cy + float_of_int(j) * pixel_size / 3.0,
  )
  let c = get_color(d)
  r = r + c.asr(16).land(0xFF)
  g = g + c.asr(8).land(0xFF)
  b = b + c.land(0xFF)
  j = j + 1
}
i = i + 1
}
r = r / 9
g = g / 9
b = b / 9
return r.lsl(16).lor(g.lsl(8)).lor(b)
}
```

calculate the coordinates of the center point of the square represented on the complex plane by the pixel at row and col.

```rust
let pixel_size = width / image_width
  let cx = (float_of_int(col) - coffset) * pixel_size + ox
  let cy = (float_of_int(row) - roffset) * pixel_size + oy
```

We know that for a complex number c, it belongs to the Mandelbrot set if and only if the infinite sequence of complex numbers obtained by the following recursive definition remains within a circle in the complex plane centered at the origin with a radius of 2: $z_0 = 0$; $z_n=z^2_{n-1} + c$; If we express $z_k$ as $x_k + y_{k}i$ with its real and imaginary parts separated, and similarly express $c$ as $c_x + c_{y}i$ (with its real and imaginary parts denoted as real $c_x$ and imag $c_y$ respectively), then the recursive definition above is essentially stating $x_0 = 0$, $y_0 = 0$; $x_n = x^2_{n-1} - y^2_{n-1} + c_x$, $y_n = 2x_{n-1}y_{n-1}+c_y$; a complex number $c_x +c_{y}i$ belongs to the Mandelbrot set if and only if for all natural numbers 'n', $x^2_n + y^2_n <2^2 = 4$.

`calc_color` then calls `iter` to calculate x_n and y_n. This function returns the number of iterations at which the sequence escapes for the first time, or -1.0 if it does not escape after max_iter_number iterations.

```rust
pub func iter(cx : Float64, cy : Float64) -> Float64 {
    var x = 0.0
    var y = 0.0
    var newx = 0.0
    var newy = 0.0
    var smodz = 0.0
    var i = 0
    while i < max_iter_number {
      newx = x * x - y * y + cx
      newy = 2.0 * x * y + cy
      x = newx
      y = newy
      i = i + 1
      smodz = x * x + y * y
      if smodz >= escape_radius {
        return float_of_int(i) + 1.0 - log(log(smodz) * 0.5) / log(2.0)
      }
    }
    return -1.0
  }
```

Next, we need to choose the appropriate color based on the returned number of  `interpolation`. What we need first is a color palette, and this is where interpolation comes into play.  `interpolation` is used to generate a gradient of colors.

```rust
func interpolation(f : Float64, c0 : Int, c1 : Int) -> Int {
    let r0 = c0.asr(16).land(0xFF)
    let g0 = c0.asr(8).land(0xFF)
    let b0 = c0.land(0xFF)
    let r1 = c1.asr(16).land(0xFF)
    let g1 = c1.asr(8).land(0xFF)
    let b1 = c1.land(0xFF)
    let r = floor((1.0 - f) * float_of_int(r0) + f * float_of_int(r1) + 0.5)
    let g = floor((1.0 - f) * float_of_int(g0) + f * float_of_int(g1) + 0.5)
    let b = floor((1.0 - f) * float_of_int(b0) + f * float_of_int(b1) + 0.5)
    return r.lsl(16).lor(g.lsl(8).lor(b))
  }
```

`get_color` first performs some transformation on the number of iterations and then passes it to `interpolation` to obtain the corresponding color.

```rust
pub func get_color(d : Float64) -> Int {
    if d >= 0.0 {
      var k = 0.021 * (d - 1.0 + log(log(128.0)) / log(2.0))
      k = log(1.0 + k) - 29.0 / 400.0
      k = k - float_of_int(floor(k))
      k = k * 400.0
      if k < 63.0 {
        return interpolation(k / 63.0, 0x000764, 0x206BCB)
      } else if k < 167.0 {
        return interpolation((k - 63.0) / (167.0 - 63.0), 0x206BCB, 0xEDFFFF)
      } else if k < 256.0 {
        return interpolation((k - 167.0) / (256.0 - 167.0), 0xEDFFFF, 0xFFAA00)
      } else if k < 342.0 {
        return interpolation((k - 256.0) / (342.0 - 256.0), 0xFFAA00, 0x310230)
      } else {
        return interpolation((k - 342.0) / (400.0 - 342.0), 0x310230, 0x000764)
      }
    } else {
      return 0x000000
    }
  }
```

Color calculation is now completed.

## **Paint with canvas**

Create a canvas:

```rust
<html>
<body>
  <canvas id="canvas"></canvas>
</body>
```

In the JavaScript code, obtain the canvas and set its size:

```rust
let canvas = document.getElementById("canvas");
var IMAGEWIDTH = 800;
var IMAGEHEIGHT = 600;
canvas.width = IMAGEWIDTH;
canvas.height = IMAGEHEIGHT;
```

Create an `ImageData` object to store the computed colors of the pixels:

```rust
var imagedata = context.createImageData(IMAGEWIDTH, IMAGEHEIGHT);
```

Then import the MoonBit code:

```rust
WebAssembly.instantiateStreaming(fetch("target/mandelbrot.wasm"), spectest).then(
    (obj) => {
      obj.instance.exports._start();
      const calcColor = obj.instance.exports["mandelbrot/lib::calc_color"];
      const drawColor = obj.instance.exports["mandelbrot/lib::draw_color"];
      
      //...
```

Draw the image:

```rust
function saveImage() {
    context.putImageData(imagedata, 0, 0);
  }
  
  function generateImage() {
    for (row = 0; row < IMAGEHEIGHT; row++) {
      for (col = 0; col < IMAGEWIDTH; col++) {
        let x = +ox.value;
        let y = +oy.value;
        let w = +width.value;
        var color = calcColor(col, row, x, y, w);
        drawColor(imagedata, col, row, color);
      }
    }
  
    saveImage();
  }    
```

This is how the specific implementation looks like:

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/mandelbrot/640.png)

Drawing the Mandelbrot set involves a lot of mathematical derivation, which is not extensively explained in this tutorial. You can refer to:
[https://eigolomoh.bitbucket.io/math/draw_mandelbrot.1.html](https://eigolomoh.bitbucket.io/math/draw_mandelbrot.1.html)

Full code: [https://github.com/moonbitlang/moonbit-docs/tree/main/examples/mandelbrot](https://github.com/moonbitlang/moonbit-docs/tree/main/examples/mandelbrot)