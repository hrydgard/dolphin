# -*- python -*-

import os
import sys
import platform

# Home made tests
from SconsTests import utils
from SconsTests import wxconfig

# Some features need at least SCons 1.2
EnsureSConsVersion(1, 2)

# Construction presets for platform
env = Environment(ENV = os.environ)

# Handle command line options
vars = Variables('args.cache')

vars.AddVariables(
    BoolVariable('verbose', 'Set to show compilation lines', False),
    BoolVariable('bundle', 'Set to create distribution bundle', False),
    BoolVariable('lint', 'Set for lint build (fail on warnings)', False),
    BoolVariable('nowx', 'Set for building without wxWidgets', False),
    PathVariable('wxconfig', 'Path to wxconfig', None),
    EnumVariable('flavor', 'Choose a build flavor', 'release', allowed_values =
                ('release','devel','debug','fastlog','prof'), ignorecase = 2),
    )

if env['PLATFORM'] == 'posix': vars.AddVariables(
    PathVariable('destdir', 'Temporary install location (for package building)',
                 None, PathVariable.PathAccept),
    EnumVariable('install', 'Choose a local or global installation', 'local',
                 allowed_values = ('local', 'global'), ignorecase = 2),
    PathVariable('prefix', 'Installation prefix (only used for a global build)',
                 '/usr', PathVariable.PathAccept),
    PathVariable('userdir', 'Set the name of the user data directory in home',
                 '.dolphin-emu', PathVariable.PathAccept),
    EnumVariable('pgo', 'Profile-Guided Optimization (generate or use)', 'none',
                 allowed_values = ('none', 'generate', 'use'), ignorecase = 2),
    BoolVariable('shared_glew', 'Use system shared libGLEW', True),
    BoolVariable('shared_lzo', 'Use system shared liblzo2', True),
    BoolVariable('shared_png', 'Use system shared libpng', True),
    BoolVariable('shared_sdl', 'Use system shared libSDL', True),
    BoolVariable('shared_sfml', 'Use system shared libsfml-network', True),
    BoolVariable('shared_soil', 'Use system shared libSOIL', True),
    BoolVariable('shared_zlib', 'Use system shared libz', True),
    PathVariable('CC', 'The C compiler', 'gcc', PathVariable.PathAccept),
    PathVariable('CXX', 'The C++ compiler', 'g++', PathVariable.PathAccept),
    )

# Save the given command line options
vars.Update(env)
vars.Save('args.cache', env)

# Die on unknown variables
unknown = vars.UnknownVariables()
if unknown:
    print "Unknown variables:", unknown.keys()
    Exit(1)

# Verbose compile
if not env['verbose']:
    env['ARCOMSTR'] = "Archiving $TARGET"
    env['ASCOMSTR'] = "Assembling $TARGET"
    env['ASPPCOMSTR'] = "Assembling $TARGET"
    env['CCCOMSTR'] = "Compiling $TARGET"
    env['CXXCOMSTR'] = "Compiling $TARGET"
    env['LINKCOMSTR'] = "Linking $TARGET"
    env['RANLIBCOMSTR'] = "Indexing $TARGET"
    env['SHCCCOMSTR'] = "Compiling $TARGET"
    env['SHCXXCOMSTR'] = "Compiling $TARGET"
    env['SHLINKCOMSTR'] = "Linking $TARGET"
    env['TARCOMSTR'] = "Creating $TARGET"

if not env['flavor'] == 'debug':
    env['CCFLAGS'] += ['-O3']
if env['flavor'] == 'release':
    env['CCFLAGS'] += ['-fomit-frame-pointer']
elif not env['flavor'] == 'fastlog':
    env['CCFLAGS'] += ['-ggdb']
env['CCFLAGS'] += ['-fno-exceptions', '-fno-strict-aliasing']

if env['lint']:
    env['CCFLAGS'] += ['-Werror']
env['CCFLAGS'] += ['-Wall', '-Wextra', '-Wshadow', '-Wno-unused-parameter']
if env['CCVERSION'] < '4.2.0':
    env['CCFLAGS'] += ['-Wno-pragmas']
if env['CCVERSION'] >= '4.3.0':
    env['CCFLAGS'] += ['-Wno-array-bounds', '-Wno-unused-result']

env['CPPDEFINES'] = []
if env['flavor'] == 'debug':
    env['CPPDEFINES'] += ['_DEBUG']
elif env['flavor'] == 'fastlog':
    env['CPPDEFINES'] += ['DEBUGFAST']
env['CPPPATH'] = []
env['LIBPATH'] = []
env['LIBS'] = []

# Object files
env['build_dir'] = 'Build' + os.sep + platform.system() + \
    '-' + platform.machine() + '-' + env['flavor']

# Default install path
if not env.has_key('install') or env['install'] == 'local':
    env['prefix'] = 'Binary' + os.sep + platform.system() + \
        '-' + platform.machine()
    if env['flavor'] == 'debug' or env['flavor'] == 'prof':
        env['prefix'] += '-' + env['flavor']

rev = utils.GenerateRevFile(env['flavor'], '.', None)

# OS X specifics
if sys.platform == 'darwin':
    #ccld = ['-mmacosx-version-min=10.5.4']
    #ccld += ['-arch', 'x86_64', '-mssse3']
    #ccld += ['-arch', 'i386', '-msse3']
    ccld = ['-arch', 'x86_64', '-arch', 'i386', '-mmacosx-version-min=10.5.4']
    env['CCFLAGS'] += ccld
    env['CCFLAGS'] += ['-Xarch_i386', '-msse3', '-Xarch_x86_64', '-mssse3']
    env['CCFLAGS'] += ['-march=core2', '-mdynamic-no-pic']
    env['CCFLAGS'] += ['-Wextra-tokens', '-Wnewline-eof']
    env['CC'] = '/Developer/usr/bin/clang'
    env['CXX'] = '/Developer/usr/bin/clang++'
    env['CC'] = '/Developer/usr/bin/llvm-gcc'
    env['CXX'] = '/Developer/usr/bin/llvm-g++'
    env['CXXFLAGS'] += ['-x', 'objective-c++']
    env['LINKFLAGS'] += ccld
    env['LINKFLAGS'] += ['-Wl,-dead_strip,-dead_strip_dylibs']
    env['LINKFLAGS'] += ['-Wl,-pagezero_size,0x1000']
    #env['LINKFLAGS'] += ['-Wl,-read_only_relocs,suppress']

    #if float(os.popen('xcode-select -version').read()[21:]) < 2000:
    #    print 'Xcode 4 running on Snow Leopard is required to build Dolphin'
    #    print 'It is available from http://developer.apple.com/devcenter/mac/'
    #    Exit(1)

    if env['ENV'].has_key('CC'):
        env['CC'] = env['ENV']['CC']
    if env['ENV'].has_key('CXX'):
        env['CXX'] = env['ENV']['CXX']

    if env['nowx']:
        env['HAVE_WX'] = 0
    else:
        env['CPPDEFINES'] += ['__WXOSX_COCOA__']
        wxenv = env.Clone(CPPPATH = '', LIBPATH = '', LIBS = '')
        conf = wxenv.Configure(conf_dir = None, log_file = None,
            custom_tests = {'CheckWXConfig' : wxconfig.CheckWXConfig})
        env['HAVE_WX'] = \
            conf.CheckWXConfig(2.92, 'aui adv core base gl'.split(),
                env['flavor'] == 'debug')
        conf.Finish()
        if not env['HAVE_WX']:
            print '\nWARNING:'
            print 'wxWidgets 2.9.2 not found using ' + wxenv['wxconfig']
            print '\nwxWidgets r66814 or newer is required to build Dolphin.'
            print 'See http://code.google.com/p/dolphin-emu/wiki/MacOSX_Build'
            print 'for instructions on building and installing wxWidgets.\n'
        else:
            wxconfig.ParseWXConfig(wxenv)
            env['CPPPATH'] += wxenv['CPPPATH']
            env['LIBS'] += wxenv['LIBS']

    env['data_dir'] = '#' + env['prefix'] + '/Dolphin.app/Contents/Resources'

    if env['bundle']:
        app = env['prefix'] + '/Dolphin.app'
        dmg = env['prefix'] + '/Dolphin-r' + rev + '.dmg'
        env.Command(dmg, app, 'rm -f ' + dmg +
            ' && hdiutil create -srcfolder ' + app + ' -format UDBZ ' + dmg +
            ' && hdiutil internet-enable -yes ' + dmg)

elif sys.platform == 'win32':
    pass

else:
    env['CCFLAGS'] += ['-msse2', '-pthread']
    if env['CCVERSION'] >= '4.2.0':
        env['CCFLAGS'] += ['-fvisibility=hidden']
        env['CXXFLAGS'] += ['-fvisibility-inlines-hidden']
    env['CPPDEFINES'] += ['HAVE_CONFIG_H']
    env['CPPPATH'].insert(0, '#') # Make sure we pick up our own config.h
    env['LINKFLAGS'] += ['-pthread']
    env['RPATH'] = []
    if sys.platform == 'linux2':
        env['CPPDEFINES'] += [('_FILE_OFFSET_BITS', 64), '_LARGEFILE_SOURCE']
        env['CXXFLAGS'] += ['-Wno-deprecated'] # XXX <hash_map>
        env['LIBS'] += ['dl']

    conf = env.Configure(config_h = "#config.h", custom_tests = {
        'CheckPKG'       : utils.CheckPKG,
        'CheckPKGConfig' : utils.CheckPKGConfig,
        'CheckPortaudio' : utils.CheckPortaudio,
        'CheckSDL'       : utils.CheckSDL,
        'CheckWXConfig'  : wxconfig.CheckWXConfig,
    })

    if not conf.CheckPKGConfig('0.15.0'):
        print "Can't find pkg-config, some tests will fail"

    conf.CheckPKG('iconv')

    if env['shared_glew']:
        env['shared_glew'] = conf.CheckPKG('GLEW')
    if env['shared_png']:
        env['shared_png'] = conf.CheckPKG('libpng')
    if env['shared_sdl']:
        env['shared_sdl'] = conf.CheckPKG('SDL')
    if env['shared_zlib']:
        env['shared_zlib'] = conf.CheckPKG('z')
    if env['shared_lzo']:
        env['shared_lzo'] = conf.CheckPKG('lzo2')
    # TODO:  Check the version of sfml.  It should be at least version 1.5
    if env['shared_sfml']:
        env['shared_sfml'] = conf.CheckPKG('sfml-network') and \
                             conf.CheckCXXHeader("SFML/Network/Ftp.hpp")
    if env['shared_soil']:
        env['shared_soil'] = conf.CheckPKG('SOIL')
    for var in env.items():
        if var[0].startswith('shared_') and var[1] == False:
            print "Shared library " + var[0][7:] + " not detected, " \
                  "falling back to the static library"

    if env['nowx']:
        env['HAVE_WX'] = 0
    else:
        if not conf.CheckPKG('gtk+-2.0'):
            print "gtk+-2.0 developement headers not detected"
            print "gtk+-2.0 is required to build the WX GUI"
            Exit(1)
        env['CPPDEFINES'] += ['__WXGTK__']
        conf.Define('HAVE_WX', 1)
        env['HAVE_WX'] = conf.CheckWXConfig(2.8, 'aui adv core base'.split(),
            env['flavor'] == 'debug')
        if env['HAVE_WX']:
            wxconfig.ParseWXConfig(env)
        else:
            print "wxWidgets not found - see config.log"

    env['HAVE_BLUEZ'] = conf.CheckPKG('bluez')
    conf.Define('HAVE_BLUEZ', env['HAVE_BLUEZ'])

    env['HAVE_ALSA'] = conf.CheckPKG('alsa')
    conf.Define('HAVE_ALSA', env['HAVE_ALSA'])
    env['HAVE_AO'] = conf.CheckPKG('ao')
    conf.Define('HAVE_AO', env['HAVE_AO'])
    env['HAVE_OPENAL'] = conf.CheckPKG('openal')
    conf.Define('HAVE_OPENAL', env['HAVE_OPENAL'])
    env['HAVE_PORTAUDIO'] = conf.CheckPortaudio(1890)
    conf.Define('HAVE_PORTAUDIO', env['HAVE_PORTAUDIO'])
    env['HAVE_PULSEAUDIO'] = conf.CheckPKG('libpulse')
    conf.Define('HAVE_PULSEAUDIO', env['HAVE_PULSEAUDIO'])

    env['HAVE_X11'] = conf.CheckPKG('x11')
    env['HAVE_XRANDR'] = env['HAVE_X11'] and conf.CheckPKG('xrandr')
    conf.Define('HAVE_XRANDR', env['HAVE_XRANDR'])
    conf.Define('HAVE_X11', env['HAVE_X11'])

    if not conf.CheckPKG('GL'):
        print "Must have OpenGL to build"
        Exit(1)
    if not conf.CheckPKG('GLU'):
        print "Must have GLU to build"
        Exit(1)
    if not conf.CheckPKG('Cg') and sys.platform == 'linux2':
        print "Must have Cg toolkit from NVidia to build"
        Exit(1)
    if not conf.CheckPKG('CgGL') and sys.platform == 'linux2':
        print "Must have CgGL to build"
        Exit(1)

    # PGO - Profile Guided Optimization
    if env['pgo'] == 'generate':
        env['CCFLAGS'] += ['-fprofile-generate']
        env['LINKFLAGS'] += ['-fprofile-generate']
    if env['pgo'] == 'use':
        env['CCFLAGS'] += ['-fprofile-use']
        env['LINKFLAGS'] += ['-fprofile-use']

    # Profiling
    if env['flavor'] == 'prof':
        proflibs = ['/usr/lib/oprofile', '/usr/local/lib/oprofile']
        env['LIBPATH'] += ['proflibs']
        env['RPATH'] += ['proflibs']
        if conf.CheckPKG('opagent'):
            conf.Define('USE_OPROFILE', 1)
        else:
            print "Can't build prof without oprofile, disabling"

    tarname = 'dolphin-' + rev
    env['TARFLAGS'] = '-cj'
    env['TARSUFFIX'] = '.tar.bz2'

    if env['install'] == 'local':
        env['binary_dir'] = '#' + env['prefix']
        env['data_dir'] = '#' + env['prefix']
        if env['bundle']:
            env.Tar(tarname, env['prefix'])
    else:
        env['prefix'] = Dir(env['prefix']).abspath
        env['binary_dir'] = env['prefix'] + '/bin'
        env['data_dir'] = env['prefix'] + "/share/dolphin-emu"
        conf.Define('DATA_DIR', "\"" + env['data_dir'] + "/\"")
        conf.Define('LIBS_DIR', "\"" + env['prefix'] + '/lib/dolphin-emu/' + "\"")
        # Setup destdir for package building
        # Warning: The program will not run from this location.
        # It is assumed the package will later install it to the prefix.
        if env.has_key('destdir'):
            env['destdir'] = Dir(env['destdir']).abspath
            env['binary_dir'] = env['destdir'] + env['binary_dir']
            env['data_dir'] = env['destdir'] + env['data_dir']
            env['prefix'] = env['destdir'] + env['prefix']
            if env['bundle']:
                env.Command(tarname + env['TARSUFFIX'], env['prefix'],
                    'tar ' + env['TARFLAGS'] + 'C ' + env['destdir'] + \
                    ' -f ' + tarname + env['TARSUFFIX'] + ' ' + \
                    env['prefix'].split(env['destdir'], 1)[1][1:])

    conf.Define('USER_DIR', "\"" + env['userdir'] + "\"")

    # After all configuration tests are done
    conf.Finish()

    env.Alias('install', env['prefix'])

dirs = [
    'Source/Core/Core/Src',
    'Source/Core/Common/Src',
    'Source/Core/DiscIO/Src',
    'Source/Core/DolphinWX/Src',
    'Source/Plugins/Plugin_VideoOGL/Src',
    'Source/Plugins/Plugin_VideoSoftware/Src',
    'Source/Core/AudioCommon/Src',
    'Source/Core/InputCommon/Src',
    'Source/Core/VideoCommon/Src',
    'Source/DSPTool/Src',
    'Source/UnitTests',
    'Externals/Bochs_disasm',
    'Externals/CLRun',
    'Externals/GLew',
    'Externals/LZO',
    'Externals/SDL',
    'Externals/SOIL',
    'Externals/SFML',
    'Externals/libpng',
    'Externals/wxWidgets3',
    'Externals/zlib',
    ]

# Now that platform configuration is done, propagate it to modules
for subdir in dirs:
    SConscript(dirs = subdir, duplicate = 0, exports = 'env',
        variant_dir = env['build_dir'] + os.sep + subdir)
    if subdir.startswith('Source/Core'):
        env['CPPPATH'] += ['#' + subdir]

# Print a nice progress indication when not compiling
Progress(['-\r', '\\\r', '|\r', '/\r'], interval = 5)

# Generate help, printing current status of options
Help(vars.GenerateHelpText(env))
