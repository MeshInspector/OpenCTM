# OpenCTM (MeshInspector fork)

OpenCTM is a file format and a C library for compressing 3D triangle meshes, written by Marcus Geelnard and released under the zlib/libpng license. This repository is the fork that [MeshLib](https://github.com/MeshInspector/MeshLib) uses for reading and writing `.ctm` files.

It started from upstream **v1.0.3 (2010-01-15)**, the last release on [sourceforge.net/projects/openctm](https://sourceforge.net/projects/openctm/). The original README with the upstream change log and credits is kept verbatim as [README.txt](README.txt); the file format specification is [doc/FormatSpecification.pdf](doc/FormatSpecification.pdf).

## What differs from upstream 1.0.3

### Scope and build

- **Library only.** Only `lib/` (the OpenCTM sources plus the bundled LZMA SDK) is kept. The `ctmconv` and `ctmviewer` tools, the language bindings, the Maya/Blender plugins and the upstream Makefiles are removed.
- **CMake build.** `CMakeLists.txt` builds a shared `OpenCTM` target and installs headers, the library and an `OpenCTMConfig.cmake` package, so consumers use `find_package(OpenCTM CONFIG)` and link `OpenCTM::OpenCTM`. Installation rules can be disabled with `-DOPENCTM_INSTALL=OFF`.
- **Export macro.** Symbol export is driven by CMake's `OpenCTM_EXPORTS` define instead of upstream's `OPENCTM_BUILD`; `OPENCTM_STATIC` still selects static linkage. On non-Windows builds every API function carries default visibility.

### API additions and changes

- **`ctmRearrangeTriangles(context, on)`** (new). Upstream always sorts triangles before MG1/MG2 compression to improve the ratio, which changes their order in the file. The fork keeps that as the default, but turning it off preserves the caller's triangle order. In both modes each triangle is still rotated so that its smallest vertex index comes first, which does not change the mesh.
- **Compression progress and cancellation.** `ctmSaveCustom` gained a third parameter, a `CTMcompressProgress` callback `(size_t pos, size_t total, void * userData)` that is called during LZMA compression with the number of bytes processed so far. Returning a non-zero value aborts the save, which then fails with `CTM_LZMA_ERROR`. Pass `NULL` for the upstream behaviour. `ctmSave` is unchanged. Existing `ctmSaveCustom` call sites need the extra `NULL` argument.
- **Empty meshes.** A mesh with zero vertices and/or zero triangles can be defined, saved and loaded. Upstream rejects such data with `CTM_INVALID_MESH` on save and `CTM_BAD_FORMAT` on load, so files of empty meshes written by this fork are not readable by the original library.
- **C++ wrapper.** `openctmpp.h` uses `noexcept` instead of the removed `throw()` specification.

### Loader hardening

Fixes for two heap-buffer overflows reachable from `ctmLoad` on a crafted file, reported by Kamal Sentassi (S9S Security Research) under coordinated disclosure in 2026:

- **String lengths** ([#2](https://github.com/MeshInspector/OpenCTM/pull/2)). A file-declared string length of `0xFFFFFFFF` made `malloc(len + 1)` wrap to a zero-size buffer that was then overrun. Such lengths and short reads are rejected, and the loader stops parsing after the first failed read instead of continuing with an error already set. `ctmLoadCustom` also clears a stale error from a previous load on the same context.
- **Vertex and triangle counts** ([#3](https://github.com/MeshInspector/OpenCTM/pull/3)). File-declared counts drove `malloc(count * stride)` directly, which wraps on a 32-bit `size_t` (wasm32) and in the 32-bit intermediates of the temporary buffers. Counts above `UINT_MAX / 16` (268,435,455) are rejected with `CTM_BAD_FORMAT`.

Other small fixes: compiler warnings, line endings, typos, and a Windows install rule for the DLL.

### File format

The format itself is unchanged (format version 5). Files written by this fork load in upstream OpenCTM and vice versa, with the two exceptions noted above: empty meshes, and meshes above the count limit.

## License

zlib/libpng, unchanged from upstream. See [LICENSE.txt](LICENSE.txt).
