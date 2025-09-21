+++
title = "Writing Rust Bindings for My Python App"
description = "Walk through my experience of writing Rust FFI for my Python CLI application"
date = "2023-11-06"

[taxonomies]
tags = ["rust", "python", "ffi", "performance", "spoti-dl"]
+++

## Introduction

[spoti-dl](https://github.com/dhruv-ahuja/spoti-dl "https://github.com/dhruv-ahuja/spoti-dl"), a Python-based CLI song downloading tool was the first “proper” application that I developed. It acted as a proof-of-concept of my programming skills as a self-taught developer, and helped me land my first job. However, it lacked some basic features, mainly- no parallel downloads for albums and playlists.

I recently added a few new features and re-wrote its core functionality in Rust, as I have been enjoying working with Rust’s robust type system, compiler-level error handling and syntax.

## Development

Development was relatively smooth for the most part, as the app logic is straightforward — you accept and parse the input Spotify link, the CLI flag parameters and process downloads. I figured out general things by googling and/or through some experimentation, such as the trait implementations to parse CLI flags from `String`s into `enum`s and vice-versa. The `lazy_static` macro helped me allocate a static `HashSet` containing disallowed characters for files and folder names, on runtime. I also became more comfortable with bound traits and experienced the power of generics. I was able to use the following function across all of my download flows, as it accepts any input `P` that can be referenced as `Path` and any input `S` that can be converted into a `String` type:

```rust
pub fn add_metadata<P, S>(
    file_path: P,
    album_art_path: P,
    simple_song: spotify::SimpleSong,
    album_name: S,
) where
    P: AsRef<Path> + Debug,
    S: Into<String>,
{...}
```

I mainly struggled when implementing the async logic to download songs in parallel, due to my inexperience with writing async code in Rust. I had to spend a lot of time working with the compiler’s restrictions and [Tokio’s](https://tokio.rs/ "https://tokio.rs/") `’static + Send` requirements for spawning tasks, as its work-stealing scheduler model means that a task running in one thread could be picked up by another thread. I used `tokio::task::block_in_place` to wrap the `add_metadata` function call as the [lofty](https://github.com/Serial-ATA/lofty-rs "https://github.com/Serial-ATA/lofty-rs") crate does not support async.

I added a CLI flag, allowing users to specify the number of tasks to use to process parallel downloads, and used batch downloads of 100 songs for playlists, as they can contain several thousands of songs.

The following is the core async logic for parallel downloads — calculate songs to be downloaded by each task, make `Arc`s to pass cheap, shareable clones for certain values, chunk the list of songs and create and wait for the spawned tasks to finish downloads:

```rust
let parallel_tasks: usize = if album.songs.len() >= cli_args.parallel_downloads as usize {
    cli_args.parallel_downloads as usize
} else {
    album.songs.len()
};

let songs_per_task = album.songs.len() / parallel_tasks;
let remaining_songs = album.songs.len() % parallel_tasks;

let cli_args = Arc::new(cli_args);
let album_art_dir = Arc::new(album_art_dir);
let album_name = Arc::new(album.name);

let mut handles = Vec::with_capacity(parallel_tasks);
let mut start = 0;

for i in 0..parallel_tasks {
    let mut end = start + songs_per_task;
    if i < remaining_songs {
        end += 1
    }

    let songs_chunk = &album.songs[start..end];
    let handle = tokio::spawn(download_songs(
        file_path.clone(),
        cli_args.clone(),
        album_art_dir.clone(),
        album_name.clone(),
        songs_chunk.to_vec(),
    ));

    start = end;
    handles.push(handle)
}

for handle in handles {
    handle.await?;
}
```

## Tooling

I dropped [Poetry](https://python-poetry.org/ "https://python-poetry.org/") as it would not be compatible with the Rust bindings and used simple virtual environments for dependency management, and [Twine](https://twine.readthedocs.io/en/stable/ "https://twine.readthedocs.io/en/stable/") for distributing built wheels.

[Pyo3](https://pyo3.rs/v0.20.0/ "https://pyo3.rs/v0.20.0/") acts as the bridge between the parent Python code that calls a single exposed Rust function and enables all the inter-op between the two systems. [Maturin](https://github.com/PyO3/maturin "https://github.com/PyO3/maturin") compiles the Rust code into a Python library, and also compiles both codebases into a distributable Python wheel.

The following is a list of changes I had to make in my `Cargo` and `pyproject` TOML files, to ensure that the build process and `pip` installed package worked as intended:

- `Maturin` did not recognize the project as a mixed Python-Rust project, hence did not include Rust code in the distributable Python wheel. Setting `lib.name` table’s value to match Python source directory (`spotidl`) in `Cargo.toml` fixed this error.

- `pyproject.toml` required several modifications — I needed to set the `project.scripts` value to `spoti-dl = "spotidl.main:main"`, partially because the project name (`spoti-dl`) and Python source directory  names were different. I also added the `python-packages = ["spotidl"]` value under `tool.maturin` to ensure its inclusion during the build process. I also had to add my dependencies and relevant project metadata in their apt sections, after dropping `Poetry`.

- `Maturin` compiles the Rust code as a library inside our Python source directory. It adds an underscore `_` to the library’s name by default, which is quite confusing. I rectified this by configuring the `module-name` value under `tool.maturin`.

I faced several problems when attempting to build wheels for Linux using Docker, on my M1 MacBook. I must have easily spent 15-20 hours trying to get the `openssl-sys` crate to compile as it was the single point of failure, using both the python `manylinux` and `maturin` Docker images. I tried to integrate a CI/CD setup using GitHub Actions too, but to no avail, as the crate kept failing to compile. You can check the graveyard of my CI’s failed runs [here](https://github.com/dhruv-ahuja/spoti-dl/actions "https://github.com/dhruv-ahuja/spoti-dl/actions"). Eventually I had to settle for manually compiling wheels on Linux, Mac and Windows and copying them to a folder before publishing them with `Twine`.

## Conclusion

This was a rewarding experience for me, as I dealt with efficiently processing large amounts of data and sharpened my skills with Rust and `Tokio`.

I witnessed a 20-25% speed increase and 50% less memory consumption in my Rust code when downloading a single song. The development process was smooth as `Pyo3` and `Maturin` are very well-documented and provide convenient APIs, make it incredibly easy to get started with writing FFIs for Python.
