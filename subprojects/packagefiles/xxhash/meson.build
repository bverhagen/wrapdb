project(
    'xxHash',
    ['c'],
    # library files are BSD licensed. xxhsum utility is GPL licensed.
    license: ['BSD-2-Clause', 'GPL-2.0'],
    version: '0.8.1',
)

cc = meson.get_compiler('c')

force_memory_access = {
    'attribute': 1,
    'unaligned-reads': 2,
    'byteshift': 3,
}

vector = {
    'scalar': 'XXH_SCALAR',
    'sse2': 'XXH_SSE2',
    'avx2': 'XXH_AVX2',
    'avx512': 'XXH_AVX512',
    'neon': 'XXH_NEON',
    'vsx': 'XXH_VSX',
}

c_args = []
c_build_args = []
c_use_args = []

if get_option('inline-all')
    c_args += '-DXXH_INLINE_ALL'
endif

if get_option('namespace') != ''
    c_args += '-DXXH_NAMESPACE=@0@'.format(get_option('namespace'))
endif

if get_option('force-memory-access') != 'default'
    c_args += '-DXXH_FORCE_MEMORY_ACCESS=@0@'.format(force_memory_access[get_option('force-memory-access')])
endif

if get_option('force-align-check')
    c_args += '-DXXH_FORCE_ALIGN_CHECK'
endif

if get_option('vector') != 'default'
    c_args += '-DXXH_VECTOR=@0@'.format(vector[get_option('vector')])
endif

if not get_option('prefetch')
    c_args += '-DXXH_NO_PREFETCH'
endif

if get_option('prefetch-dist') > 0
    c_args += '-DXXH_PREFETCH_DIST=@0@'.format(get_option('prefetch-dist'))
endif

if get_option('inline-hints')
    c_args += '-DXXH_INLINE_HINTS'
endif

if get_option('reroll')
    c_args += '-DXXH_REROLL'
endif

if get_option('accept-null-input-pointer')
    c_args += '-DXXH_ACCEPT_NULL_INPUT_POINTER'
endif

if not get_option('long-long')
    c_args += '-DXXH_NO_LONG_LONG'
endif

if get_option('cpu-little-endian').enabled()
    c_args += '-DXXH_CPU_LITTLE_ENDIAN=1'
elif get_option('cpu-little-endian').disabled()
    c_args += '-DXXH_CPU_LITTLE_ENDIAN=0'
endif

if host_machine.system() == 'windows'
    c_args += '-DWIN32'
endif

if host_machine.system() == 'windows'
    # On Windows/MinGW, we need to define -DXXH_EXPORT and -DXXH_IMPORT
    if get_option('default_library') != 'static'
        c_build_args += '-DXXH_EXPORT'
        c_use_args += '-DXXH_IMPORT'
    endif
    if cc.get_id().contains('clang') 
        c_args += [
            # XXX xxhsum.c:1540:38: warning: comparison of integers of different signs: 'const AlgoSelected' and 'unsigned long long' [-Wsign-compare]
            # see: https://github.com/Cyan4973/xxHash/issues/588
            '-Wno-sign-compare',
            # XXX xxhash.h:3503:1: warning: unused function 'XXH3_accumulate_512_scalar' [-Wunused-function]
            # see: https://github.com/Cyan4973/xxHash/issues/556
            '-Wno-unused-function',
        ]
    endif
endif

pkgconfig_data = configuration_data({
    'PREFIX': get_option('prefix'),
    'EXECPREFIX': get_option('prefix'),
    'INCLUDEDIR': get_option('includedir'),
    'LIBDIR': get_option('libdir'),
    'VERSION': meson.project_version(),
})

pkgconfig = configure_file(
    input: 'libxxhash.pc.in',
    output: 'libxxhash.pc',
    configuration: pkgconfig_data,
)

install_data(
    pkgconfig,
    install_dir: get_option('prefix') / get_option('libdir') / 'pkgconfig'
)

inc = include_directories('.')

xxhash = library(
    'xxhash',
    ['xxhash.c'],
    version: meson.project_version(),
    c_args: c_args + c_build_args,
    include_directories: [
        inc,
    ],
    install: true,
)

# For use is xxhsum only
xxhash_internal_dep = declare_dependency(
    link_with: xxhash,
    compile_args: c_args + c_use_args,
    include_directories: [
        inc,
    ],
)

if get_option('default_library') == 'static'
    c_args += '-DXXH_STATIC_LINKING_ONLY'
endif

xxhash_dep = declare_dependency(
    link_with: xxhash,
    compile_args: c_args,
    include_directories: [
        inc,
    ],
)

if get_option('cli')
    install_man('cli/xxhsum.1')

    cli_c_args = []

    cli_sources = [
        files(
            'cli/xsum_bench.c',
            'cli/xsum_os_specific.c',
            'cli/xsum_output.c',
            'cli/xsum_sanity_check.c',
            'cli/xxhsum.c',
        ),
    ]

    if get_option('dispatch')
        cli_c_args += '-DXXH_DISPATCH'
        cli_sources += files(
            'xxh_x86dispatch.c',
        )
    endif

    xxhsum = executable(
        'xxhsum',
        cli_sources,
        c_args: cli_c_args,
        dependencies: [
            xxhash_internal_dep,
        ],
        install: true,
    )
endif
