cmake_minimum_required(VERSION 3.10)
project(mkbootimg C)

# ---- Compiler settings ----
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Compile flags equivalent to CFLAGS from the original Makefile
add_compile_options(-O3 -ffunction-sections -Werror)

# Linker flags, platform-dependent
# (equivalent to the UNAME_S logic in the original Makefile)
if(APPLE)
    add_link_options(-Wl,-dead_strip)
else()
    add_link_options(-Wl,--gc-sections -s)
endif()

# ---- Static library: libmincrypt ----
# Delegates to libmincrypt/CMakeLists.txt.
# This replaces the separate `libmincrypt.a` rule and the `$(CP) $@ ..` copy step.
add_subdirectory(libmincrypt)

# ---- Executable: mkbootimg ----
# Equivalent to:
#   mkbootimg$(EXT): mkbootimg.o libmincrypt.a
#       $(CC) -o $@ $^ -L. -lmincrypt $(LDFLAGS)
add_executable(mkbootimg mkbootimg.c)
target_include_directories(mkbootimg PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(mkbootimg PRIVATE mincrypt)

# ---- Executable: unpackbootimg ----
# Equivalent to:
#   unpackbootimg$(EXT): unpackbootimg.o
#       $(CC) -o $@ $^ $(LDFLAGS)
add_executable(unpackbootimg unpackbootimg.c)
target_include_directories(unpackbootimg PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# ---- Installation ----
# Equivalent to the install target in the Makefile:
#   install -m 755 mkbootimg$(EXT) $(PREFIX)/bin
#   install -m 755 unpackbootimg$(EXT) $(PREFIX)/bin
install(TARGETS mkbootimg unpackbootimg
    RUNTIME DESTINATION bin
)

# ---- Note on the 'static' target ----
# The original Makefile has a 'static' target that rebuilds with -static.
# With CMake, static linking is done at configure time instead:
#   cmake -DCMAKE_EXE_LINKER_FLAGS="-static" ..
# No separate target is needed.
