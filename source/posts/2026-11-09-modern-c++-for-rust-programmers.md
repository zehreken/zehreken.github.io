layout = "post"
title = "[DRAFT] Modern C++ for Rust Programmers"
created = "2026-11-09"
updated = "2026-11-16"
tags = "#c++ #rust #computer-science"
markdown = """
I have been doing a lot of C++ recently. I think nowadays it is especially a good area to explore. I always wanted to get into robotics and electronics and these require low level thinking etc.

I've programmed extensively in C++ more than a decade ago. And then I became a Unity developer. To be frank I think I had a good run with Unity but now with the advancement of AI, I think commercial game engines might become a thing of the past. It is going to take a long time but I think it will be quite possible for anyone to roll out their own engine, maybe not even an engine but games will have their own specific tech that is only for that game. This was not feasible before LLMs.

Since I know Rust, when I started to study modern C++, I thought that most of these modern concepts were borrowed from Rust and I understand them better in Rust terms but then realized that these were actually introduced in C++11 and Rust actually borrowed some of the ideas.

I came up with this simple example which uses all three smart pointers in C++ and also resembles real code, I think.

Let's go:
### std::unique_ptr
This pointer type is what I love most about Rust, strict single ownership. In Rust terms, this is the same as Box. There can only be one pointer to the same resource. Let's look at this Texture struct

```cpp
struct Texture {
  public:
    explicit Texture(std::string p, int w, int h)
        : path_(std::move(p)), width_{w}, height_{h},
          pixels_{std::make_unique<uint8_t[]>(w * h * 4)} {
        std::cout << "Loading " << path_ << '\n';
    }
    std::string_view path() { return path_; }

  private:
    std::string path_;
    int width_, height_;
    std::unique_ptr<uint8_t[]> pixels_;
};
```

And this is our current main function, note that I included two cases where the program does not compile. Because of the unique_ptr, the compiler deletes copy ctor and copy assignment and this is good because unique_ptr should be the only one pointing. There is an important issue with move operations. As you can see, width and height is copied but path is empty and pixels is null. 

```cpp
int main() {
    Texture foo{"foo.png", 2, 2};
    Texture bar("bar.png", 2, 2);
    // Texture foo{bar}; // Does not compile because copy ctor is deleted
    // foo = bar; // Does not compile because copy assignment is deleted
    Texture move_ctor{std::move(foo)};
    foo = std::move(bar);
    std::cout << bar.path() << " " << bar.size() << " " << bar.has_pixels() << '\n';
}
```

I think this part is better handled in Rust. You can't use the object that is moved from after the operation. You can see it in the very simple example below.
```
fn main() {
    let box_a = Box::new(5);
    
    let box_b = box_a;
    
    // Compile error: Rust compiler complains that box_a is moved, so we can't use it anymore
    // println!("box_a: {:p}->{}", box_a, box_a);
}
```

### std::shared_ptr
Shared pointer is similar to Rust's Arc. It is reference counted pointer meaning, there can be multiple pointers to the same resource and each pointer increments the counter. In Rust, there is a strict rule that even with Arc, it is not mutable.

```cpp
struct TextureCache {
  public:
    std::shared_ptr<Texture> load(const std::string& path, int w, int h) {
        if (auto it = cache_.find(path); it != cache_.end()) {
            if (auto texture = it->second.lock()) {
                return texture;
            }
        }

        auto texture = std::make_shared<Texture>(path, w, h);
        cache_[path] = texture;
        return texture;
    }

  private:
    std::unordered_map<std::string, std::weak_ptr<Texture>> cache_;
};
```

### std::weak_ptr
As the name suggest this is almost the same thing as Weak in Rust.
"""