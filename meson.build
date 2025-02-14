project('uenv', ['cpp'],
        default_options : [
          'cpp_std=c++20',
          'default_library=static',
          'werror=true',
          'warning_level=3',
        ],
        version: files('VERSION'),
        meson_version: '>=0.57')
version = meson.project_version()

uenv_slurm_plugin = get_option('slurm_plugin')
uenv_cli = get_option('cli')

conf_data = configuration_data()

add_global_arguments('-Wno-missing-field-initializers', language : 'cpp')

# configure the dependencies

# use meson wrap to provide the dependencies, because the 'default_library=static' option
# will be propogated to each dependency, for a statically linked executable.
catch_dep = subproject('catch2', default_options: ['werror=false', 'warning_level=0']).get_variable('catch2_with_main_dep')
cli11_dep = subproject('cli11', default_options: ['werror=false', 'warning_level=0']).get_variable('CLI11_dep')
fmt_dep   = subproject('fmt', default_options: ['werror=false', 'warning_level=0']).get_variable('fmt_dep')
json_dep = subproject('nlohmann_json', default_options: ['werror=false', 'warning_level=0']).get_variable('nlohmann_json_dep')
spdlog_dep   = subproject('spdlog', default_options: ['werror=false', 'warning_level=0','std_format=disabled','external_fmt=enabled']).get_variable('spdlog_dep')
sqlite3_dep = subproject('sqlite3', default_options: ['werror=false', 'warning_level=0']).get_variable('sqlite3_dep')
subproject('curl', default_options: ['werror=false', 'warning_level=0'])

curl_dep = dependency('libcurl', required: true)

# the lib dependency is all of the common funtionality shared between the CLI
# and the slurm plugin.
lib_src = [
        'src/uenv/cscs.cpp',
        'src/uenv/env.cpp',
        'src/uenv/envvars.cpp',
        'src/uenv/lex.cpp',
        'src/uenv/log.cpp',
        'src/uenv/meta.cpp',
        'src/uenv/parse.cpp',
        'src/uenv/repository.cpp',
        'src/uenv/uenv.cpp',
        'src/util/fs.cpp',
        'src/util/shell.cpp',
        'src/util/strings.cpp',
        'src/util/subprocess.cpp',
]
lib_inc = include_directories('src')

lib_dep = declare_dependency(
        sources: lib_src,
        dependencies: [curl_dep, sqlite3_dep, fmt_dep, spdlog_dep, json_dep],
        include_directories: lib_inc
)

subdir('src/uenv')

# the uenv executable
if uenv_cli
    uenv_src = [
            'src/cli/add_remove.cpp',
            'src/cli/color.cpp',
            'src/cli/help.cpp',
            'src/cli/image.cpp',
            'src/cli/inspect.cpp',
            'src/cli/ls.cpp',
            'src/cli/repo.cpp',
            'src/cli/run.cpp',
            'src/cli/start.cpp',
            'src/cli/uenv.cpp',
    ]
    uenv_dep = [sqlite3_dep]

    cli = executable('uenv',
            sources: uenv_src,
            dependencies: [uenv_dep, lib_dep, fmt_dep, spdlog_dep, cli11_dep],
            c_args: ['-DVERSION="@0@"'.format(version)],
            install: true)
endif

unit_src = [
        'test/unit/dates.cpp',
        'test/unit/env.cpp',
        'test/unit/envvars.cpp',
        'test/unit/fs.cpp',
        'test/unit/lex.cpp',
        'test/unit/main.cpp',
        'test/unit/parse.cpp',
        'test/unit/repository.cpp',
        'test/unit/subprocess.cpp',
]

unit = executable('unit',
        sources: unit_src,
        dependencies: [catch_dep, lib_dep],
        build_by_default: true,
        install: false)

if uenv_slurm_plugin
    subdir('src/slurm')
endif
