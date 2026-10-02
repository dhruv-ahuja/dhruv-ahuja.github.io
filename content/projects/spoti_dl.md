+++
title = "spoti-dl"
description = "A Python CLI for downloading songs, albums, and playlists from Spotify links. I moved its performance-sensitive download path to Rust and added concurrent downloads, making large albums and playlists more than 4x faster."
weight = 1

[extra]
featured = true
status = "Maintenance"
eyebrow = "Python CLI / Rust rewrite"
stack = ["Python", "Rust", "PyO3"]
highlights = ["45,000+ PyPI downloads", "70+ GitHub stars", "More than 4x faster on large albums and playlists"]
link_to = "https://github.com/dhruv-ahuja/spoti-dl"
primary_label = "View repository"
primary_url = "https://github.com/dhruv-ahuja/spoti-dl"
secondary_label = "Read the Rust rewrite"
secondary_url = "https://dhruvahuja.me/posts/writing-rust-bindings/"
+++

spoti-dl started as my first serious application. I shared the Python version publicly, used feedback to shape it, and later moved the performance-sensitive core to Rust while keeping the Python interface and packaging workflow intact.
