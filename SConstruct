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


def CreateNewEnv():

    AddOption(
        '--debug_build',
        dest='debug_build',
        action='store_true',
        metavar='DIR',
        default=False,
        help='Build in debug mode'
    )

    env = Environment(DEBUG_BUILD = GetOption('debug_build'),TARGET_ARCH='x86');

    baseProjectDir = os.path.abspath(Dir('.').abspath).replace('\\', '/')
    print("BASE_DIR="+baseProjectDir)
    env.VariantDir('build', 'src', duplicate=0)

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
    env = ConfigPlatformEnv(env, baseProjectDir)
    env = ConfigPlatformIDE(env, sourceFiles, headerFiles, prog)

    return env

def ConfigPlatformEnv(env, baseProjectDir):

    if platform == "linux" or platform == "linux2":
        print("Eclipse C++ project not implemented yet")
    elif platform == "darwin":
        print("XCode project not implemented yet")
    elif platform == "win32":

        degugDefine = 'NDEBUG',
        debugFlag = "/O2"
        degug = '/DEBUG:NONE',
        debugRuntime = "/MD",
        libType = "Release"
        if(env['DEBUG_BUILD']):
            degugDefine = 'DEBUG',
            debugFlag = "/Od"
            degug = '/DEBUG:FULL',
            debugRuntime = "/MDd",
            libType = 'Debug'

        env.Append(CPPDEFINES=[
            "WIN32",
            degugDefine,
        ])

        env.Append(CCFLAGS= [
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
            debugRuntime,               # Use Multithread DLL version of the runt-time library
            "/EHsc",
            "/nologo",
        ])

        env.Append(LINKFLAGS=[
            degug
        ])

    return env

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


CreateNewEnv()
