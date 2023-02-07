cmake_minimum_required(VERSION 3.16)
project(deadcell-menu LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Dear ImGui core + the backends targeted by the Deadcell menu
# (Win32 for platform/input, Direct3D 11 for rendering).
add_library(imgui STATIC
    imgui.cpp
    imgui_demo.cpp
    imgui_draw.cpp
    imgui_tables.cpp
    imgui_widgets.cpp
    backends/imgui_impl_dx11.cpp
    backends/imgui_impl_win32.cpp
)

target_include_directories(imgui PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/backends
)

if(WIN32)
    target_link_libraries(imgui PUBLIC d3d11 dxgi dwmapi)
endif()
