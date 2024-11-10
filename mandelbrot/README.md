# Mandelbrot Set Renderer

This Rust program generates an image of the Mandelbrot set.

## Usage

To build and run the program, follow these steps:

### Build the Program

First, build the program in release mode to optimize performance:

```sh
cargo build --release
```

### Run the Program

After building the program, you can run it with the following command:

```sh
target/release/mandelbrot <OUTPUT_FILE> <PIXELS> <UPPERLEFT> <LOWERRIGHT>
```

- `<OUTPUT_FILE>`: The name of the output PNG file.
- `<PIXELS>`: The dimensions of the output image in the format `WIDTHxHEIGHT`.
- `<UPPERLEFT>`: The coordinates of the upper-left corner of the image in the complex plane, in the format `REAL,IMAGINARY`.
- `<LOWERRIGHT>`: The coordinates of the lower-right corner of the image in the complex plane, in the format `REAL,IMAGINARY`.

### Example

Here is an example command to generate a 4000x3000 image of the Mandelbrot set:

```sh
target/release/mandelbrot mandel.png 4000x3000 -1.20,0.33 -1.0,0.20
```

## Code Explanation

### `escape_time` Function

This function determines if a complex number `c` is in the Mandelbrot set, using at most `limit` iterations to decide.

```rust
fn escape_time(c: Complex<f64>, limit: usize) -> Option<usize> {
    let mut z = Complex { re: 0.0, im: 0.0 };
    for i in 0..limit {
        if z.norm_sqr() > 4.0 {
            return Some(i);
        }
        z = z * z + c;
    }
    None
}
```

### `parse_pair` Function

This function parses a string `s` and attempts to split it into two values of type `T` separated by `separator`.

```rust
fn parse_pair<T: FromStr>(s: &str, separator: char) -> Option<(T, T)> {
    match s.find(separator) {
        None => None,
        Some(index) => {
            match (T::from_str(&s[..index]), T::from_str(&s[index + 1..])) {
                (Ok(l), Ok(r)) => Some((l, r)),
                _ => None
            }
        }
    }
}
```

### `parse_complex` Function

This function parses a string `s` representing a complex number in the format `REAL,IMAGINARY`.

```rust
fn parse_complex(s: &str) -> Option<Complex<f64>> {
    match parse_pair(s, ',') {
        Some((re, im)) => Some(Complex { re, im }),
        None => None
    }
}
```

### `pixel_to_point` Function

This function converts pixel coordinates to a point in the complex plane.

```rust
fn pixel_to_point(bounds: (usize, usize), pixel: (usize, usize), upper_left: Complex<f64>, lower_right: Complex<f64>) -> Complex<f64> {
    let (width, height) = (lower_right.re - upper_left.re, upper_left.im - lower_right.im);

    Complex {
        re: upper_left.re + pixel.0 as f64 * width / bounds.0 as f64,
        im: upper_left.im - pixel.1 as f64 * height / bounds.1 as f64
    }
}
```

### `render` Function

This function renders the Mandelbrot set into a pixel buffer.

```rust
fn render(pixels: &mut [u8], bounds: (usize, usize), upper_left: Complex<f64>, lower_right: Complex<f64>) {
    assert!(pixels.len() == bounds.0 * bounds.1);
    for row in 0..bounds.1 {
        for column in 0..bounds.0 {
            let point = pixel_to_point(bounds, (column, row), upper_left, lower_right);
            pixels[row * bounds.0 + column] = match escape_time(point, 255) {
                None => 0,
                Some(count) => 255 - count as u8
            }
        }
    }
}
```

### `write_image` Function

This function writes the pixel buffer to a PNG file.

```rust
fn write_image(filename: &str, pixels: &[u8], bounds: (usize, usize)) -> Result<(), std::io::Error> {
    let output = File::create(filename)?;
    let encoder = PNGEncoder::new(output);
    encoder.encode(pixels, bounds.0 as u32, bounds.1 as u32, ColorType::Gray(8))?;
    Ok(())
}
```

### `main` Function

This is the entry point of the program. It parses command-line arguments, renders the Mandelbrot set, and writes the output image.

```rust
fn main() {
    let args: Vec<String> = env::args().collect();
    if args.len() != 5 {
        eprintln!("Usage: mandelbrot FILE PIXELS UPPERLEFT LOWERRIGHT");
        eprintln!("Example: {} mandel.png 1000x750 -1.20,0.35 -1,0.20", args[0]);
        std::process::exit(1);
    }
    let bounds = parse_pair(&args[2], 'x').expect("error parsing image dimensions");
    let upper_left = parse_complex(&args[3]).expect("error parsing upper left corner point");
    let lower_right = parse_complex(&args[4]).expect("error parsing lower right corner point");
    let mut pixels = vec![0; bounds.0 * bounds.1];
    render(&mut pixels, bounds, upper_left, lower_right);
    write_image(&args[1], &pixels, bounds).expect("error writing PNG file");
}
```

## Testing

The code includes several test functions to verify the correctness of the parsing and conversion functions.

### `test_parser_pair` Function

```rust
#[test]
fn test_parser_pair() {
    assert_eq!(parse_pair::<i32>("", ','), None);
    assert_eq!(parse_pair::<i32>("10,", ','), None);
    assert_eq!(parse_pair::<i32>(",10", ','), None);
    assert_eq!(parse_pair::<i32>("10,20", ','), Some((10, 20)));
    assert_eq!(parse_pair::<i32>("10,20xy", ','), None);
}
```

### `test_parse_complex` Function

```rust
#[test]
fn test_parse_complex() {
    assert_eq!(parse_complex("1.25,-0.0625"), Some(Complex { re: 1.25, im: -0.0625 }));
    assert_eq!(parse_complex(",-0.0625"), None);
}
```

### `test_pixel_to_point` Function

```rust
#[test]
fn test_pixel_to_point() {
    assert_eq!(pixel_to_point((100, 200), (25, 175), Complex { re: -1.0, im: 1.0 }, Complex { re: 1.0, im: -1.0 }), Complex { re: -0.5, im: -0.75 })
}
```
