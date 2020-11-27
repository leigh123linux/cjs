### Jasmine tests ##############################################################

jsunit_resources_files = gnome.compile_resources('jsunit-resources',
    'jsunit.gresources.xml', c_name: 'jsunit_resources')

minijasmine = executable('minijasmine', '../minijasmine.cpp',
    jsunit_resources_files, dependencies: libgjs_dep,
    cpp_args: [
        '-DINSTTESTDIR="@0@"'.format(installed_tests_execdir),
    ],
    include_directories: top_include,
    install: get_option('installed_tests'), install_dir: installed_tests_execdir)

gidatadir = gi.get_pkgconfig_variable('gidatadir')
gi_tests = gidatadir / 'tests'

test_gir_extra_c_args = []
test_gir_warning_c_args = []

if cc.get_argument_syntax() == 'msvc'
  # We need to ensure the symbols in the test DLLs export in clang-cl builds
  test_gir_extra_c_args += ['-D_GI_EXTERN=__declspec(dllexport)extern']
else
  # These consist of external code (from gobject-introspection) so they should not
  # error out even when building with -Werror
  test_gir_warning_c_args += ['-Wno-error']
endif

regress_dependencies = [glib, gobject, gio]
regress_gir_includes = ['Gio-2.0']
regress_gir_c_args = test_gir_extra_c_args
if build_cairo
    regress_gir_includes += 'cairo-1.0'
    regress_dependencies += [cairo, cairo_gobject]
else
    regress_gir_c_args += ['-D_GI_DISABLE_CAIRO']
endif

regress_sources = [
    gi_tests / 'regress.c',
    gi_tests / 'regress.h',
]
libregress = library('regress', regress_sources,
    c_args: regress_gir_c_args + test_gir_warning_c_args,
    dependencies: regress_dependencies, install: get_option('installed_tests'),
    install_dir: installed_tests_execdir)
regress_gir = gnome.generate_gir(libregress, includes: regress_gir_includes,
    sources: regress_sources, namespace: 'Regress', nsversion: '1.0',
    identifier_prefix: 'Regress', symbol_prefix: 'regress_',
    extra_args: ['--warn-all', '--warn-error'] + regress_gir_c_args,
    install: get_option('installed_tests'), install_dir_gir: false,
    install_dir_typelib: installed_tests_execdir)
regress_typelib = regress_gir[1]

warnlib_sources = [
    gi_tests / 'warnlib.c',
    gi_tests / 'warnlib.h',
]
libwarnlib = library('warnlib', warnlib_sources,
    c_args: test_gir_warning_c_args + test_gir_extra_c_args,
    dependencies: [glib, gobject, gio], install: get_option('installed_tests'),
    include_directories: top_include,
    install_dir: installed_tests_execdir)
# This should have --warn-all turned off, but there is currently no way to do so
# in gnome.generate_gir(). See https://github.com/mesonbuild/meson/issues/5876
warnlib_gir = gnome.generate_gir(libwarnlib, includes: ['Gio-2.0'],
    sources: warnlib_sources, namespace: 'WarnLib', nsversion: '1.0',
    symbol_prefix: 'warnlib_', header: 'warnlib.h',
    install: get_option('installed_tests'), install_dir_gir: false,
    install_dir_typelib: installed_tests_execdir)
warnlib_typelib = warnlib_gir[1]

gimarshallingtests_sources = [
    gi_tests / 'gimarshallingtests.c',
    gi_tests / 'gimarshallingtests.h',
]
libgimarshallingtests = library('gimarshallingtests',
    gimarshallingtests_sources, dependencies: [glib, gobject, gio],
    c_args:  test_gir_extra_c_args + test_gir_warning_c_args,
    install: get_option('installed_tests'), install_dir: installed_tests_execdir)
gimarshallingtests_gir = gnome.generate_gir(libgimarshallingtests,
    includes: ['Gio-2.0'], sources: gimarshallingtests_sources,
    namespace: 'GIMarshallingTests', nsversion: '1.0',
    symbol_prefix: 'gi_marshalling_tests_', extra_args: '--warn-error',
    install: get_option('installed_tests'), install_dir_gir: false,
    install_dir_typelib: installed_tests_execdir)
gimarshallingtests_typelib = gimarshallingtests_gir[1]

jasmine_tests = [
    'self',
    'ByteArray',
    'Exceptions',
    'Format',
    'Fundamental',
    'Gettext',
    'GIMarshalling',
    'Gio',
    'GLib',
    'GObject',
    'GObjectClass',
    'GObjectInterface',
    'GTypeClass',
    'Importer',
    'Introspection',
    'Lang',
    'LegacyByteArray',
    'LegacyClass',
    'LegacyGObject',
    'Mainloop',
    'Namespace',
    'Package',
    'ParamSpec',
    'Print',
    'Regress',
    'Signals',
    'System',
    'Tweener',
    'WarnLib',
]

if build_cairo
    jasmine_tests += 'Cairo'
endif

if not get_option('skip_gtk_tests')
    jasmine_tests += [
        'Gtk3',
        'GObjectDestructionAccess',
        'LegacyGtk',
    ]

    if have_gtk4
        jasmine_tests += 'Gtk4'
    endif
endif

installed_js_tests_dir = installed_tests_execdir / 'js'

gschemas_compiled = gnome.compile_schemas(
    depend_files: 'org.cinnamon.CjsTest.gschema.xml')

foreach test : jasmine_tests
    test_file = files('test@0@.js'.format(test))

    test(test, minijasmine, args: test_file,
        depends: [
            gschemas_compiled,
            gimarshallingtests_typelib,
            regress_typelib,
            warnlib_typelib,
        ],
        env: tests_environment, protocol: 'tap', suite: 'JS')

    test_description_subst = {
        'name': 'test@0@.js'.format(test),
        'installed_tests_execdir': installed_tests_execdir,
    }
    test_description = configure_file(configuration: test_description_subst,
        input: '../minijasmine.test.in',
        output: 'test@0@.test'.format(test),
        install_dir: installed_tests_metadir)

    if get_option('installed_tests')
        install_data(test_file, install_dir: installed_js_tests_dir)
    endif
endforeach

# testGDBus.js is separate, because it can be skipped, and during build should
# be run using dbus-run-session

if not get_option('skip_dbus_tests')
    test_file = files('testGDBus.js')
    bus_config = files('../../test/test-bus.conf')
    test('GDBus', dbus_run_session,
        args: ['--config-file', bus_config, '--', minijasmine, test_file],
        env: tests_environment, protocol: 'tap', suite: 'dbus')
endif

gdbus_test_description_subst = {
    'name': 'testGDBus.js',
    'installed_tests_execdir': installed_tests_execdir,
}
gdbus_test_description = configure_file(
    configuration: gdbus_test_description_subst,
    input: '../minijasmine.test.in', output: 'testGDBus.test',
    install_dir: installed_tests_metadir)

if get_option('installed_tests')
    install_data('testGDBus.js', install_dir: installed_js_tests_dir)
endif