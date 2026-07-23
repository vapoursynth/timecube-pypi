project('TIMECUBE', 'cpp',
    default_options: ['buildtype=release', 'warning_level=2', 'b_lto=true', 'b_ndebug=if-release', 'cpp_std=c++17'],
    license: 'WTFPL',
    license_files: 'COPYING',
    meson_version: '>=1.2.3',
    version: '5.0',
)

graphengine_sources = [
	'graphengine/graphengine/cpuinfo.h',
	'graphengine/graphengine/node.h',
	'graphengine/graphengine/state.h',
	'graphengine/graphengine/x86/cpuinfo_x86.h',
	'graphengine/include/graphengine/filter.h',
	'graphengine/include/graphengine/graph.h',
	'graphengine/include/graphengine/namespace.h',
	'graphengine/include/graphengine/types.h',
	'graphengine/graphengine/cpuinfo.cpp',
	'graphengine/graphengine/graph.cpp',
	'graphengine/graphengine/node.cpp',
	'graphengine/graphengine/x86/cpuinfo_x86.cpp'
]

timecube_sources = [
	'timecube/cube.h',
	'timecube/lut.h',
	'timecube/timecube.h',
	'timecube/x86/lut_x86.h',
	'timecube/cube.cpp',
	'timecube/lut.cpp',
	'timecube/timecube.cpp',
	'timecube/x86/lut_x86.cpp',
]

vscube_sources = [
	'vscube/vscube.cpp',
]

vsxx_sources = [
	'vsxx/vapoursynth/VapourSynth4.h',
	'vsxx/vapoursynth/VSConstants4.h',
	'vsxx/vapoursynth/VSHelper4.h',
	'vsxx/VapourSynth4++.hpp',
	'vsxx/vsxx4_pluginmain.h',
    'vsxx/vsxx4_pluginmain.cpp'
]

sources = graphengine_sources + vsxx_sources + timecube_sources + vscube_sources

incdir = include_directories(
    run_command(
        find_program('python', 'python3'),
        '-c',
        'import vapoursynth as vs; print(vs.get_include())',
        check: true,
    ).stdout().strip(),
    'graphengine/include',
    'timecube',
    'vsxx',
    'vsxx/vapoursynth'
)

cxx = meson.get_compiler('cpp')

add_project_arguments(
    cxx.get_supported_arguments(
        '-DGRAPHENGINE_IMPL_NAMESPACE=timecube',
    ),
    language: 'cpp',
)

libs = []

if host_machine.cpu_family() in ['x86', 'x86_64']    
    add_project_arguments(
        cxx.get_supported_arguments(
            '-D_CRT_SECURE_NO_WARNINGS',
            '-DCUBE_X86',
        ),
        language: 'cpp',
    )
    
    libs += static_library('kernel_avx2', 'timecube/x86/lut_avx2.cpp',
        include_directories: incdir,
        cpp_args: ['-mf16c', '-mavx2', '-mfma', '-march=haswell']
    )
    
    libs += static_library('kernel_avx512', 'timecube/x86/lut_avx512.cpp',
        include_directories: incdir,
        cpp_args: ['-mf16c', '-mfma', '-mavx512f', '-mavx512cd', '-mavx512vl', '-mavx512bw', '-mavx512dq', '-mtune=skylake-avx512']
    )
    
    libs += static_library('kernel_sse41', 'timecube/x86/lut_sse41.cpp',
        include_directories: incdir,
        cpp_args: ['-msse4.1']
    )
endif

py = import('python').find_installation(pure: false)

shared_module('timecube',
    sources,
    gnu_symbol_visibility: 'hidden',
    include_directories: incdir,
    link_with: libs,
    install: true,
    install_dir: py.get_install_dir() / 'vapoursynth/plugins',
    name_prefix: '',
)
