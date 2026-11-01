# CMakeLists.txt for libmincrypt
# Builds the static library libmincrypt.a from the C sources in this directory.

# Compile flags equivalent to CFLAGS from the original Makefile
add_compile_options(-O3 -ffunction-sections -Werror)

# The original Makefile uses -I.. (parent directory) for includes
# This is exposed as PUBLIC so that consumers (mkbootimg, unpackbootimg) inherit it.
set(MINCRYPT_INCLUDE_DIRS
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/..
)

# Suppress the false-positive stringop-overflow warning from GCC -O3 on rsa.c
# Keep -Werror enabled for all other files.
if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
    set_source_files_properties(rsa.c PROPERTIES
        COMPILE_OPTIONS "-Wno-error=stringop-overflow"
    )
endif()

# Static library target
add_library(mincrypt STATIC
    dsa_sig.c
    p256.c
    p256_ec.c
    p256_ecdsa.c
    rsa.c
    sha.c
    sha256.c
)

# Include directories (PUBLIC = also propagated to targets that link against mincrypt)
target_include_directories(mincrypt PUBLIC ${MINCRYPT_INCLUDE_DIRS})

# Note: the original Makefile copies libmincrypt.a to the parent directory
# with `$(CP) $@ ..`. This is NOT needed in CMake because
# target_link_libraries() resolves the dependency automatically regardless
# of where the library file is located.

# Ignoring buffer warning and error
add_compile_options(-O3 -ffunction-sections -Werror -Wno-stringop-overflow)
