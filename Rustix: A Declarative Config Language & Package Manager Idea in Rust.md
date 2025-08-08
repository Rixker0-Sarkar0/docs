# 🤖 Rustix: A Declarative Config Language & Package Manager Idea in Rust

> _"Nix-like reproducibility, C-level performance, Java-style safety — but embedded in Rust."_

## 📅 Project Idea

**Rustix** is an early-stage concept for a Rust-native system configuration and package management tool inspired by Nix, Cargo, and Bazel.

This project is **not implemented yet**. It is a **proposal and sketch** for what such a system could look like.

---

## 📊 Motivation

- Nix offers declarative, reproducible config, but:
  - Uses a custom, dynamic language
  - Difficult to extend
  - Error-prone at runtime

- Rust offers:
  - Compile-time safety
  - Strong ecosystem (crates, cargo)
  - Performance and low-level control
  - Macros and traits for DSLs

> Combine these strengths to explore a typed, embedded config system + reproducible package management in Rust.

---

## 📚 Concept Summary

**Rustix** would aim to:

- Embed a **declarative configuration DSL** into Rust via macros
- Define **typed system configs and packages**
- Use **content-addressed builds** like Nix
- Build **DAGs of dependencies** and sandboxed derivations

```rust
config! {
    system: {
        hostname: "rustos",
        users: [ user! { name: "alice" } ],
        packages: [ package!("git") ]
    }
}
```

---

## 🧠 Algorithmic Components

### 1. **AST Construction (Macro Expansion)**
- The `config!` macro expands into a Rust struct or enum that represents the configuration AST.
- Uses `proc_macro` to generate types that are valid Rust and enforce compile-time constraints.
- Internal representation is converted into a `ConfigNode` enum tree:

```rust
enum ConfigNode {
    Object(HashMap<String, ConfigNode>),
    Array(Vec<ConfigNode>),
    Value(ConfigValue),
}
```

### 2. **Dependency Graph Generation**
- A directed acyclic graph (DAG) is constructed using `petgraph` or a custom adjacency list:
  - Nodes = packages or system components
  - Edges = build or runtime dependencies

```rust
struct BuildGraph {
    nodes: HashMap<PackageId, Node>,
    edges: Vec<(PackageId, PackageId)>,
}
```

- Performs **topological sorting** for safe build ordering.

### 3. **Derivation Hashing**
- Each derivation is a struct containing:
  - Source code
  - Environment variables
  - Build inputs (dependencies)
- Serialized and hashed using SHA256:

```rust
fn hash_derivation(deriv: &Derivation) -> String {
    let serialized = serde_json::to_string(deriv).unwrap();
    sha2::Sha256::digest(serialized.as_bytes()).to_hex()
}
```

- The hash is used to locate/store outputs in the content-addressed `/store/`.

### 4. **Sandbox Execution**
- Builds can be isolated with a `SandboxRunner` trait:

```rust
trait SandboxRunner {
    fn run(&self, command: &str, inputs: &[PathBuf]) -> Result<Output>;
}
```

- Possible implementations:
  - Native process isolation (chroot, namespaces)
  - WASI execution for portability
  - Docker/container backend

### 5. **Store Manager**
- Ensures output artifacts are cached and retrieved by hash.
- Verifies integrity and avoids duplicate rebuilds.

```rust
struct Store {
    path: PathBuf,
    index: HashMap<Hash, PathBuf>,
}
```

---

## 🛠 Sketch of Components

- **config-dsl/**: Macros for embedded DSL  
- **evaluator/**: Typed AST → build graph  
- **store/**: Content-addressed derivation cache  
- **sandbox/**: Optional isolated build executor  
- **cli/**: CLI interface for build/run

---

## 🤔 Inspirations

- **Nix**: Reproducibility, derivations  
- **Cargo**: Rust-native build metadata  
- **Bazel**: DAG build systems  
- **Typst**: Declarative DSL in Rust  
- **Redox**: Potential OS target

---

## 🔎 Current Status

> This is a proposal only. No implementation exists yet.

If you're interested in exploring this further, reach out or fork this into a real prototype.

---

## 🚀 Future Possibility

> A Rust-native ecosystem for system builds, configuration, and packaging — fully typed, declarative, and reproducible.

---

## 📁 License

Open idea. Use it freely. Attribution appreciated.
