layout = "post"
title = "Modern C++ Explained with Rust"
created = "2026-09-11"
updated = "2026-11-09"
tags = "#c++ #rust #computer-science"
markdown = """
I have been doing a lot of C++ recently. I think nowadays it is especially good area to discover. I always wanted to get into robotics and electronics and these require low level thinking etc.

I've programmed with C++ extensively more than a decade ago. And then I became a Unity developer. To be frank I think I had a good run with Unity but now with the advancement of AI, I think commercial game engines might become a thing of a past. It is going to take a long time but I think it will quite possible for anyone to roll out their own engine, maybe not even engine but games with their own specific tech that is only for that game.

Since I have good Rust knowledge, when I started to study modern C++, I first realized that most of these modern concepts were borrowed from Rust and I understood them better in Rust terms. Maybe I should change the title to Modern C++ for Rust Programmers.

Let's go:
### std::unique_ptr
This pointer type is what I love most about Rust, strict single ownership. In Rust terms, this is the same as Box. There can only be one pointer to the same resource. Explain more about ownership rules

### std::shared_ptr
Shared pointer is similar to Rust's Arc. It is reference counted pointer meaning, there can multiple pointers to the same resource and each pointer increments the counter. In Rust, there is a strict rule that even with Arc, there can only be 1 mutable reference.

### std::weak_ptr
"""