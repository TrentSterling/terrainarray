# TerrainArray

**Bit-packed splat maps in WebGL 2.** Blend 32+ terrain textures through a single `sampler2DArray` by packing four layer IDs and four blend weights into one RGBA `Float32` value per pixel.

- **Live tutorial:** <https://tront.xyz/terrainarray/>
- **Live demo:** <https://tront.xyz/terrainarray/demo/>

## The trick

Standard splat maps cap at 4 layers (R/G/B/A each weight one texture) and burn one GPU sampler per layer texture. Unity Shader Graph caps out at 16 samplers; fine for hobby work, miles short of any AAA terrain.

Two ideas, composed:

1. **`sampler2DArray`**: one sampler binding holds N textures, sampled with `vec3(uv, index)`.
2. **Bit-packed control map**: write the splat as `RGBA / Float32`, but treat each channel as a bag of bits. Two 16-bit values per channel = 8 values per pixel = 4 layer IDs + 4 weights. Shader reinterprets with `floatBitsToUint`, masks and shifts to get the originals back. Any pixel blends 4 textures from the array of however-many.

Per-pixel cost: 1 control-map sample + 4 array-texture samples + a tiny bit of int math. Same shader scales from 4 layers to 4,000.

```glsl
void unpackFloat(float p, out float a, out float b){
  uint u = floatBitsToUint(p);
  a = float(u >> 16u) / 65535.0;
  b = float(u & 0xFFFFu) / 65535.0;
}
```

The tutorial page has the full walkthrough: JS splat painter, top-4 weight extraction, the GLSL fragment shader, the bilinear-filtering gotcha, plus a verbatim Stack Overflow mirror for the 2015 post the whole technique traces back to.

## Files

| Path | What |
|------|------|
| `index.html` | The tutorial (long-form, 12 sections, embeds the demo via iframe) |
| `demo/index.html` | The standalone WebGL2 demo (32 textures, 200 splats, top-4 extraction, isolation UI) |
| `nuka-message.txt` | Nukadelic's original Unreal shader-graph debug node, 101 lines, copied verbatim |
| `img/` | Screenshots of Nuka's BitShift1 and Pack/Unpack node graphs |

## Credits

- **Nukadelic**: the technique. Shipped it on Meta Quest 2 with `tex2d_array` index packing in a paint texture.
- **Adam Miles**: the `asfloat` / `asuint` pair-packing pattern, [answered on Stack Overflow](https://stackoverflow.com/questions/32915724/pack-two-floats-within-range-into-one-float/32921832#32921832) Oct 3 2015. Mirrored in the tutorial under CC BY-SA 4.0.
- **Duck (SCUG)**: for hitting the wall in public and asking the right question.
- **Trent Sterling**: port to Three.js / WebGL 2, procedural texture library, tutorial page.

## License

- Code in `index.html`, `demo/index.html`, plus the JS helpers: **MIT** (see `LICENSE`).
- Stack Overflow excerpt in `index.html`: reproduced under **CC BY-SA 4.0** with attribution.
- `nuka-message.txt`: reproduced with Nukadelic's permission.
- Screenshots in `img/`: Nukadelic's work, shown for educational context.

If you ship something built on this, drop a note in the [tront.xyz Discord](https://tront.xyz/discord/).
