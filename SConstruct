import os.path
esp_env = Environment(tools=['default','textfile'], CPPPATH=['#/src'])
esp_env['STATIC_AND_SHARED_OBJECTS_ARE_THE_SAME']=1

def filtered_glob(env, pattern, omit=[], ondisk=True, source=False, strings=False):
    return filter(lambda f: os.path.basename(f.path) not in omit, env.Glob(pattern))

def prefix_with(prefix):
    return lambda x: prefix + x

esp_env.AddMethod(filtered_glob, "FilteredGlob")

build_platform = None
build_64 = (ARGUMENTS.get('build_64', 'false') == 'true')
build_debug = (ARGUMENTS.get('debug', 'false') == 'true')
build_tests = (ARGUMENTS.get('tests', 'false') == 'true')

if esp_env['PLATFORM'] == 'posix':
    esp_env.Replace(CXX = "g++-4.8")
    esp_env.Replace(CC = "gcc-4.8")
    build_platform = 'posix'
elif esp_env['PLATFORM'] == 'win32':
    build_platform = 'win'
elif esp_env['PLATFORM'] == 'darwin':
    esp_env.Replace(CXX = "clang")
    build_64 = True
    build_platform = 'mac'

esp_libs = []

if build_platform == 'posix':
    esp_env.AppendUnique(CPPDEFINES = ['ESP_LINUX'])
    esp_env.AppendUnique(CCFLAGS = Split('-fPIC'))
    esp_env.AppendUnique(CXXFLAGS = Split('-std=c++11 -pedantic -fno-exceptions -fno-rtti'))
    esp_env.AppendUnique(LINKFLAGS = Split('-rdynamic -fno-exceptions -fno-rtti'))
    esp_libs = ['rt', 'pthread']

esp_sources = esp_env.Glob("#/src/*.cpp")
esp_objects = esp_env.Object(esp_sources)

sqlite_sources = esp_env.FilteredGlob("#/src/sqlite3/*.c", ["shell.c"])
sqlite_objects = esp_env.Object(sqlite_sources)

sqlite_shell_object = esp_env.Object(["#/src/sqlite3/shell.c"])

esp_lib = esp_env.Library("esp", esp_objects + sqlite_objects, LIBS=esp_libs)
sqlite_shell = esp_env.Program("sqlite", sqlite_objects + sqlite_shell_object, LIBS=['pthread', 'dl'])

if build_tests:
    test_libs = esp_libs
    test_env = esp_env.Clone()
    
    Export(['test_env', 'test_libs', 'esp_lib'])
    test_scripts = esp_env.Glob("#tests/*.scons")
    for s in test_scripts:
        rval = SConscript(s)
        Depends(rval, esp_lib)




