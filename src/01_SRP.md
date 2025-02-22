# Single Responsibility Principle (SRP)

The **Single Responsibility Principle (SRP)** is the first of the SOLID principles and states:  

> *A class (or struct, in Rust) should have only one reason to change.*  

 ```mermaid
    ---
    config:
      look: handDrawn
      theme: neutral
    ---
    flowchart LR
      U(U) -.-> F1(F1)
      U(U) -.-> F2(F2)
```

This means that a module should focus on a single functionality, making the code easier to maintain and extend.  

## A Bad Example: Report Data Strcture

Let's consider a `Report` struct that handles both data and file operations:  

```rust
use std::fs::File;
use std::io::Write;

struct Report {
    title: String,
    content: String,
}

impl Report {
    fn new(title: &str, content: &str) -> Self {
        Self { title: title.to_string(), content: content.to_string() }
    }

    fn save_to_file(&self, filename: &str) {
        let mut file = File::create(filename).expect("Failed to create file");
        writeln!(file, "{}\n{}", self.title, self.content).expect("Failed to write to file");
    }
}
```

Here, `Report` has **two responsibilities**:  

1. **Managing report data** (title and content).  
2. **Handling file operations** (saving to disk).  

If we need to change how the report is stored (e.g., switch to a database), we must modify `Report`, violating SRP.  

## A Better Example: Applying SRP

We can refactor by separating concerns:  

```rust
use std::fs::File;
use std::io::Write;

struct Report {
    title: String,
    content: String,
}

impl Report {
    fn new(title: &str, content: &str) -> Self {
        Self { title: title.to_string(), content: content.to_string() }
    }

    fn format(&self) -> String {
        format!("{}\n{}", self.title, self.content)
    }
}

struct FileStorage;

impl FileStorage {
    fn save(filename: &str, content: &str) {
        let mut file = File::create(filename).expect("Failed to create file");
        writeln!(file, "{}", content).expect("Failed to write to file");
    }
}

fn main() {
    let report = Report::new("SRP in Rust", "Keep responsibilities separate!");
    let formatted_report = report.format();
    FileStorage::save("report.txt", &formatted_report);
}
```

### Why is this better?

✅ **Report** is now only responsible for managing report data.  
✅ **FileStorage** handles saving, making it easier to switch storage methods.  

By applying SRP, we create **more modular and maintainable** Rust code. 🚀
