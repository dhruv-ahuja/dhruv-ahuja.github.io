+++
title = "spoti-dl"
description = "A CLI tool for downloading songs, albums, and playlists from Spotify links, with metadata and album cover art. I rewrote its core functionality in Rust and introduced parallel downloads, achieving more than 4x speedup for large albums and playlists."
weight = 1

[extra]
featured = true
status = "Not actively maintained"
eyebrow = "Built in public / Python and Rust"
stack = ["Python", "Rust", "PyO3"]
highlights = ["45,000+ PyPI downloads", "70+ GitHub stars", "More than 4x faster large-album and playlist downloads"]
link_to = "https://github.com/dhruv-ahuja/spoti-dl"
primary_label = "View repository"
primary_url = "https://github.com/dhruv-ahuja/spoti-dl"
secondary_label = "Read the Rust rewrite"
secondary_url = "https://dhruvahuja.me/posts/writing-rust-bindings/"
+++

The project started as my first serious application and became the project through which I learned to build in public. I began with a Python implementation, listened to feedback from the communities where I shared it, and later moved the performance-sensitive core to Rust while keeping the Python interface and packaging workflow intact.
