#
# Copyright 2010-2013 Fabric Engine Inc. All rights reserved.
#

import os, sys, platform, copy

Import(
  'parentEnv',
  'FABRIC_DIR',
  'FABRIC_SPLICE_VERSION',
  'STAGE_DIR',
  'FABRIC_BUILD_TYPE',
  'FABRIC_BUILD_OS',
  'FABRIC_BUILD_ARCH',
  'BOOST_DIR'
  )

# configuration flags
if FABRIC_BUILD_OS == "Windows":
  baseCPPDefines = [
    '_SCL_SECURE_NO_WARNINGS=1',
    '_ITERATOR_DEBUG_LEVEL=0',
    '_WIN32_WINNT=0x0600',
  ]
  baseCPPFlags = [
    '/EHsc', 
    '/wd4624',
  ]
  baseLinkFlags = ['chkstk.obj']
  spliceDebugFlags = {
    'CCFLAGS': baseCPPFlags + ['/Od', '/Z7', '/MTd', '/DEBUG'],
    'CPPDEFINES': baseCPPDefines + ['_ITERATOR_DEBUG_LEVEL=0', '_DEBUG', 'DEBUG'],
    'LINKFLAGS': baseLinkFlags + ['/DEBUG', '/NODEFAULTLIB:LIBCMT'],
  }
  spliceReleaseFlags = {
    'CCFLAGS': baseCPPFlags + ['/Ox', '/MT'],
    'CPPDEFINES': baseCPPDefines + ['NDEBUG'],
    'LINKFLAGS': baseLinkFlags + ['/NDEBUG', '/NODEFAULTLIB:LIBCMTD'],
  }
  if FABRIC_BUILD_ARCH.endswith('64'):
    baseCPPDefines.append( 'WIN64' )
else:
  spliceDebugFlags = {
    'CCFLAGS': ['-fPIC', '-m64', '-g'],
    'LINKFLAGS': ['-m64']
  }
  spliceReleaseFlags = {
    'CCFLAGS': ['-fPIC', '-m64', '-O2'],
    'LINKFLAGS': ['-m64'],
    'CPPDEFINES': ['NDEBUG'],
  }

if FABRIC_BUILD_OS == "Darwin":
  for flags in [spliceDebugFlags, spliceReleaseFlags]:
    flags['CCFLAGS'] += [
      '-fvisibility=hidden',
      '-mmacosx-version-min=10.7',
      '-stdlib=libstdc++',
      '-fno-omit-frame-pointer',
      ]      
    flags['LINKFLAGS'] += [
      '-mmacosx-version-min=10.7',
      '-stdlib=libstdc++',
      ]

Export('spliceDebugFlags', 'spliceReleaseFlags')

if FABRIC_BUILD_TYPE == 'Debug':
  parentEnv.MergeFlags(spliceDebugFlags)
else:
  parentEnv.MergeFlags(spliceReleaseFlags)

baseCapiFlags = {
  'CPPPATH': [os.path.join(FABRIC_DIR, 'include')],
  'CPPDEFINES': [
    'FEC_PROVIDE_STL_BINDINGS',
    '__STDC_LIMIT_MACROS',
    '__STDC_CONSTANT_MACROS',
  ],
  'LIBPATH': [],
  'LIBS': [],
}

FABRIC_CORE_VERSION = FABRIC_SPLICE_VERSION.rpartition('.')[0]

staticCapiFlags = copy.deepcopy(baseCapiFlags)
staticCapiFlags['CPPDEFINES'] += ['FEC_STATIC']
if FABRIC_BUILD_OS == 'Windows':
  staticCapiFlags['LIBS'] += [File(os.path.join(FABRIC_DIR, 'lib', 'FabricCore-'+FABRIC_CORE_VERSION+'_s.lib'))]
else:
  staticCapiFlags['LIBS'] += [File(os.path.join(FABRIC_DIR, 'lib', 'libFabricCore-'+FABRIC_CORE_VERSION+'_s.a'))]
Export('staticCapiFlags')

if FABRIC_BUILD_OS == 'Windows':
  fabricCoreLibName = 'FabricCore-'+FABRIC_CORE_VERSION
else:
  fabricCoreLibName = 'FabricCore'

sharedCapiFlags = copy.deepcopy(baseCapiFlags)
sharedCapiFlags['CPPDEFINES'] += ['FEC_SHARED']
sharedCapiFlags['LIBPATH'] += [os.path.join(FABRIC_DIR, 'lib')]
sharedCapiFlags['LIBS'] += [fabricCoreLibName]
Export('sharedCapiFlags')

apiVersion = FABRIC_SPLICE_VERSION.split('-')[0].split('.')
for i in range(1, len(apiVersion)):
  while len(apiVersion[i]) < 3:
    apiVersion[i] = '0'+apiVersion[i]

parentEnv.Append(CPPDEFINES=['SPLICE_API_VERSION='+''.join(apiVersion)])
parentEnv.Append(CPPDEFINES=['SPLICE_MAJOR_VERSION='+FABRIC_SPLICE_VERSION.split('.')[0]])
parentEnv.Append(CPPDEFINES=['SPLICE_MINOR_VERSION='+FABRIC_SPLICE_VERSION.split('.')[1]])
parentEnv.Append(CPPDEFINES=['SPLICE_REVISION_VERSION='+FABRIC_SPLICE_VERSION.split('.')[2]])

if FABRIC_BUILD_OS == 'Windows':
  parentEnv.Append(LIBS = ['advapi32', 'shell32'])

if BOOST_DIR is None:
  print "BOOST_DIR not found. Please specify BOOST_DIR."
  print "Refer to README.txt for more information."
  sys.exit(1)

if not os.path.exists(BOOST_DIR):
  print "BOOST_DIR path invalid: " + BOOST_DIR
  print "Refer to README.txt for more information."
  sys.exit(1)

boostFlags = {
  'CPPPATH': [BOOST_DIR],
  'LIBPATH': [os.path.join(BOOST_DIR, 'lib')],
}
if FABRIC_BUILD_OS == 'Windows':
  if FABRIC_BUILD_TYPE == 'Debug':
    boostFlags['LIBS'] = [
      'libboost_thread-vc100-mt-sgd-1_55.lib',
      'libboost_system-vc100-mt-sgd-1_55.lib',
      'libboost_filesystem-vc100-mt-sgd-1_55.lib'
      ]
  else:
    boostFlags['LIBS'] = [
      'libboost_thread-vc100-mt-s-1_55.lib',
      'libboost_system-vc100-mt-s-1_55.lib',
      'libboost_filesystem-vc100-mt-s-1_55.lib'
      ]
else:
  boostFlags['LIBS'] = ['boost_thread','boost_system','boost_filesystem']
Export('boostFlags')

parentEnv.MergeFlags(boostFlags)

env = parentEnv.Clone()

libNameBase = 'FabricSplice-'+FABRIC_SPLICE_VERSION

staticEnv = env.Clone()
staticEnv.Append(CPPDEFINES=['FEC_SHARED', 'FECS_STATIC', 'FECS_BUILDING'])
staticEnv.MergeFlags(sharedCapiFlags)
staticLibName = libNameBase+'_s'
if FABRIC_BUILD_OS == 'Windows':
  staticLibName += '.lib'
else:
  staticLibName += '.a'

staticLibrary = staticEnv.Library(
  staticLibName, 
  staticEnv.Glob('*.cpp')
)
installedStaticLibrary = staticEnv.Install(STAGE_DIR.Dir('lib'), staticLibrary)

spliceDistHeader = staticEnv.Install(STAGE_DIR.Dir('include'), 'FabricSplice.h')
Export('spliceDistHeader')

installedFiles = [installedStaticLibrary, spliceDistHeader]

spliceFlags = {
  'CPPPATH': [STAGE_DIR],
  'LIBS': [installedStaticLibrary],
  'CPPDEFINES': ['_BOOL', 'FECS_STATIC']
}
Export('spliceFlags')

alias = env.Alias('spliceapi', installedFiles)
spliceData = (alias, installedFiles)
Return('spliceData')
