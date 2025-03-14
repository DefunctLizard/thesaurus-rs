# thesaurus-rs
[![Crates.io](https://img.shields.io/crates/v/thesaurus.svg)](https://crates.io/crates/thesaurus)
[![Crates.io](https://img.shields.io/crates/d/thesaurus)](https://crates.io/crates/thesaurus)
[![API](https://docs.rs/thesaurus/badge.svg)](https://docs.rs/thesaurus)

The offline thesaurus library for Rust that can use both [wordnet](https://wordnet.princeton.edu/) and [moby](https://en.wikipedia.org/wiki/Moby_Project) thesauruses.

Add to `Cargo.toml` for wordnet:
```toml
thesaurus = "0.5"
```

Add to `Cargo.toml` for just moby *(wordnet and static are on by default)*:
```toml
thesaurus = { version = "0.5", features = ["moby","static"], default_features = false }
```

## Crate Feature "`static`"
`static` is a feature that is turned on by default that the thesaurus dictionary to be stored in memory during runtime. This allows calls to [`dict`] and [`synonyms`] to be **2.5-3x faster after initialization** with the added downside of increased memory usage. To turn off `static`, use the `default_features = false` option.

## Backend Comparison
| Name    | Simple Example Binary Size | Simple Example Binary Size (Stripped) | Available Words | Average Number of Synonyms | Compressed Dictionary Size | License                                                                     |
|---------|----------------------------|---------------------------------------|-----------------|----------------------------|----------------------------|-----------------------------------------------------------------------------|
| Moby    | 15M                        | 11M                                   | 30159           | 83.287                     | 11M                        | US Public Domain                                                            |
| Wordnet | 6.9M                       | 3.4M                                  | 125701          | 3.394                      | 2.9M                       | [Wordnet License](https://wordnet.princeton.edu/license-and-commercial-use) |

## Basic Example
```rust
use std::{env, process};

fn main() {
    let args = env::args().collect::<Vec<String>>();

    let word: String = match args.get(1) {
        Some(word) => word.to_string(),
        None => {
            eprintln!("Must include a word as an argument");
            process::exit(1);
        }
    };

    let synonyms = thesaurus::synonyms(&word);
    let num_words = thesaurus::dict().len();

    cfg_if::cfg_if! {
        if #[cfg(all(feature = "moby", feature = "wordnet"))] {
            print!("both wordnet and moby have ");
        } else if #[cfg(feature = "moby")] {
            print!("moby has ");
        } else if #[cfg(feature = "wordnet")] {
            print!("wordnet has ");
        }
    }

    println!("{num_words} words indexed, and {} synonyms for \"{word}\"...", synonyms.len());
    println!("synonyms...");
    for x in &synonyms {
        println!("   {x}");
    }
}
```

### Wordnet Output
```shell
 $ cargo r -rq --example simple -- good
```

```
wordnet has 125701 words indexed, and 107 synonyms for "good"...
synonyms...
   acceptable
   adept
   advantageous
   ample
   angelic
   angelical
   bang-up
   beatific
   beneficial
   beneficial
   best
...
```

### Moby Output
```shell
 $ cargo r -rq --example simple --no-default-features --features=moby -- good
```

```
moby has 30195 words indexed, and 666 synonyms for "good"...
synonyms...
   able to pay
   absolutely
   acceptable
   accomplished
   according to hoyle
   ace
   actual
   adept
   adequate
   admirable
   admissible
   adroit
   advantage
   advantageous
...
```

## Both Output
```shell
 $ cargo r -rq --example simple --features=moby,wordnet -- good
```

```
both wordnet and moby have 132592 words indexed, and 773 synonyms for "good"...
synonyms...
   able to pay
   absolutely
   acceptable
   acceptable
   accomplished
   according to hoyle
   ace
   actual
   adept
   adept
   adequate
   admirable
   admissible
   adroit
...
```