# OGF CLI

OGF commands inspect X-Ray model files and safely rewrite motion or texture references.

## Commands

| Command                  | Purpose                                               | Writes files |
| ------------------------ | ----------------------------------------------------- | ------------ |
| `ogf info`               | Print header, textures, bones, lods, and motion refs. | No           |
| `ogf patch-motion-refs`  | Replace the motion references stored inside a model.  | Yes          |
| `ogf patch-texture-refs` | Rename a texture reference stored inside a model.     | Yes          |

## `ogf info`

```powershell
xrf-cli ogf info --path ./meshes/example.ogf
```

Options:

- `-p, --path <path>`: path to an `.ogf` file. Required.
- `-s, --silent`: disable logging.
- `-v, --verbose`: turn on verbose logging, which lists each level of detail individually.

### Output

The command reads the model and prints available metadata:

- header version, model type, shader id, bounding box, and bounding sphere;
- texture and shader names;
- description chunk data when present;
- bones and parent names when present;
- motion references when present;
- progressive mesh level of detail counts when present;
- unparsed chunk ids, so an incomplete read is visible rather than silent;
- child model texture, shader names, and level of detail counts for nested OGF data.

Progressive level of detail data usually belongs to the nested child visuals rather than the root, so expect the
`[n] progressive lods` lines rather than a single figure for the model. Pass `-v` to list each level with its index
buffer offset, triangle count, and vertex count.

### When to use it

Use `ogf info` to confirm that a mesh file can be parsed, to inspect texture references, or to compare model metadata
without opening a graphical tool.

The command is read-only: it does not rewrite chunks, normalize paths, or repair model data. If a model fails to parse,
first confirm the file is an OGF from the expected game version, then compare the reported failure with neighboring
meshes from the same source archive.

## `ogf patch-motion-refs`

An animated model stores the paths of the OMF files it loads animations from. This command rewrites those paths, which
is what lets a model be moved to a different directory layout without re-exporting it from the SDK.

```powershell
xrf-cli ogf patch-motion-refs --path ./meshes/wpn_ak74_hud.ogf --refs "dynamics\weapons\wpn_ak74\wpn_ak74_hud_animation"
xrf-cli ogf patch-motion-refs --path ./hands.ogf --dest ./hands.patched.ogf --refs "dynamics\weapons\wpn_hand\hud_animation\*.omf"
xrf-cli ogf patch-motion-refs --path ./hands.ogf --refs "first\animation" "second\animation"
```

Options:

- `-p, --path <path>`: path to an `.ogf` file. Required.
- `-d, --dest <dest>`: path to the resulting file. Defaults to rewriting the source file in place.
- `-r, --refs <refs>...`: one or more motion references to store. Required.
- `--dry-run`: report what the rewrite would produce without writing anything.

Paths use backslashes and omit the `.omf` extension, matching how the engine resolves them. A reference ending in
`\*.omf` is a wildcard: the engine loads every OMF in that directory.

### What is preserved

Only the motion references chunk is rebuilt. Every other chunk, including geometry, bones and IK data, is copied byte
for byte, so patching cannot alter the model itself.

The source file's chunk form is also preserved. Older models store references as one comma-separated string, newer ones
store a counted list; a file keeps whichever form it already used rather than being silently upgraded.

### Safety checks

Model geometry cannot currently be re-serialized from parsed data, so the command verifies rather than assumes:

- Before writing, it rewrites the file's **existing** references and requires the result to reproduce the source byte
  for byte. If it does not, the chunk copy would be lossy for that file and the command refuses to touch it.
- After writing, it reads the result back and requires the references to match what was requested.

If the second check fails, the write is undone: an in-place edit is restored from the original bytes, and a separate
destination file is removed. A model without a motion references chunk is refused before anything is written.

### When to use it

Use it when relocating animation banks, for example when consolidating per-weapon hand animations into one shared
directory and pointing every hands model at it with a wildcard. Confirm the result with `ogf info`.

## `ogf patch-texture-refs`

A model stores the path of every texture it uses. This command renames one of those paths, which is what lets an
imported texture follow your project's naming without re-exporting the model.

```powershell
xrf-cli ogf patch-texture-refs --path ./meshes/wpn_ak74u.ogf --from "wpn\wpn_aksu\wpn_aksu" --to "wpn\wpn_ak74u\wpn_ak74u"
xrf-cli ogf patch-texture-refs --path ./model.ogf --dest ./model.patched.ogf --from "old\name" --to "new\name" --dry-run
```

Options:

- `-p, --path <path>`: path to an `.ogf` file. Required.
- `-d, --dest <dest>`: path to the resulting file. Defaults to rewriting the source file in place.
- `--from <name>`: the texture reference to rename, matched exactly. Required.
- `--to <name>`: the reference to write in its place. Required.
- `--dry-run`: report what the rewrite would produce without writing anything.

It also accepts `-s, --silent` and `-v, --verbose`. Paths use backslashes and omit the extension. Matching is exact;
rename one source name per run.

Texture names live inside nested child visuals. The command walks those children and rebuilds every texture chunk whose
name matches, including when the replacement has a different length.

### What is preserved

Only matching texture chunks are rebuilt. Every other chunk, including geometry, is copied byte for byte; each paired
shader name is preserved.

### Safety checks

The command applies three guards:

- Before writing, it renames the reference to itself and requires a byte-for-byte copy of the source.
- A `--from` that matches nothing is refused, and the error lists the references the model actually has, so a typo
  cannot pass as a silent no-op.
- After writing, it reads the result back and requires the old name to be gone and the new name to be present.

If the final check fails the write is undone: an in-place edit is restored from the original bytes, and a separate
destination file is removed.

### When to use it

Rename the texture files first, patch every model that referenced them, then confirm with `ogf info` and
`gamedata verify`. A missed model leaves a dangling reference.

## Command reference

{{#include reference/ogf.md:commands}}
