# MiniGrep

MiniGrep is a simple command-line tool written in Rust that searches for a given query in a specified file. This project is inspired by the Unix `grep` command and is built to demonstrate Rust's error handling, file reading, and environment variable usage.

## Features
- Searches for a specified word or phrase in a file
- Supports case-sensitive and case-insensitive search (via an environment variable)
- Displays matching lines from the file
- Includes error handling and test cases

## Installation
To use MiniGrep, ensure that you have Rust installed. You can install Rust using [rustup](https://www.rust-lang.org/tools/install):

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Then, clone this repository and navigate into the project directory:

```sh
git clone https://github.com/ravipatelctf/minigrep.git
cd minigrep
```

## Usage
To run MiniGrep, use the following command:

```sh
cargo run -- <query> <input_file_path> > <output_file_path>
```

For example:

```sh
cargo run -- to poem.txt > output.txt
```

To enable case-insensitive search, set the `IGNORE_CASE` environment variable:

```sh
IGNORE_CASE=1 cargo run -- to poem.txt > output.txt
```

## Code Structure

### `main.rs`
The `main.rs` file handles command-line argument parsing and error handling. It:
- Collects command-line arguments
- Passes them to the `Config` struct for parsing
- Calls the `run` function from the `lib.rs` file to perform the search

```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();
    
    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```

### `lib.rs`
The `lib.rs` file contains the core functionality of MiniGrep:

#### `Config` Struct
- Stores the search query, file path, and an optional flag for case-insensitive search.
- The `build` method parses command-line arguments and checks for the `IGNORE_CASE` environment variable.

```rust
pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config { query, file_path, ignore_case })
    }
}
```

#### `run` Function
- Reads the file content and searches for the query based on the case sensitivity setting.
- Prints matching lines to the console.

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}
```

#### Search Functions
- `search`: Performs case-sensitive search.
- `search_case_insensitive`: Performs case-insensitive search.

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

### Testing
The project includes unit tests in `lib.rs` to verify the search functions.

To run the tests, use:

```sh
cargo test
```

Example test case:

```rust
#[test]
fn case_sensitive() {
    let query = "duct";
    let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

    assert_eq!(vec!["safe, fast, productive."], search(query, contents));
}
```

## License
This project is licensed under the MIT License.

---

This `README.md` provides a structured overview of the MiniGrep project, making it easier for users to understand, install, and contribute to the repository.

