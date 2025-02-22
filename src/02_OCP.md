# Open/Close Principle (OCP)

The **Open/Closed Principle (OCP)** states that:

> *Software entities (such as classes, modules, and functions) should be **open for extension but closed for modification**.*

This means we should design our code in a way that allows new functionality to be added **without altering existing code**.  

In Rust, we can achieve this by leveraging **traits and generics**, instead of modifying existing structures directly.  

## Example: A Basic Shape Renderer

Imagine we have a simple shape rendering system. A naive approach might look like this:  

```rust
struct Circle;
struct Square;

struct Renderer;

impl Renderer {
    fn render(&self, shape: &str) {
        match shape {
            "circle" => println!("Rendering a circle"),
            "square" => println!("Rendering a square"),
            _ => println!("Unknown shape"),
        }
    }
}
```

This implementation **violates OCP** because every time we introduce a new shape (e.g., a Triangle), we must **modify** the `Renderer`'s `render` method.  

## Applying the Open/Closed Principle  

A better approach is to use **traits** to define a contract for rendering:  

```rust
trait Shape {
    fn render(&self);
}

struct Circle;
struct Square;

impl Shape for Circle {
    fn render(&self) {
        println!("Rendering a circle");
    }
}

impl Shape for Square {
    fn render(&self) {
        println!("Rendering a square");
    }
}

struct Renderer;

impl Renderer {
    fn render<T: Shape>(&self, shape: &T) {
        shape.render();
    }
}
```

## Benefits of This Approach  

1. **Open for Extension** – We can add new shapes (e.g., `Triangle`) without modifying existing code.  
2. **Closed for Modification** – The `Renderer` struct remains unchanged regardless of new shapes.  

## Adding a New Shape Without Modification  

```rust
struct Triangle;

impl Shape for Triangle {
    fn render(&self) {
        println!("Rendering a triangle");
    }
}

fn main() {
    let renderer = Renderer;
    let circle = Circle;
    let square = Square;
    let triangle = Triangle;

    renderer.render(&circle);
    renderer.render(&square);
    renderer.render(&triangle); // Works without modifying Renderer!
}
```

This implementation follows the **Open/Closed Principle** by allowing new shapes to be added **without modifying existing code**, keeping our system flexible and maintainable. 🚀

Let's explore a more advanced approach to the **Open/Closed Principle (OCP)** in Rust, using **dynamic dispatch** with trait objects and enums.  

---

## **Advanced OCP in Rust: Dynamic Dispatch and Enums**

In the previous example, we used **generics** and **static dispatch** (`T: Shape`), which works well but can lead to excessive monomorphization (code duplication at compile-time). Instead, we can use **dynamic dispatch** (`dyn Shape`) or **enums** to achieve extensibility.

### **1. Using Dynamic Dispatch (`dyn Shape`)**  

With **dynamic dispatch**, we store different shape types as **trait objects** (`Box<dyn Shape>`), allowing us to handle multiple shapes **without modifying existing code**.

```rust
trait Shape {
    fn render(&self);
}

struct Circle;
struct Square;

impl Shape for Circle {
    fn render(&self) {
        println!("Rendering a circle");
    }
}

impl Shape for Square {
    fn render(&self) {
        println!("Rendering a square");
    }
}

// Renderer handles any Shape dynamically
struct Renderer {
    shapes: Vec<Box<dyn Shape>>,
}

impl Renderer {
    fn new() -> Self {
        Self { shapes: Vec::new() }
    }

    fn add_shape(&mut self, shape: Box<dyn Shape>) {
        self.shapes.push(shape);
    }

    fn render_all(&self) {
        for shape in &self.shapes {
            shape.render();
        }
    }
}

fn main() {
    let mut renderer = Renderer::new();
    
    renderer.add_shape(Box::new(Circle));
    renderer.add_shape(Box::new(Square));
    
    renderer.render_all();
}
```

### **Why This Works?**

✅ **Open for Extension** – We can add new shapes **without modifying** `Renderer`.  
✅ **Closed for Modification** – The `Renderer` struct stays unchanged regardless of new shapes.  

---

### **2. Using Enums for Compile-Time Safety**  

Instead of **dynamic dispatch**, we can use **enums** to enforce shape types at compile time.  

```rust
enum Shape {
    Circle,
    Square,
}

impl Shape {
    fn render(&self) {
        match self {
            Shape::Circle => println!("Rendering a circle"),
            Shape::Square => println!("Rendering a square"),
        }
    }
}

struct Renderer {
    shapes: Vec<Shape>,
}

impl Renderer {
    fn new() -> Self {
        Self { shapes: Vec::new() }
    }

    fn add_shape(&mut self, shape: Shape) {
        self.shapes.push(shape);
    }

    fn render_all(&self) {
        for shape in &self.shapes {
            shape.render();
        }
    }
}

fn main() {
    let mut renderer = Renderer::new();
    
    renderer.add_shape(Shape::Circle);
    renderer.add_shape(Shape::Square);
    
    renderer.render_all();
}
```

### **When to Use Each Approach?**

| Approach | Pros | Cons |
|----------|------|------|
| **Dynamic Dispatch (`dyn Shape`)** | More flexible, allows runtime polymorphism | Slightly slower due to vtable lookup |
| **Enums (`enum Shape`)** | Faster (no dynamic dispatch), safer | Requires modifying enum when adding new shapes |

---

## **Conclusion**  

By using **trait objects** or **enums**, we can design our code to be **open for extension but closed for modification**, adhering to the **Open/Closed Principle** in Rust.  

- Use **`dyn Trait`** for flexibility and runtime polymorphism.  
- Use **enums** for performance and compile-time safety.  

Would you like an even more advanced example, perhaps with **functional-style composition** or **strategy patterns**? 🚀
