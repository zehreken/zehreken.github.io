layout = "post"
title = "Modern C++ Explained with Rust"
created = "2026-11-09"
updated = "2026-11-16"
tags = "#c++ #rust #computer-science"
markdown = """
I have been doing a lot of C++ recently. I think nowadays it is especially a good area to discover. I always wanted to get into robotics and electronics and these require low level thinking etc.

I've programmed with C++ extensively more than a decade ago. And then I became a Unity developer. To be frank I think I had a good run with Unity but now with the advancement of AI, I think commercial game engines might become a thing of the past. It is going to take a long time but I think it will quite possible for anyone to roll out their own engine, maybe not even engine but games with their own specific tech that is only for that game.

Since I have good Rust knowledge, when I started to study modern C++, I thought that most of these modern concepts were borrowed from Rust and I understood them better in Rust terms but then realized that these were actually introduced in C++11 and Rust actually borrowed some of the ideas. Maybe I should change the title to Modern C++ for Rust Programmers.

Let's go:
### std::unique_ptr
This pointer type is what I love most about Rust, strict single ownership. In Rust terms, this is the same as Box. There can only be one pointer to the same resource. Explain more about ownership rules

<pre class="prettyprint linenums">
#include <iostream>
#include <unordered_map>
#include <vector>

struct Texture {
  public:
    explicit Texture(std::string p, int w, int h)
        : path_(std::move(p)), width_{w}, height_{h},
          pixels_{std::make_unique<uint8_t[]>(w * h * 4)} {
        std::cout << "Loading " << path_ << '\n';
    }
    ~Texture() { std::cout << "Free " << path_ << '\n'; }
    std::string_view path() { return path_; }

  private:
    std::string path_;
    int width_, height_;
    std::unique_ptr<uint8_t[]> pixels_;
};

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

struct Enemy {
  public:
    explicit Enemy(std::shared_ptr<Texture> t) : texture_{std::move(t)} {
        std::cout << "Creating Enemy: " << texture_->path() << '\n';
    }
    ~Enemy() { std::cout << "Destroying Enemy: " << texture_->path() << '\n'; }
    std::string name_;
    void render() const { std::cout << "Rendering " << name_ << '\n'; }

  private:
    std::shared_ptr<Texture> texture_;
};

int main() {
    TextureCache texture_cache;
    std::string textures[] = {"orc.png", "goblin.png", "troll.png"};
    std::vector<Enemy> enemies;
    enemies.reserve(11);

    for (size_t i = 0; i < 11; ++i) {
        const auto& path = textures[i % 3];
        enemies.emplace_back(texture_cache.load(path, 64, 128));
    }
}
</pre>

### std::shared_ptr
Shared pointer is similar to Rust's Arc. It is reference counted pointer meaning, there can be multiple pointers to the same resource and each pointer increments the counter. In Rust, there is a strict rule that even with Arc, it is not mutable.

### std::weak_ptr
As the name suggest this is almost the same thing as Weak in Rust.
"""