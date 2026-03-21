# zt-fonts — Bitmap font blob for zt terminal emulator

Pre-built bitmap font blob for [zt](https://github.com/midasdf/zt), a minimal terminal emulator in Zig.

## Download

```sh
curl -Lo src/fonts/ufo-nf.bin https://github.com/midasdf/zt-fonts/raw/main/ufo-nf.bin
```

## Contents

`ufo-nf.bin` — merged bitmap font blob containing **62,595 glyphs**:

| Source | Glyphs | Coverage |
|--------|--------|----------|
| [UFO](https://github.com/akahuku/ufo) | ~55K | Japanese (Hiragana, Katakana, CJK), Latin, symbols |
| [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts) | ~4.6K | Developer icons (Devicons, Font Awesome, Octicons, Powerline, etc.) |
| [GNU Unifont Upper](https://unifoundry.com/unifont/) | ~3K | Emoji and Plane 1 symbols (16x16 native bitmaps) |

## Font format

8-byte header + glyph table + bitmap data. Each glyph entry is 16 bytes: codepoint (u32), width (u16), height (u16), bitmap offset (u32), bitmap length (u16), padding (u16). Glyphs sorted by codepoint for binary search.

- Half-width glyphs: 8x16 pixels (1 bit per pixel)
- Full-width glyphs: 16x16 pixels (CJK, emoji)

## Build scripts

| Script | Purpose |
|--------|---------|
| `scripts/bdf2blob.py` | Convert BDF font to zt blob format |
| `scripts/merge_blobs.py` | Merge two font blobs (base takes priority) |
| `scripts/hex2blob.py` | Convert GNU Unifont HEX format to zt blob |
| `scripts/ttf2bdf.py` | Render TTF/OTF to BDF (Pillow) |
| `scripts/emoji2bdf.py` | Render NotoColorEmoji to monochrome BDF |

### Rebuild from source

```sh
# 1. UFO base font
git clone --depth 1 https://github.com/akahuku/ufo.git /tmp/ufo
python3 scripts/bdf2blob.py /tmp/ufo/build/ufo.bdf /tmp/ufo.bin

# 2. Nerd Fonts icons (requires a Nerd Font TTF)
python3 scripts/ttf2bdf.py /path/to/NerdFont.ttf /tmp/nf.bdf 16
python3 scripts/bdf2blob.py /tmp/nf.bdf /tmp/nf.bin

# 3. Merge UFO + Nerd Fonts
python3 scripts/merge_blobs.py /tmp/ufo.bin /tmp/nf.bin /tmp/ufo-nf-base.bin

# 4. GNU Unifont Upper emoji
curl -sL "https://unifoundry.com/pub/unifont/unifont-16.0.02/font-builds/unifont_upper-16.0.02.hex.gz" | gunzip > /tmp/unifont_upper.hex
python3 scripts/hex2blob.py /tmp/unifont_upper.hex /tmp/emoji.bin \
  --range 2600-27BF --range 2900-2BFF \
  --range 1F000-1F02F --range 1F0A0-1F0FF \
  --range 1F100-1F1FF --range 1F200-1F2FF \
  --range 1F300-1F9FF --range 1FA00-1FAFF \
  --range 1FB00-1FBFF

# 5. Merge emoji (priority) + base
python3 scripts/merge_blobs.py /tmp/emoji.bin /tmp/ufo-nf-base.bin ufo-nf.bin
```

## Licenses

See [FONT_LICENSES.md](FONT_LICENSES.md) for detailed attribution.

- UFO: GPL-2.0+ with font embedding exception
- GNU Unifont Upper: GPL-2.0+ with font embedding exception
- Nerd Fonts: MIT / OFL-1.1 / CC-BY-4.0 / Apache-2.0 (per glyph set)
