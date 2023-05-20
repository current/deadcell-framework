<div align="center">

# Deadcell Menu

**Vendored Dear ImGui foundation for the Deadcell menu concept.**

![License](https://img.shields.io/badge/license-MIT-blue)
![Language](https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white)
![Dear ImGui](https://img.shields.io/badge/Dear%20ImGui-1.89.5--WIP-8A2BE2)
![Build](https://img.shields.io/badge/build-CMake-064F8C?logo=cmake&logoColor=white)
![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20DX11-0078D6?logo=windows&logoColor=white)

</div>

## Overview

This repository pins a self-contained snapshot of [Dear ImGui](https://github.com/ocornut/imgui)
(`1.89.5-WIP`) that serves as the rendering/UI layer for the **Deadcell** menu
concept. The tree is vendored rather than pulled as a submodule so that the
exact revision the menu is built against is always known, builds never depend
on network state, and upstream breaking changes can be evaluated on our own
schedule.

The primary targets are **Win32** (platform and input) paired with
**Direct3D 11** (rendering); the remaining upstream backends are kept in-tree
for reference and future portability work.

## Why vendored?

- **Reproducible builds** — the dependency is committed, not resolved at
  configure time.
- **Stable ABI surface** — internal ImGui APIs (`imgui_internal.h`) are used
  heavily by custom widgets; a pinned snapshot guarantees they don't shift
  underneath us.
- **Patching freedom** — upstream fixes can be cherry-picked as needed
  without dragging in unrelated churn.

## Repository layout

```
.
├── imgui.cpp               # Core: context, windows, ID stack, layout
├── imgui.h                 # Public API
├── imgui_internal.h        # Internal API (custom widgets live here)
├── imgui_draw.cpp          # Rendering: vertex buffers, font atlas, primitives
├── imgui_widgets.cpp       # Widgets: sliders, combo boxes, trees, tabs, ...
├── imgui_tables.cpp        # Table API
├── imgui_demo.cpp          # Reference implementations for every widget
├── imconfig.h              # Compile-time configuration
├── backends/               # Platform + renderer backends
│   ├── imgui_impl_win32.*  # Win32 message loop / raw input (primary)
│   ├── imgui_impl_dx11.*   # Direct3D 11 renderer (primary)
│   └── ...                 # DX9/10/12, OpenGL, Vulkan, SDL, GLFW, ...
├── misc/                   # Fonts, freetype binding, std::string helpers,
│   └── debuggers/          #   .natvis / .gdb helpers
├── docs/                   # Upstream documentation (FAQ, FONTS, BACKENDS...)
└── CMakeLists.txt          # Builds imgui as a static library
```

## Core architecture

Dear ImGui is an **immediate-mode** GUI library: there is no retained widget
tree or saved state per widget. Every frame the application calls
`ImGui::NewFrame()`, issues widget calls, and calls `ImGui::Render()`, which
produces a vertex/index buffer that a backend draws. State is keyed by widget
**ID** (derived from labels and pushed ID scopes), which makes dynamic UIs
trivial to express.

| File | Responsibility |
| --- | --- |
| `imgui.cpp` | Context lifecycle, window management, ID stack, input routing, layout |
| `imgui_draw.cpp` | Draw-list generation, font atlas rasterization, anti-aliased primitives |
| `imgui_widgets.cpp` | All stock widgets built on top of the core |
| `imgui_tables.cpp` | Column/table API with freezing, sorting, resizable columns |
| `imgui_internal.h` | Internal API — required for custom widgets and advanced theming |
| `imconfig.h` | Compile-time options (asserts, calling conventions, types) |

## Backends

Backends come in two flavors and are combined freely:

- **Platform backends** feed input (mouse, keyboard, time, display size) and
  own the swapchain window — `imgui_impl_win32` is the one used here.
- **Renderer backends** translate ImGui's draw data into GPU calls —
  `imgui_impl_dx11` is the one used here.

Both compile into the `imgui` static library target; an application links
against it, calls the two `*_Impl_*` `Init` functions, and owns the frame
loop.

## Building

Requires CMake ≥ 3.16 and a C++17 compiler.

```sh
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

On Windows the DX11 backend links against `d3d11`, `dxgi`, and `dwmapi`
automatically.

## Integration

Minimal frame loop once the Win32/DX11 device is set up:

```cpp
ImGui::CreateContext();
ImGui_ImplWin32_Init(hwnd);
ImGui_ImplDX11_Init(device, deviceContext);

// per frame
ImGui_ImplDX11_NewFrame();
ImGui_ImplWin32_NewFrame();
ImGui::NewFrame();

ImGui::Begin("Deadcell");
ImGui::Text("Menu concept");
ImGui::End();

ImGui::Render();
ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData());
```

## Documentation

Upstream documentation ships in [`docs/`](docs/) — the
[FAQ](docs/FAQ.md) and [BACKENDS](docs/BACKENDS.md) pages are the most
useful. `imgui_demo.cpp` contains a live sample of every widget.

## License

Dear ImGui is Copyright (c) 2014–2023 Omar Cornut, licensed under the MIT
license — see [LICENSE](LICENSE). This repository redistributes an unmodified
snapshot; all modifications to the menu itself live in downstream projects.
