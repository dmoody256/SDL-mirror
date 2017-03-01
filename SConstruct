# This file is licensed under the MIT License.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

import os
import json
import sys
import glob
import re
from sys import platform

subsystems = [
    'atomic',
    'audio',
    'video',
    'render',
    'events',
    'joystick',
    'haptic',
    'power',
    'threads',
    'timers',
    'file',
    'loadso',
    'cpuinfo',
    'filesystem',
    'dlopen',
]

def CreateNewEnv():

    CreateOptions()
    #TODO dynamic susystem options
    #options_List =

    env = Environment(
            DEBUG_BUILD = GetOption('debug_build'),
            ALSA_SHARED = GetOption('alsa-shared'),
            ARTS_SHARED = GetOption('arts-shared'),
            ASSEMBLY = GetOption('disable-asm'),
            ASSERTIONS = GetOption('assertions'),
            CLOCK_GETTIME = GetOption('clock_gettime'),
            FORCE_STATIC_VCRT = GetOption('force_static_vcrt'),
            TARGET_ARCH = GetOption('arch'),
        );


    baseProjectDir = os.path.abspath(Dir('.').abspath).replace('\\', '/')
    print("BASE_DIR="+baseProjectDir)
    env.VariantDir(baseProjectDir +'/build', baseProjectDir +'/src', duplicate=0)

    env = ConfigPlatformEnv(env, baseProjectDir)

    sourceFiles = []
    headerFiles = []

    sourceFiles.extend(glob.glob(baseProjectDir +'/src/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/atomic/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/audio/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/cpuinfo/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/dynapi/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/events/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/file/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/libm/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/render/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/render/*/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/stdlib/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/thread/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/timer/*.c'))
    sourceFiles.extend(glob.glob(baseProjectDir +'/src/video/*.c'))

    #if(env['SDL_JOYSTICK')):
    #    sourceFiles.extend(glob.glob(baseProjectDir +'/src/joystick/*.c'))

    sourceFiles = [file.replace("src", 'build') for file in sourceFiles]



    headerFiles.extend(glob.glob(baseProjectDir +'/src/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/atomic/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/audio/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/cpuinfo/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/dynapi/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/events/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/file/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/libm/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/render/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/render/*/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/stdlib/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/thread/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/timer/*.h'))
    headerFiles.extend(glob.glob(baseProjectDir +'/src/video/*.h'))

    env.Append(CPPPATH=[
        baseProjectDir + "/include",
    ])


    prog = env.SharedLibrary("build/SDL", sourceFiles)
    env = ConfigPlatformIDE(env, sourceFiles, headerFiles, prog)

    return env

def ConfigPlatformEnv(env, baseProjectDir):

    _3dnow_test_source_file = """
        #include <mm3dnow.h>
        #if define(__GNUC__) || define(__clang__)
        #ifndef __3dNOW__
        #error Assembler CPP flag not enabled
        #endif
        #endif
        int main(int argc, char **argv) {
            void *p = 0;
            _m_prefetch(p);
        }
    """

    altivec_h_test_source_file = """
        #include <altivec.h>
        vector unsigned int vzero() {
            return vec_splat_u32(0);
        }
        int main(int argc, char **argv) { }
    """

    altivec_test_source_file = """
        vector unsigned int vzero() {
            return vec_splat_u32(0);
        }
        int main(int argc, char **argv) { }
    """

    def FindRealLibrary(env, libname):
        libpath = ""
        for path in env["LIBPATH"]:
            libpath = path + "/" + env['SHLIBPREFIX'] + libname + env['SHLIBSUFFIX']
            if(os.path.exists(libpath)):
                libpath = os.path.realpath(libpath)
                break
        return libpath.replace("\\", "/")

    def Check_3DNow(context):
        context.Message('Checking for 3DNow... ')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            context.env.Append(CCFLAGS=['-m3dnow'])
        result = context.TryCompile(_3dnow_test_source_file, '.c')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            env.Replace(CCFLAGS = [flag for flags in env['CCFLAGS'] if flag not in ['-m3dnow']])
        context.Result(result)
        return result

    def Check_Altivec(context):
        context.Message('Checking for Altivec... ')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            context.env.Append(CCFLAGS=["-maltivec"])
        result = context.TryCompile(altivec_test_source_file, '.c')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            env.Replace(CCFLAGS = [flag for flags in env['CCFLAGS'] if flag not in ['-maltivec']])
        context.Result(result)
        return result

    def Check_Altivec_H(context):
        context.Message('Checking for Altivec with Header... ')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            context.env.Append(CCFLAGS=["-maltivec"])
        result = context.TryCompile(altivec_h_test_source_file, '.c')
        if('__GNUC__' in context.env or '__clang__' in conf.env):
            env.Replace(CCFLAGS = [flag for flags in env['CCFLAGS'] if flag not in ['-maltivec']])
        context.Result(result)
        return result

    def Check_ARTS(context):
        context.Message('Checking for ARTS...')
        results = context.TryAction('artsc-config --cflags')
        context.Result(results[0])
        return results[0]

    def ConfigureARTS(conf):
        if(conf.Check_ARTS()):
            results = context.TryAction('artsc-config --cflags')
            if(results[0]):
                for flag in results[1].split(' '):
                    if(flag.startswith('-I')):
                        context.env.Append(CPPPATH=[flag[2:]])
                    else:
                        context.env.Append(CCFLAGS=[flag])

                libResults = context.TryAction('artsc-config --libs')
                for flag in libResults[1].split(' '):
                    if(flag.startswith('-L')):
                        context.env.Append(LIBPATH=[flag[2:]])
                    elif(flag.startswith('-l')):
                        context.env.Append(LIBS=[flag[2:]])
                    else:
                        context.env.Append(LDFLAGS=[flag])
            context.env.Append(CPPDEFINES=[
                'SDL_AUDIO_DRIVER_ARTS=1',
                'HAVE_ARTS=1',
                'HAVE_SDL_AUDIO=1',
            ])
            if('ARTS_SHARED' in conf.env):
                if('HAVE_DLOPEN' not in conf.env or conf.env['HAVE_DLOPEN'] == 0):
                    print("You must have SDL_LoadObject() support for dynamic ARTS loading")
                else:
                    conf.env.Append(CPPDEFINES=['SDL_AUDIO_DRIVER_ARTS_DYNAMIC='+FindRealLibrary(env, 'artsc')])
                    conf.env.Append(CPPDEFINES=['HAVE_ARTS_SHARED=1'])
            print ('Configured for ARTS')
        else:
            print ('Missing ARTS, continuing without it...')

    def Configure3DNow(conf):
        if conf.Check_3DNow():
            if('__GNUC__' in conf.env or '__clang__' in conf.env):
                conf.env.Append(CCFLAGS=["-m3dnow"])
            conf.env.Append(CPPDEFINES=['HAVE_3DNOW'])
            print ('Configured for 3DNow')
        else:
            print ('Missing 3DNow, contuning without it...')

    def ConfigureDLOPEN(conf):
        if conf.CheckFunc('dlopen'):
            conf.env.Append(CPPDEFINES=['HAVE_DLOPEN=1'])
            print("Dynamic Linking Available...")
        else:
            for lib in ['dl', 'tdl']:
                if(conf.CheckLib(lib, 'dlopen')):
                    conf.env['_DLLIB'] = lib
                    conf.env.Append(LIBS=[lib])
                    conf.env.Append(CPPDEFINES=['HAVE_DLOPEN=1'])
                    print ("Dynamic Linking Available...")
                    break
            if 'CPPDEFINES' in conf.env and 'HAVE_DLOPEN' not in conf.env['CPPDEFINES']:
                conf.env.Append(CPPDEFINES=['HAVE_DLOPEN=0'])
                print ("Dynamic Linking NOT Available...")

    def ConfigureALSA(conf):

        if conf.CheckLib('asound', 'snd_pcm_recover','alsa/asoundlib.h'):
            conf.env.Append(CPPDEFINES=[
                'HAVE_ALSA=1',
                'HAVE_LIBASOUND=1',
                'SDL_AUDIO_DRIVER_ALSA=1',
            ])
            if('ALSA_SHARED' in conf.env):
                if('HAVE_DLOPEN' not in conf.env or conf.env['HAVE_DLOPEN'] == 0):
                    print("You must have SDL_LoadObject() support for dynamic ALSA loading")
                else:
                    conf.env.Append(CPPDEFINES=['SDL_AUDIO_DRIVER_ALSA_DYNAMIC='+FindRealLibrary(env, 'asound')])
                    conf.env.Append(CPPDEFINES=['HAVE_ALSA_SHARED=1'])

            conf.env.Append(CPPDEFINES=['HAVE_SDL_AUDIO=1'])
            print ('Configured for ALSA')
        else:
            print ('Missing ALSA, contuning without it...')

    def ConfigureAltivec(conf):

        have_altivec = conf.Check_Altivec()
        have_altivec_h = conf.Check_Altivec_H()
        if(have_altivec or have_altivec_h):
            if('__GNUC__' in conf.env or '__clang__' in conf.env):
                conf.env.Append(CCFLAGS=["-maltivec"])
            conf.env.Append(CPPDEFINES=[
                'HAVE_ALTIVEC=1',
                'SDL_ALTIVEC_BLITTERS=1'
            ])
        if have_altivec_h:
            conf.env.Append(CPPDEFINES=['HAVE_ALTIVEC_H=1'])

        if not have_altivec and not have_altivec_h:
            print ('Missing Altivec, contuning without it...')
        else:
            print ('Configured for Altivec')

    def ConfigureAssertions(conf):
        if(conf.env['ASSERTIONS'] == "auto"):
            print('Using optimization settings to determine the assertion level')
        elif(conf.env['ASSERTIONS'] == "disabled"):
            conf.env.Append(CPPDEFINES=['SDL_DEFAULT_ASSERT_LEVEL=0'])
        elif(conf.env['ASSERTIONS'] == "release"):
            conf.env.Append(CPPDEFINES=['SDL_DEFAULT_ASSERT_LEVEL=1'])
        elif(conf.env['ASSERTIONS'] == "enabled"):
            conf.env.Append(CPPDEFINES=['SDL_DEFAULT_ASSERT_LEVEL=2'])
        elif(conf.env['ASSERTIONS'] == "paranoid"):
            conf.env.Append(CPPDEFINES=['SDL_DEFAULT_ASSERT_LEVEL=3'])
        else:
            print("unknown assertion level")
        conf.env.Append(CPPDEFINES=['HAVE_ASSERTIONS='+conf.env['ASSERTIONS']])

    def ConfigureClockGetTime(conf):
        result = conf.CheckLib('rt', 'clock_gettime')
        if(result):
            conf.env.Append(LIBS=["rt"])
            conf.env.Append(CPPDEFINES=['HAVE_CLOCK_GETTIME=1'])
            print('Configured to use clock_gettime...')
            return
        else:
            result = conf.CheckLib('c', 'clock_gettime')
            if(result):
                conf.env.Append(CPPDEFINES=['HAVE_CLOCK_GETTIME=1'])
                print('Configured to use clock_gettime...')
                return
        print('Missing clock_gettime, continuing without it...')

    def ConfigureSDLVersion(conf):
        SDL_MAJOR_VERSION=2
        SDL_MINOR_VERSION=0
        SDL_MICRO_VERSION=5
        SDL_INTERFACE_AGE=1
        SDL_BINARY_AGE=5

        LT_CURRENT=SDL_MICRO_VERSION-SDL_INTERFACE_AGE
        LT_AGE=SDL_BINARY_AGE-SDL_INTERFACE_AGE
        LT_MAJOR=LT_CURRENT-LT_AGE

        LT_REVISION = str(SDL_INTERFACE_AGE)
        LT_RELEASE = str(SDL_MAJOR_VERSION)+'.'+str(SDL_MINOR_VERSION)
        LT_VERSION = str(LT_MAJOR)+'.'+str(LT_AGE)+'.'+str(SDL_INTERFACE_AGE)

        SDL_VERSION = str(SDL_MAJOR_VERSION)+'.'+str(SDL_MINOR_VERSION)+'.'+str(SDL_MICRO_VERSION)

        conf.env.Append(CPPDEFINES=[
            'SDL_MAJOR_VERSION='+str(SDL_MAJOR_VERSION),
            'SDL_MINOR_VERSION='+str(SDL_MINOR_VERSION),
            'SDL_MICRO_VERSION='+str(SDL_MICRO_VERSION),
            'SDL_INTERFACE_AGE='+str(SDL_INTERFACE_AGE),
            'SDL_BINARY_AGE='+str(SDL_BINARY_AGE),
            'LT_REVISION='+LT_REVISION,
            'LT_RELEASE='+LT_RELEASE,
            'LT_VERSION='+LT_VERSION,
            'SDL_VERSION='+SDL_VERSION
        ])

        print ("Building SDL version " + SDL_VERSION + "\nlibtool version: "
                + LT_VERSION + " :: "
                + str(LT_AGE) + " :: "
                + LT_REVISION + " :: "
                + str(LT_CURRENT) + " :: "
                + LT_RELEASE)

    def ConfigureArch(conf):

        if(conf.env['TARGET_ARCH'] == 'x86'):
            print( 'Building for x86 target' )
            conf.env.Append(CPPDEFINES=[
                'ARCH_64=0',
                'PROCESSOR_ARCH=x86',
            ])
        elif(conf.env['TARGET_ARCH'] == 'x86_64'):
            print( 'Building for x86_64 target' )
            conf.env.Append(CPPDEFINES=[
                'ARCH_64=1',
                'PROCESSOR_ARCH=x64',
            ])

    def ConfigureSubSystems(conf):

        for sub in subsystems:
            conf.env.Append(CPPDEFINES=[
                'SDL_'  + sub.upper() + '_ENABLED_BY_DEFAULT=1'
            ])
            conf.env[sub.upper()] = True

    def ConfigureMMX(conf):

        if('MSVC_VERSION' in env and float(env['MSVC_VERSION']) > 15.0):
            if('ARCH_64=0' in conf.env['CPPDEFINES']):
                conf.env.Append(CPPDEFINES=[
                    'HAVE_MMX=1',
                ])
                print ('32 bit MSVC > 15.0 found, enabling MMX...')

    def ConfigureSSE(conf):

        if('MSVC_VERSION' in env and float(env['MSVC_VERSION']) > 15.0):
            conf.env.Append(CPPDEFINES=[
                'HAVE_SSE=1',
                'HAVE_SSE=1',
                'HAVE_SSE=1',
                'SDL_ASSEMBLY_ROUTINES=1',
            ])
            print ('MSVC > 15.0 found, enabling SSE...')

    def ConfigureLIBC(conf):
        if(env['LIBC']):
            print('Using libc...')
        else:
            if('WINDOWS' in conf.env['CPPDEFINES']):
                conf.env.Append(CPPDEFINES=[
                    'HAVE_STDARG_H=1',
                    'HAVE_STDDEF_H=1',
                ])

    conf = Configure(env, custom_tests = dict([
        ('Check_3DNow', Check_3DNow),
        ('Check_Altivec', Check_Altivec),
        ('Check_Altivec_H', Check_Altivec_H),
        ('Check_ARTS', Check_ARTS),
    ]))

    conf.env.Append(CPPDEFINES=[
        'LIBNAME=SDL2',
        'LIBTYPE=SHARED',
    ])


    if platform == "linux" or platform == "linux2":

        conf.env.Append(CPPDEFINES=['LINUX'])

        ConfigureSDLVersion(conf)
        ConfigureArch(conf)

        ConfigureDLOPEN(conf)
        ConfigureAssertions(conf)

        if(env['ASSEMBLY']):
            Configure3DNow(conf)
            ConfigureAltivec(conf)

        if(env['CLOCK_GETTIME']):
            ConfigureClockGetTime(conf)

        ConfigureALSA(conf)
        ConfigureARTS(conf)

    elif platform == "darwin":
        print("XCode project not implemented yet")
    elif platform == "win32":

        conf.env.Append(CPPDEFINES=[
            'WIN32',
            'WINDOWS=1',
            'UNIX_SYS=OFF',
            'UNIX_OR_MAC_SYS=OFF',
            'SDL_PTHREADS_ENABLED_BY_DEFAULT=OFF',
            'OPT_DEF_SSEMATH=ON',
            'OPT_DEF_LIBC=ON',
            'USE_GCC=0',
        ])

        ConfigureSDLVersion(conf)

        if('MSVC_VERSION' in env):
            print('Using MSVC version ' + env['MSVC_VERSION'])
            if(float(env['MSVC_VERSION']) < 14.0):
                print('MSVC Version is too low (<14), turning off assenbly routines')
                env['ASSEMBLY'] == False

        ConfigureArch(conf)
        ConfigureAssertions(conf)

        if(env['ASSEMBLY'] == True):
            Configure3DNow(conf)
            ConfigureAltivec(conf)
            ConfigureMMX(conf)
            ConfigureSSE(conf)

        degugDefine = 'NDEBUG',
        debugFlag = "/O2"
        degug = '/DEBUG:NONE',
        msvcRuntime = "/MD",
        libType = "Release"
        rtcFlag = ''
        if(conf.env['FORCE_STATIC_VCRT']):
            msvcRuntime = "/MT"

        if(conf.env['DEBUG_BUILD']):
            degugDefine = 'DEBUG',
            debugFlag = "/Od"
            degug = '/DEBUG:FULL',
            msvcRuntime = "/MDd",
            rtcFlag = '/RTC1'
            if(conf.env['FORCE_STATIC_VCRT']):
                msvcRuntime = "/MTd"
                rtcFlag = ''
            libType = 'Debug'




        conf.env.Append(CPPDEFINES=[
            "WIN32",
            degugDefine,
        ])

        conf.env.Append(CCFLAGS= [
            "/analyze-",
            "/GS",
            "/Zc:wchar_t",
            "/W1",
            "/Z7",
            "/Gm-",
            debugFlag,
            "/WX-",
            '/FC',
            "/Zc:forScope",             # Force Conformance in for Loop Scope
            "/GR",                      # Enable Run-Time Type Information
            "/Oy-",                     # Disable Frame-Pointer Omission
            msvcRuntime,               # Use Multithread DLL version of the runt-time library
            "/EHsc",
            "/nologo",
            rtcFlag,
        ])

        conf.env.Append(LINKFLAGS=[
            degug
        ])

    return conf.Finish()

def ConfigPlatformIDE(env, sourceFiles, headerFiles, program):
    if platform == "linux" or platform == "linux2":
        print("Eclipse C++ project not implemented yet")
    elif platform == "darwin":
        print("XCode project not implemented yet")
    elif platform == "win32":

        variantSourceFiles = []
        for file in sourceFiles:
            variantSourceFiles.append(re.sub("^build", "../src", file))
        variantHeaderFiles = []
        for file in headerFiles:
                variantHeaderFiles.append(re.sub("^src", "../src", file))
        buildSettings = {
            'LocalDebuggerCommand':os.path.abspath(Dir('.').abspath).replace('\\', '/') + "/build/EveOrePrices.exe"
        }
        buildVariant = 'Debug|Win32'
        if(env['DEBUG_BUILD']):
            buildVariant = 'Release|Win32'
        env.MSVSProject(target = 'VisualStudio/EveOrePricesApp' + env['MSVSPROJECTSUFFIX'],
                    srcs = variantSourceFiles,
                    localincs = variantHeaderFiles,
                    buildtarget = program[0],
                    DebugSettings = buildSettings,
                    variant = buildVariant)
    return env

def CreateOptions():

    AddOption(
        '--debug_build',
        dest='debug_build',
        action='store_true',
        metavar='DIR',
        default=False,
        help='Build in debug mode'
    )

    AddOption(
        '--arch',
        dest='arch',
        action='store',
        metavar='DIR',
        default='x86',
        help='platform arch type [x86|x86_64|arm|arm64]'
    )

    AddOption(
        '--alsa-shared',
        dest='alsa-shared',
        action='store_true',
        metavar='DIR',
        default=False,
        help='Dynamic load ALSA lib'
    )

    AddOption(
        '--arts-shared',
        dest='arts-shared',
        action='store_true',
        metavar='DIR',
        default=False,
        help='Dynamic load ARTS lib'
    )

    AddOption(
        '--disable-assembly',
        dest='disable-asm',
        action='store_false',
        metavar='DIR',
        default=True,
        help='Turn off assembly routines'
    )

    AddOption(
        '--clock_gettime',
        dest='clock_gettime',
        action='store_true',
        metavar='DIR',
        default=False,
        help='Use clock_gettime() instead of gettimeofday()'
    )

    AddOption(
        '--force_static_vcrt',
        dest='force_static_vcrt',
        action='store_true',
        metavar='DIR',
        default=False,
        help="Force /MT for static VC runtimes"
    )

    AddOption(
        '--assertion-level',
        dest='assertions',
        action='store',
        metavar='DIR',
        default='auto',
        help='Enable internal sanity checks (auto/disabled/release/enabled/paranoid)'
    )

    for sub in subsystems:
        AddOption(
            '--disable-' + sub,
            dest='disable-' + sub,
            action='store_false',
            metavar='DIR',
            default='True',
            help='Enable the ' + sub + ' subsystem'
        )

CreateNewEnv()
