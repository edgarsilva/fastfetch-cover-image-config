# 🐉 Fastfetch + Kitty Image Wrapper (`ff`)

This repository contains a small **Bash wrapper** (`ff`) and a matching **Fastfetch configuration** that displays a randomly selected, scaled, and centered image next to your Fastfetch system info — using **Kitty’s graphics protocol**.

The goal is to reproduce the aesthetic of *Neofetch’s image backends* (like `kitty` or `w3m`) but with:

- better scaling and cropping (“CSS `cover`” behavior),
- smart caching for instant reloads,
- and high-quality preprocessing via `ffmpeg` or `ImageMagick`.

---

## 📁 Repository Layout

```
~/.config/fastfetch/
├── ff                 # The Bash wrapper script (executable)
└── config.jsonc       # Fastfetch configuration
```

---

## ⚙️ Features

✅ **Random image selection**  
Pulls a random `.png`, `.jpg`, `.jpeg`, or `.webp` file from `~/Pictures/neofetch/`.

✅ **Aspect-ratio–aware scaling**  
Uses FFmpeg (or ImageMagick) to crop and scale images according to a *cover* or *contain* mode, mimicking CSS behavior.

✅ **Smart caching**  
Preprocessed images are hashed and stored under `~/.cache/fastfetch/` for fast reuse.

✅ **Kitty-native image rendering**  
Draws the image using `kitty +kitten icat` with precise control of cell placement (`--place WxH@XxY`).

✅ **Fastfetch integration**  
The Fastfetch config references the generated binary stream (`image.bin`) directly as a `logo` of type `raw`.

✅ **Customizable image box size**  
Control via environment variables (`W`, `H`, `X`, `Y`) to fit your terminal’s Fastfetch layout.

---

## 🚀 Installation

1. **Copy files (or clone the repo)**

   ```bash
   mkdir -p ~/.config/fastfetch
   cd ~/.config/fastfetch
   # Save ff and config.jsonc here
   chmod +x ff
   ```

2. **Install dependencies**

   ```bash
   sudo apt install fastfetch ffmpeg kitty imagemagick
   ```

   *(Use your distro equivalents — e.g., `brew install fastfetch ffmpeg imagemagick` on macOS.)*

3. **Add images**

   ```bash
   mkdir -p ~/Pictures/neofetch
   # drop your .png/.jpg/.jpeg/.webp files here
   ```

---

## 🖼️ Usage

Run the wrapper instead of Fastfetch:

```bash
~/.config/fastfetch/ff
```

It will:

1. Pick a random image from `~/Pictures/neofetch/` (or any other image directory)
2. Preprocess and cache it under `~/.cache/fastfetch/`
3. Render it inside Kitty using the same cell box as the Fastfetch logo block
4. Run Fastfetch with `config.jsonc`, displaying the image + system info side-by-side

---

## 🔧 Configuration

The image box is controlled by these environment variables:

| Variable | Default | Description |
|-----------|----------|-------------|
| `W` | 32 | width in terminal cells |
| `H` | 16 | height in terminal cells |
| `X` | 1 | horizontal offset (cells) |
| `Y` | 1 | vertical offset (cells) |
| `FIT_MODE` | cover | `cover` crops, `contain` letterboxes |
| `COVER_TOOL` | auto | chooses between `ffmpeg`, `magick`, or `none` |

Example:

```bash
FIT_MODE=contain W=40 H=20 ~/.config/fastfetch/ff
```

---

## 🧠 How it works

1. `ff` selects a random image and preprocesses it to a fixed canvas (`TARGET_WPX` × `TARGET_HPX`) matching your terminal box ratio.
2. It crops/scales the image using FFmpeg or ImageMagick depending on availability.
3. The processed image is cached by SHA1 hash of its parameters.
4. The script pipes the image to Kitty via:

   ```bash
   kitty +kitten icat --transfer-mode=stream --place "${W}x${H}@${X}x${Y}" "$TMP_IMG"
   ```

5. The resulting binary stream is saved to `~/.cache/fastfetch/image.bin`.
6. `config.jsonc` tells Fastfetch to use this binary as a raw logo:

   ```jsonc
   "logo": {
     "type": "raw",
     "source": "~/.cache/fastfetch/image.bin",
     "width": 32,
     "height": 16
   }
   ```

---

## 🧩 Tips

- Run `clear` before `ff` if the image sometimes overlaps old text.
- You can use `kitty +kitten icat --clear` manually to remove leftover images.
- Works perfectly with Fastfetch’s `brightColor` and `color.keys` settings.
- Works only in terminals that support the **Kitty graphics protocol** (Kitty, Ghostty, WezTerm with compatibility layer).

---

## 🧰 Optional tweaks

- Use `FIT_MODE=contain` if you want letterboxed images instead of cropped.
- Set custom image cache size or aspect ratio via:

  ```bash
  TARGET_WPX=900 TARGET_HPX=1600 ~/.config/fastfetch/ff
  ```

- For higher quality, increase `TARGET_WPX`/`TARGET_HPX` (e.g., 2000x2500).

---

## 🧼 Cleanup

To clear the cache:

```bash
rm -rf ~/.cache/fastfetch/*
```

---

## 🩵 Credits

- **Fastfetch** – modern C reimplementation of Neofetch  
- **Kitty** – GPU-accelerated terminal with native image protocol  
- **Neofetch** – original concept inspiration  
- **ffmpeg / ImageMagick** – for preprocessing and cover scaling

---

Enjoy your stylish, image-backed terminal dashboard!
