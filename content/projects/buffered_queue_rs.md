+++
title = "buffered-queue-rs"
description = "A custom blocking, buffered queue in Rust, built to understand backpressure, mutexes, condition variables, and thread synchronisation in a small but complete system."
weight = 4

[extra]
eyebrow = "Concurrency / Rust"
stack = ["Rust", "Concurrency"]
link_to = "https://github.com/dhruv-ahuja/buffered-queue-rs"
primary_label = "View repository"
primary_url = "https://github.com/dhruv-ahuja/buffered-queue-rs"
secondary_label = "Read the implementation notes"
secondary_url = "https://dhruvahuja.me/posts/implementing-buffered-queue-in-rust/"
+++

Inspired by the concurrency chapter of *Programming Rust*, this project helped me understand how producers and consumers coordinate around a bounded queue, and gave me a reason to work through the details instead of treating concurrency as an abstract concept.
