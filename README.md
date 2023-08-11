# deadcell menu

imgui foundation for the deadcell menu. this is a pinned snapshot of
[dear imgui](https://github.com/ocornut/imgui) (`1.89.5-wip`) — everything the
menu draws, from the window frames down to the sliders and tabs, goes through
this code and out the back into dx11.

the tree is vendored on purpose, not pulled as a submodule:

- builds can't break because something moved upstream. the revision the menu
  compiles against is the one sitting in the repo, always.
- custom widgets lean on `imgui_internal.h`, and that header shifts a lot.
  pinning it means it can't get pulled out from under us mid-development.
- upstream fixes get cherry-picked when we want them, not when they show up
  with six months of unrelated churn attached.

## what's in here

```
imgui.cpp             core — context, windows, id stack, input, layout
imgui.h               public api
imgui_internal.h      internal api, the one custom widgets actually need
imgui_draw.cpp        vertex buffers, font atlas, antialiased primitives
imgui_widgets.cpp     sliders, combos, trees, tabs, all the stock widgets
imgui_tables.cpp      table api
imgui_demo.cpp        live sample of every widget, with source. read it.
imconfig.h            compile-time toggles
backends/             platform + renderer backends (see below)
misc/                 fonts, freetype, std::string helpers, .natvis files
docs/                 upstream docs — faq and BACKENDS.md are the good ones
CMakeLists.txt        builds imgui as a static lib
```

## how it works

imgui is immediate mode. there's no widget tree and no per-widget state
hanging around between frames — you call `NewFrame()`, issue your widget
calls, call `Render()`, and out comes a vertex/index buffer a backend shoves
into the gpu. every frame. that's the whole trick, and it's why dynamic uis
(quick-loading configs, iterating over a vector of features) are one-liners
instead of bookkeeping.

widget identity comes from labels plus whatever's on the id stack, so two
`InputInt` calls in a loop don't collide as long as you push an id per item.
mostly you never think about it. sometimes you do, and that's what
`imgui_internal.h` is for.

| file | does |
| --- | --- |
| `imgui.cpp` | windows, id stack, input routing, layout |
| `imgui_draw.cpp` | draw lists, font atlas rasterization |
| `imgui_widgets.cpp` | stock widgets, built on the core |
| `imgui_tables.cpp` | tables — scroll, sort, freeze, resize |
| `imgui_internal.h` | internals: custom widgets, theming hooks |

## backends

two kinds, mix and match:

- **platform** — feeds input and owns the window. win32 here.
  `ImGui_ImplWin32_WndProcHandler` goes in your wndproc and input just works.
- **renderer** — turns draw data into gpu calls. dx11 here.

win32 + dx11 is the pair that actually gets exercised. everything else in
`backends/` (dx9/10/12, opengl, vulkan, sdl2/sdl3, glfw, metal...) is vendored
as-is for reference — fine to compile against, just not battle-tested on this
snapshot.

both primaries compile straight into the `imgui` static target, and the
dx11 side links `d3d11`/`dxgi`/`dwmapi` on its own when building on windows.

## building

cmake >= 3.16, c++17 compiler:

```
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

## wiring it up

once you've got a device and a window:

```cpp
ImGui::CreateContext();
ImGui_ImplWin32_Init(hwnd);
ImGui_ImplDX11_Init(device, device_context);

// frame loop
ImGui_ImplDX11_NewFrame();
ImGui_ImplWin32_NewFrame();
ImGui::NewFrame();

ImGui::Begin("deadcell");
// menu contents
ImGui::End();

ImGui::Render();
ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData());
```

that's it. no scene graph, no callbacks to register.

## license

mit, © omar cornut — see [LICENSE](LICENSE). this is an unmodified snapshot of
upstream; the menu itself lives in downstream code.
