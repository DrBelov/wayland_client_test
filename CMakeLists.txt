cmake_minimum_required(VERSION 3.5.0)
project(xdg_shell_basics VERSION 0.1.0 LANGUAGES C CXX)

# Find wayland-scanner tool
find_program(WAYLAND_SCANNER NAMES wayland-scanner REQUIRED)

# Define paths for the protocol and generated files
set(PROTOCOL_DIR "/usr/share/wayland-protocols/stable/xdg-shell")
set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(PROTOCOL_XML "${PROTOCOL_DIR}/xdg-shell.xml")
set(GENERATED_HEADER "${GENERATED_DIR}/xdg-shell-client-protocol.h")
set(GENERATED_CODE "${GENERATED_DIR}/xdg-shell-protocol.c")

# Create the output directory
file(MAKE_DIRECTORY ${GENERATED_DIR})

# Generate the client header file
add_custom_command(
    OUTPUT ${GENERATED_HEADER}
    COMMAND ${WAYLAND_SCANNER} client-header < ${PROTOCOL_XML} > ${GENERATED_HEADER}
    DEPENDS ${PROTOCOL_XML}
    COMMENT "Generating client header from ${PROTOCOL_XML}"
)
add_custom_target(wayland_client_header DEPENDS ${GENERATED_HEADER})

# Generate the client source file
add_custom_command(
    OUTPUT ${GENERATED_CODE}
    COMMAND ${WAYLAND_SCANNER} private-code < ${PROTOCOL_XML} > ${GENERATED_CODE}
    DEPENDS ${PROTOCOL_XML}
    COMMENT "Generating protocol C source from ${PROTOCOL_XML}"
)
add_custom_target(protocol_source DEPENDS ${GENERATED_CODE})

add_executable(xdg_shell_basics xdg_shell_basics.c ${GENERATED_CODE})
add_dependencies(xdg_shell_basics wayland_client_header)
target_include_directories(xdg_shell_basics PRIVATE ${GENERATED_DIR})
target_link_libraries(xdg_shell_basics PRIVATE wayland-client rt)