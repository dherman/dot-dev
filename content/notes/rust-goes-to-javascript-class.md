+++
date = '2025-12-03T11:22:19-08:00'
title = 'Rust Goes to JavaScript Class'
featured_image = '/images/hero/classroom.jpg'
+++

Today we merged syntax for defining classes in [Neon](https://neon-rs.dev), a library I maintain with [K.J. Valencik](https://github.com/kjvalencik) for embedding Rust code in Node.

Bridging two languages with different object models is a fun challenge. Rust has structs and impls; JavaScript has prototypes and constructors. The goal was to make it feel natural and idiomatic on both sides, and I'm pretty happy with how it came out.

<!--more-->

Here's a taste:

```rust
struct Point {
    x: u32,
    y: u32,
}

#[neon::class]
impl Point {
    fn new(x: u32, y: u32) -> Self {
        Self { x, y }
    }

    fn x(&self) -> u32 {
        self.x
    }

    fn y(&self) -> u32 {
        self.y
    }

    fn distance(&self, other: &Self) -> f64 {
        let dx = (self.x as i32 - other.x as i32).pow(2);
        let dy = (self.y as i32 - other.y as i32).pow(2);
        ((dx + dy) as f64).sqrt()
    }
}
```

And on the JavaScript side:

```javascript
const { Point } = addon;

const source = new Point(0, 0);
const dest = new Point(3, 4);

console.log(source.distance(dest)); // 5
```

That's it and it feels pretty clean to me! With one little annotation, you get implementations in two languages with well-defined interop.

## Under the Hood

The implementation was fun. We needed the macro to take one syntax and generate code for both worlds:
```goat {.f7}
+-------------------------+
| #[neon::class]          |
| impl Point { … }        |
+-------------------------+
      |           |
      v           v
+------------+  +-------------+
| impl Point |  | class Point |
| { … }      |  | { … }       |
+------------+  +-------------+
     Rust          JavaScript
```
This builds on K.J.'s earlier work on the [`#[neon::export]`](https://docs.rs/neon/latest/neon/attr.export.html) macro and [conversion traits](https://docs.rs/neon/latest/neon/types/extract/index.html#traits), which handle the heavy lifting of translating types across the language boundary.

The key trick of the `#[neon::class]` macro is generating custom JavaScript source code that runs at class initialization time. For our `Point` example, it looks something like this:

```javascript
(function makeClass(wrap, xMethod, yMethod, distanceMethod) {
  class Point {
    constructor(x, y) {
      wrap(this, x, y); // wrap `this` around Rust struct
    }
  }

  const prototype = Point.prototype;

  prototype.x = xMethod;
  prototype.y = yMethod;
  prototype.distance = distanceMethod;

  return Point;
})
```

The initialization logic calls [`neon::reflect::eval`](https://docs.rs/neon/latest/neon/reflect/fn.eval.html) with that generated source string to create the `makeClass` function, then calls it with the Rust implementations as arguments:

```rust
const CLASS_MAKER_SCRIPT: &str = r#"
(function makeClass(wrap, xMethod, yMethod, distanceMethod) {
  // etc ...
})
"#;

let src = cx.string(CLASS_MAKER_SCRIPT);
let make_class = neon::reflect::eval(&mut cx, src)?
    .downcast::<JsFunction, _>(&mut cx)
    .or_throw(&mut cx)?;

// constructs the Rust struct and wraps `this` around it
let wrap = JsFunction::new(&mut cx, |mut cx| { /* ... */ })?;

let constructor: Handle<JsFunction> = make_class
    .bind(&mut cx)
    .args((wrap, x_method, y_method, distance_method))?
    .call()?;
```

The `wrap` function is the final piece of glue. It takes the pure JavaScript object from the `Point` constructor and attaches the Rust struct to it (using Node's [`napi_wrap`](https://nodejs.org/api/n-api.html#napi_wrap) API):

```rust
let wrap = JsFunction::new(&mut cx, |mut cx| {
    let this = cx.this()?;
    let (x, y): (u32, u32) = cx.args()?;
    let instance = Point::new(x, y);
    // uses napi_wrap to attach the instance data to the JS object
    neon::object::wrap(&mut cx, &this, RefCell::new(instance))?;
    Ok(cx.undefined())
})?;
```

That's pretty much it: Rust structs, dressed up as JavaScript classes, ready for `new`. The [real implementation](https://github.com/neon-bindings/neon/pull/1117) contains more details and edge cases but this is the basic idea.

If all goes well, classes will ship in the next release of Neon.

###### image credits: [Tuyen Vo](https://unsplash.com/photos/a-dining-room-table-7o7DqXJArf4)
