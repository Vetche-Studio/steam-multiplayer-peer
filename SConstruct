#!/usr/bin/env python
import os
import sys

from methods import print_error


libname = "steam-multiplayer-peer"
projectdir = "steam-multiplayer-peer"

localEnv = Environment(tools=["default"], PLATFORM="")

# Build profiles can be used to decrease compile times.
# You can either specify "disabled_classes", OR
# explicitly specify "enabled_classes" which disables all other classes.
# Modify the example file as needed and uncomment the line below or
# manually specify the build_profile parameter when running SCons.

# localEnv["build_profile"] = "build_profile.json"

customs = ["custom.py"]
customs = [os.path.abspath(path) for path in customs]

opts = Variables(customs, ARGUMENTS)
opts.Update(localEnv)

Help(opts.GenerateHelpText(localEnv))

env = localEnv.Clone()

submodule_initialized = False
dir_name = 'godot-cpp'
if os.path.isdir(dir_name):
    if os.listdir(dir_name):
        submodule_initialized = True

if not submodule_initialized:
    print_error("""godot-cpp is not available within this folder, as Git submodules haven't been initialized.
Run the following command to download godot-cpp:
    git submodule update --init --recursive""")
    sys.exit(1)

# Local dependency paths, adapt them to your setup
steam_lib_path = "steam-multiplayer-peer/steamworks-sdk/redistributable_bin"

env = SConscript("godot-cpp/SConstruct", {"env": env, "customs": customs})
if env['platform'] in ('macos', 'osx'):
    # Set the correct Steam library
    steam_lib_path += "/osx"
    steamworks_library = 'libsteam_api.dylib'

elif env['platform'] in ('linuxbsd', 'linux'):
    # Set correct Steam library
    steam_lib_path += "/linux64" if env['arch'] == 'x86_64' else "/linux32"
    steamworks_library = 'libsteam_api.so'

elif env['platform'] == "windows":
    steam_lib_path += "/win64" if env['arch'] == 'x86_64' else ""
    steamworks_library = 'steam_api64.dll' if env['arch'] == 'x86_64' else 'steam_api.dll'

env.Append(LIBPATH=[steam_lib_path])
env.Append(CPPPATH=['steam-multiplayer-peer/steamworks-sdk/public'])
env.Append(LIBS=[
    steamworks_library.replace(".dll", "")
])

env.Append(CPPPATH=['steam-multiplayer-peer/'])
sources = [
    Glob('steam-multiplayer-peer/*.cpp'),
    ]

if env["target"] in ["editor", "template_debug"]:
    try:
        doc_data = env.GodotCPPDocData("src/gen/doc_data.gen.cpp", source=Glob("doc_classes/*.xml"))
        sources.append(doc_data)
    except AttributeError:
        print("Not including class reference as we're targeting a pre-4.3 baseline.")

file = "{}{}{}".format(libname, env["suffix"], env["SHLIBSUFFIX"])

libraryfile = "bin/{}/{}".format(env["platform"], file)
library = env.SharedLibrary(
    libraryfile,
    source=sources,
)

copy = env.InstallAs("{}/bin/{}/lib{}".format(projectdir, env["platform"], file), library)

default_args = [library, copy]
Default(*default_args)
