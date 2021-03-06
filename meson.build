project(
    'gamemode',
    'c',
    default_options : ['c_std=c11', 'warning_level=3'],
    version: '1.4-dev',
    license: 'BSD',
)

am_cflags = [
    '-fstack-protector',
    '-Wstrict-prototypes',
    '-Wundef',
    '-fno-common',
    '-Werror-implicit-function-declaration',
    '-Wformat-security',
    '-Werror=format-security',
    '-Wconversion',
    '-Wunreachable-code',
]

# Add our main flags
add_global_arguments(am_cflags, language: 'c')

cc = meson.get_compiler('c')

# additional compiler warnings, if supported
test_args = [
  '-Waggregate-return',
  '-Wunused',
  '-Warray-bounds',
  '-Wcast-align',
  '-Wclobbered',
  '-Wempty-body',
  '-Wformat=2',
  '-Wformat-nonliteral',
  '-Wformat-signedness',
  '-Wignored-qualifiers',
  '-Wimplicit-function-declaration',
  '-Winit-self',
  '-Wmissing-format-attribute',
  '-Wmissing-include-dirs',
  '-Wmissing-noreturn',
  '-Wmissing-parameter-type',
  '-Wnested-externs',
  '-Wno-discarded-qualifiers',
  '-Wno-missing-field-initializers',
  '-Wno-suggest-attribute=format',
  '-Wno-unused-parameter',
  '-Wold-style-definition',
  '-Woverride-init',
  '-Wpointer-arith',
  '-Wredundant-decls',
  '-Wreturn-type',
  '-Wshadow',
  '-Wsign-compare',
  '-Wstrict-aliasing=3',
  '-Wstrict-prototypes',
  '-Wstringop-overflow',
  '-Wstringop-truncation',
  '-Wtype-limits',
  '-Wundef',
  '-Wuninitialized',
  '-Wunused-but-set-variable',
  '-Wwrite-strings',
]

foreach arg: test_args
  if cc.has_argument(arg)
    add_global_arguments(arg, language : 'c')
  endif
endforeach


path_prefix = get_option('prefix')
path_bindir = join_paths(path_prefix, get_option('bindir'))
path_datadir = join_paths(path_prefix, get_option('datadir'))
path_includedir = join_paths(path_prefix, get_option('includedir'))
path_libdir = join_paths(path_prefix, get_option('libdir'))
path_libexecdir = join_paths(path_prefix, get_option('libexecdir'))

# Find systemd via pkgconfig
with_systemd = get_option('with-systemd')
dep_systemd = dependency('libsystemd')

# For the client, libdbus is used
dep_dbus = dependency('dbus-1')

# Allow meson to figure out how the compiler sets up threading
dep_threads = dependency('threads')

# On non glibc systems this might be a stub, i.e. for musl
libdl = cc.find_library('dl', required: false)

# Determine the location for the systemd unit
if with_systemd == true
    # If the path isn't explicitly set, ask systemd for the systemd user unit directory
    path_systemd_unit_dir = get_option('with-systemd-user-unit-dir')
    if path_systemd_unit_dir == ''
        message('Asking pkg-config for systemd\'s directories')
        pkgconfig_systemd = dependency('systemd')
        path_systemd_unit_dir = pkgconfig_systemd.get_pkgconfig_variable('systemduserunitdir')
    endif
endif

with_limits_conf = get_option('with-pam-group')
if with_limits_conf != ''
  ldata = configuration_data()
  ldata.set('LIMITSGROUP', with_limits_conf)
  # Install the limits.d configuration file
  configure_file(
      input: 'data/10-gamemode.conf.in',
      output: '10-gamemode.conf',
      configuration: ldata,
      install_dir: '/etc/security/limits.d',
  )
endif

# Set the dbus path as appropriate.
path_dbus_service_dir = get_option('with-dbus-service-dir')
if path_dbus_service_dir == ''
  path_dbus_service_dir = join_paths(path_datadir, 'dbus-1', 'services')
endif

path_polkit_action_dir = join_paths(path_datadir, 'polkit-1', 'actions')

with_daemon = get_option('with-daemon')
with_examples = get_option('with-examples')
with_util = get_option('with-util')

# Provide a config.h
cdata = configuration_data()
cdata.set_quoted('LIBEXECDIR', path_libexecdir)
cdata.set_quoted('GAMEMODE_VERSION', meson.project_version())
config_h = configure_file(
    configuration: cdata,
    output: 'build-config.h',
)
config_h_dir = include_directories('.')

# Library is always required
subdir('lib')

# common lib is always required
subdir('common')

# Utilities are always required except when having both 64 and 32 bit versions
# of libgamemode installed
if with_util == true
    subdir('util')
endif

# The daemon can be disabled if necessary, allowing multilib builds of the
# main library
if with_daemon == true
    # inih currently only needed by the daemon
    inih = subproject('inih')
    inih_dependency = inih.get_variable('inih_dependency')

    subdir('daemon')

    # All installed data is currently daemon specific
    subdir('data')
endif

# Optionally allow building of examples
if with_examples == true
    subdir('example')
endif


report = [
    '    Build configuration:',
    '    ====================',
    '',
    '    prefix:                                 @0@'.format(path_prefix),
    '    bindir:                                 @0@'.format(path_bindir),
    '    datadir:                                @0@'.format(path_datadir),
    '    libdir:                                 @0@'.format(path_libdir),
    '    libexecdir:                             @0@'.format(path_libexecdir),
    '    includedir:                             @0@'.format(path_includedir),
]

if with_systemd == true
report += [
    '    systemd user unit directory:            @0@'.format(path_systemd_unit_dir),
]
endif
report += [
    '    D-BUS service directory:                @0@'.format(path_dbus_service_dir),
]

report += [

    '    PolKit Action Directory:                @0@'.format(path_polkit_action_dir),
    '',
    '    Options:',
    '    ========',
    '',
    '    daemon:                                 @0@'.format(with_daemon),
    '    examples:                               @0@'.format(with_examples),
    '    systemd:                                @0@'.format(with_systemd),
]

# Output some stuff to validate the build config
message('\n\n\n' + '\n'.join(report) + '\n\n')
