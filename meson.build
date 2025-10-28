project('libserialport', 'c',
  version: '0.1.2',
  default_options: [
    'warning_level=2',
    'c_std=c99',
    'b_vscrt=md',
    'default_library=shared'
  ])

cc = meson.get_compiler('c')

# Version info
sp_version = meson.project_version()
version_parts = sp_version.split('.')
sp_version_major = version_parts[0]
sp_version_minor = version_parts[1]
sp_version_micro = version_parts[2]

# Library version (libtool format: current:revision:age)
sp_lib_current = 1
sp_lib_revision = 1
sp_lib_age = 1
sp_lib_version = '@0@:@1@:@2@'.format(sp_lib_current, sp_lib_revision, sp_lib_age)

# Platform detection
host_os = host_machine.system()
is_linux = host_os == 'linux'
is_windows = host_os == 'windows'
is_darwin = host_os == 'darwin'
is_freebsd = host_os == 'freebsd'

# Configuration data
conf_data = configuration_data()
conf_data.set('SP_PACKAGE_VERSION_MAJOR', sp_version_major)
conf_data.set('SP_PACKAGE_VERSION_MINOR', sp_version_minor)
conf_data.set('SP_PACKAGE_VERSION_MICRO', sp_version_micro)
conf_data.set_quoted('SP_PACKAGE_VERSION_STRING', sp_version)
conf_data.set('SP_LIB_VERSION_CURRENT', sp_lib_current)
conf_data.set('SP_LIB_VERSION_REVISION', sp_lib_revision)
conf_data.set('SP_LIB_VERSION_AGE', sp_lib_age)
conf_data.set_quoted('SP_LIB_VERSION_STRING', sp_lib_version)

# Platform-specific checks
if is_linux
  if cc.has_type('struct termios2', prefix: '#include <linux/termios.h>')
    conf_data.set('HAVE_STRUCT_TERMIOS2', 1)
  endif
  
  if cc.has_member('struct termios', 'c_ispeed', prefix: '#include <linux/termios.h>')
    conf_data.set('HAVE_STRUCT_TERMIOS_C_ISPEED', 1)
  endif
  
  if cc.has_member('struct termios', 'c_ospeed', prefix: '#include <linux/termios.h>')
    conf_data.set('HAVE_STRUCT_TERMIOS_C_OSPEED', 1)
  endif
  
  if cc.has_member('struct termios2', 'c_ispeed', prefix: '#include <linux/termios.h>')
    conf_data.set('HAVE_STRUCT_TERMIOS2_C_ISPEED', 1)
  endif
  
  if cc.has_member('struct termios2', 'c_ospeed', prefix: '#include <linux/termios.h>')
    conf_data.set('HAVE_STRUCT_TERMIOS2_C_OSPEED', 1)
  endif
  
  if cc.has_header_symbol('linux/termios.h', 'BOTHER')
    conf_data.set('HAVE_DECL_BOTHER', 1)
  else
    conf_data.set('HAVE_DECL_BOTHER', 0)
  endif
  
  if cc.has_type('struct serial_struct', prefix: '#include <linux/serial.h>')
    conf_data.set('HAVE_STRUCT_SERIAL_STRUCT', 1)
  endif
  
  if conf_data.has('HAVE_STRUCT_TERMIOS_C_ISPEED') and conf_data.has('HAVE_STRUCT_TERMIOS_C_OSPEED')
    conf_data.set('HAVE_TERMIOS_SPEED', 1)
  endif
  
  if conf_data.has('HAVE_STRUCT_TERMIOS2_C_ISPEED') and conf_data.has('HAVE_STRUCT_TERMIOS2_C_OSPEED')
    conf_data.set('HAVE_TERMIOS2_SPEED', 1)
  endif
endif

if is_darwin
  if not cc.has_header('IOKit/IOKitLib.h')
    error('IOKit/IOKitLib.h not found')
  endif
endif

if cc.has_function('realpath')
  conf_data.set('HAVE_REALPATH', 1)
endif

if cc.has_header('sys/file.h')
  conf_data.set('HAVE_SYS_FILE_H', 1)
endif

if cc.has_function('flock')
  conf_data.set('HAVE_FLOCK', 1)
endif

if cc.has_function('clock_gettime')
  conf_data.set('HAVE_CLOCK_GETTIME', 1)
endif

# Check for visibility control
visibility_code = '''
__attribute__((visibility("hidden"))) void foo(void) {}
'''
if cc.compiles(visibility_code, name: 'visibility attribute')
  conf_data.set('SP_API', '__attribute__((visibility("default")))')
  conf_data.set('SP_PRIV', '__attribute__((visibility("hidden")))')
else
  # For static libs, SP_API should be empty
  if get_option('default_library') == 'static'
    conf_data.set('SP_API', '')
    conf_data.set('SP_PRIV', '')
  else
    declspec_code = '__declspec(dllexport) void foo(void) {}'
    if cc.compiles(declspec_code, name: 'declspec')
      conf_data.set('SP_API', '__declspec(dllexport)')
      conf_data.set('SP_PRIV', '')
    else
      conf_data.set('SP_API', '')
      conf_data.set('SP_PRIV', '')
    endif
  endif
endif

if not cc.has_type('size_t', prefix: '#include <stddef.h>')
  conf_data.set('size_t', 'unsigned int')
endif

if not (is_linux or is_darwin or is_windows or is_freebsd)
  conf_data.set('NO_ENUMERATION', 1)
  conf_data.set('NO_PORT_METADATA', 1)
endif

# Generate config.h
config_h = configure_file(
  output: 'config.h',
  configuration: conf_data
)

# Compiler flags
c_args = ['-DLIBSERIALPORT_ATBUILD', '-DLIBSERIALPORT_MSBUILD']

if cc.get_id() == 'msvc'
  c_args += [
    '-DWIN32_LEAN_AND_MEAN',
    '-DNOMINMAX',
    '-D_CRT_SECURE_NO_WARNINGS',
    '-D_WIN32_WINNT=0x0601'
  ]
else
  c_args += [
    '-Wall',
    '-Wextra',
    '-pedantic',
    '-Wmissing-prototypes',
    '-Wshadow'
  ]
  if is_windows
    c_args += [
      '-DWIN32_LEAN_AND_MEAN',
      '-DNOMINMAX',
      '-D_WIN32_WINNT=0x0601'
    ]
  endif
endif

# Core sources
sources = [
  'serialport.c',
  'timing.c'
]

# Platform-specific sources
if is_linux
  sources += ['linux.c', 'linux_termios.c']
elif is_windows
  sources += ['windows.c']
elif is_darwin
  sources += ['macosx.c']
elif is_freebsd
  sources += ['freebsd.c']
endif

# Dependencies
deps = []

if is_windows
  deps += cc.find_library('setupapi')
  deps += cc.find_library('Ws2_32', required: false)
elif is_darwin
  deps += dependency('appleframeworks', modules: ['IOKit', 'CoreFoundation'])
elif is_freebsd
  deps += dependency('libusb-2.0', required: false)
endif

# Include directories
inc = include_directories('.')

# Build library
libserialport = library('serialport',
  sources,
  config_h,
  c_args: c_args,
  dependencies: deps,
  include_directories: inc,
  version: sp_version,
  soversion: sp_lib_current - sp_lib_age,
  install: true
)

# Install header
install_headers('libserialport.h')

# pkg-config
pkg = import('pkgconfig')
pkg_libs = []
if is_windows
  pkg_libs = ['-lsetupapi', '-lWs2_32']
endif

pkg_requires = []
if is_freebsd
  pkg_requires = ['libusb-2.0']
endif

pkg.generate(
  libserialport,
  name: 'libserialport',
  description: 'Cross-platform library for accessing serial ports',
  version: sp_version,
  libraries: pkg_libs,
  requires: pkg_requires
)

# Test (skip on Windows - uses unistd.h, skip on macOS - flaky timing)
if not is_windows and not is_darwin
  test_timing = executable('test_timing',
    ['timing.c', 'test_timing.c'],
    c_args: c_args,
    include_directories: inc,
    install: false
  )
  test('timing', test_timing)
endif

# Examples (skip on Windows - need porting)
if get_option('examples') and not is_windows
  subdir('examples')
endif

# Summary
summary({
  'Version': sp_version,
  'Library version': sp_lib_version,
  'Platform': host_os,
}, section: 'Configuration')

summary({
  'Linux': is_linux,
  'Windows': is_windows,
  'macOS': is_darwin,
  'FreeBSD': is_freebsd,
}, section: 'Platform', bool_yn: true)
