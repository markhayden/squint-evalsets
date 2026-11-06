<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset=".github/wafflery-mark-dark.svg">
    <img src=".github/wafflery-mark-light.svg" alt="Wafflery" width="120">
  </picture>
</p>

<h1 align="center">wafflery-evalsets</h1>

<p align="center">
  Public evaluation images for the <a href="https://github.com/markhayden/wafflery">Wafflery</a>
  tools. Every file is pinned by SHA-256, so a published result can always be rerun on exactly
  the images it was scored on.
</p>

---

## What this is

Wafflery is a local workspace for trying visual pipeline tools: you run an action, inspect what it
did, and take a tested configuration into your own code. Each tool needs evaluation data that is
honest about how well it works. This repository holds the images for that data:

- **[squint](https://github.com/markhayden/squint)** judges images: whether an edit kept what it
  had to keep, whether a product still matches its packshot, whether a scene is physically
  plausible. Its sets are pass/fail items built so the right answer is known in advance.
- **[rascal](https://github.com/markhayden/rascal)** repairs images. Its remove action masks an
  object named in a prompt, so its sets pair each image with a hand-drawn mask of what a person
  would remove.

The sets are organised by what they test, not by which tool uses them, and either tool can use any
of them. For example, segment-01's images and masks can also judge whether a removal left the
rest of the scene alone, and squint's scenes are useful test images for rascal's segmenter.

## What's here

| Use case | What it tests | Sets | Used by |
|---|---|---|---|
| `source-fidelity/` | an edit kept the source's structure and protected subjects | `scenes-v1`, `blind-v1`, `blind-v2`, `adversarial-v1` | squint |
| `reference-consistency/` | a subject still matches its reference images | `pets-v1`, `adversarial-v1` | squint |
| `product-fidelity/` | a product still matches its packshot: label text, colour, shape | `packshots-v1`, `adversarial-v1` | squint |
| `brief-compliance/` | an image delivers what the brief asked for, and nothing it forbade | `stills-v1`, `adversarial-v1` | squint |
| `reality-checks/` | anatomy, text, shadows and objects are physically plausible | `people-v1`, `adversarial-v1` | squint |
| `segmentation/` | a mask covers the whole named object, its attachments, thin parts and holes | `segment-01` | rascal |

What each set contains (item counts, families, failure types, review notes) is in its manifest:

- **squint's sets:** manifests, jobs, families and confirmed answer labels live in the squint
  repository under [`evalsets/`](https://github.com/markhayden/squint/tree/main/evalsets), with
  a table of every set in its README. This repository holds only their images.
- **Segmentation sets** have no answer labels to keep apart from the images: the truth mask *is*
  the answer, and it is an image. So each one is self-contained here, with a `set.toml` beside
  its items.

## Layout

```
<use-case>/<set>/
  families/<family>/source.png        squint: the source image of a family
  families/<family>/ref-<n>.png       squint: the reference images of a family
  items/<item>/candidate.png          squint: the image an item judges
  items/<item>/image.png              segmentation: the image
  items/<item>/truth.png              segmentation: the hand-drawn truth mask
  set.toml                            segmentation: every item's prompt, mask note, hashes and generation record
```

A truth mask is an 8-bit greyscale PNG the size of its image: 255 where the pixel is part of the
object, 0 where it is not, with no grey. It is drawn by hand from a written note of what to include,
and the note is kept in `set.toml`. For example, *"the tyre and the ropes; the grass seen through
the middle is not part of it"*.

## Using a set

**squint** fetches a set's images beside its manifest. It reads the revision the manifest pins and
refuses any file whose hash differs:

```sh
squint sets fetch --set source-fidelity/scenes-v1
```

**rascal**'s segmenter lab runs each candidate segmenter through rascal's `remove-prompt` recipe
on every `segmentation/` item and scores its mask against `truth.png`. It reports the overlap
(IoU), the share of the object missed, and the spill past a small allowance for shadows and edges.
The lab's method is in rascal's
[`labs/segment-01/`](https://github.com/markhayden/rascal/tree/main/labs/segment-01).

Anything else can use the images directly: clone this repository, or fetch a file from
`https://raw.githubusercontent.com/markhayden/wafflery-evalsets/<revision>/<path>` and check it
against the SHA-256 in its manifest.

## How the images were made

Every image is generated from a text prompt, so no photograph of a real place, product or person is
in the repository. The generators are FLUX.2 [klein] 4B (Black Forest Labs, Apache 2.0 weights,
via fal), OpenAI gpt-image-2 and Gemini 3.1 Flash Image. Each image's model and prompt are
recorded, along with its seed where the API has one.

- **squint's items are built by construction,** so each answer is known before any judge sees it.
  A pass item comes from a permitted change (a colour grade, a new scene for the same subject); a
  fail item comes from a planted defect (a mirror, a removed feature, a lookalike). The owner
  reviews every label from the construction record before it is published, and a set's seal
  binds its content to that review. Generators are mixed across a set's families so no set
  carries one vendor's style. Planted defects are cleaner than real failures, and every result
  that uses these sets says so. The method and the vendor terms are in squint's
  [ADR 0013](https://github.com/markhayden/squint/blob/main/docs/adr/0013-set-generation.md).
- **Segmentation items are generated scenes chosen to be hard for a segmenter.** There are four
  groups: whole objects; objects with attachments (straps, cables, tags); thin parts (poles, wires,
  fishing line); and objects with holes the background shows through. Each truth mask is drawn by
  the owner from the item's note and checked against the image before it is published.

## Rules that keep results honest

- **Published files never change.** A manifest pins a revision and a hash, so editing a file in
  place would break every result scored on it. A fix is a new version of the set (`pets-v2`),
  and the old one stays as it was.
- **Blind sets stay blind.** A blind set's images are public, but its labels are withheld until its
  gate has been scored. Don't publish guesses at them.
- **Note what has been tuned against a set.** A set used to tune a tool is no longer a fair test
  of that tool. A result on a set that has tuned the tool being tested says so.

## Contributing

Ideas for new sets are welcome, especially failure types the existing sets miss.

1. **Open an issue first.** Describe what the set tests, which tool or tools would use it, and
   how its answers are known: by construction, or by hand-drawn truth.
2. **Generate, don't photograph.** Images come from a recorded text prompt with one of the
   generators above, or from another source whose terms allow publishing under CC BY 4.0.
   Don't use photos of real people, private places or someone else's products.
3. **Record everything.** For each image, record the model, the prompt, the seed if there is
   one, every edit instruction, and its SHA-256. For a truth mask, also record the note it was
   drawn from.
4. **Two pull requests.** The images come here, under `<use-case>/<set>/`. A squint set's
   manifest, jobs and labels go to the squint repository, pinned to the commit here. A
   segmentation set's `set.toml` comes here with its items.
5. **The owner reviews every label** before a set is used for a result.

Check a file before you open a pull request:

```sh
shasum -a 256 segmentation/segment-01/items/whole-01/truth.png
```

## Licence

The images are published under [CC BY 4.0](LICENSE). Attribute "wafflery-evalsets, Mark Hayden" and
link to this repository.

This repository was called `squint-evalsets` until 2026-11-05. GitHub redirects the old name.
