from __future__ import print_function
import os
import sys
import json

def eprint(*args, **kwargs):
    print(*args, file=sys.stderr, **kwargs)

if sys.platform != 'darwin':
    eprint('ERROR: This is only for darwin (macOS)')
    exit(1)

# Check required homes are defined and exist
errors = False
for v in 'CBANG FAH_CLIENT_BASTET FAH_CLIENT_OSX_UNINSTALLER'.split():
    key = v + '_HOME'
    home = os.environ.get(key)
    if not home:
        errors = True
        eprint("ERROR: %s is not set or is empty string" % key)
    elif not os.path.exists(home):
        errors = True
        eprint("ERROR: %s '%s' does not exist" % (key, home))
if errors:
    exit(1)

env = Environment(ENV = os.environ)

try:
    env.Tool('config', toolpath = [os.environ.get('CBANG_HOME')])
except Exception as e:
    raise Exception('CBANG_HOME not set?\n' + str(e))

env.CBLoadTools('packager')
conf = env.CBConfigure()

client_home = os.environ.get('FAH_CLIENT_BASTET_HOME')
package_info = {}

# Version
version = None
path = client_home + '/package.json'
try:
    with open(path, 'r') as f:
        package_info = json.load(f)
    version = package_info['version'].encode('ascii')
except Exception as e:
    eprint(e)
if not version:
    eprint('ERROR: Unable to get version from ' + path)
    if not env.GetOption('clean'):
        exit(1)
    # let clean proceed
    version = '0.0.0'

# Config vars; lifted from fah-client-bastet
env.Replace(PACKAGE_VERSION   = version)
env.Replace(PACKAGE_AUTHOR = 'Joseph Coffland <joseph@cauldrondevelopment.com>')
env.Replace(PACKAGE_COPYRIGHT = '2022 foldingathome.org')
env.Replace(PACKAGE_HOMEPAGE  = 'https://foldingathome.org/')
env.Replace(PACKAGE_ORG       = 'foldingathome.org')

env.Append(PACKAGE_IGNORES = ['.DS_Store'])

# Set up Uninstaller component that installs the actual uninstaller pkg
un_home = os.environ.get('FAH_CLIENT_OSX_UNINSTALLER_HOME')
un_root = './dist/flatpkg/Uninstaller/root' # note: relative to PWD
un_pkg_name_file = os.path.join(un_home, 'package.txt')
if not env.GetOption('clean'):
    # un_pkg_name_file may not exist during a clean
    un_pkg_name = open(un_pkg_name_file).read().strip()
else:
    un_pkg_name = ''
# replace assumed '.mpkg.zip' with '.pkg'; it should exist
un_pkg_name = os.path.splitext(os.path.splitext(un_pkg_name)[0])[0] + '.pkg'
un_pkg_files = [[os.path.join(un_home, un_pkg_name),
            'Applications/Folding@home/Uninstall Folding@home.pkg']]

if not env.GetOption('clean'):
    # Create pkg root for our uninstaller component
    import shutil
    if os.path.exists(un_root):
        shutil.rmtree(un_root)
    env.CopyToPackage(un_pkg_files, un_root)

# Specify components for the distribution pkg
distpkg_components = [
    {
        # name is component pkg file name and name shown to user in installer
        'name'        : 'FAHClient',
        'pkg_id'      : 'org.foldingathome.fahclient.pkg',
        # absolute path to repo directory
        'home'        : client_home,
        # relative to home
        'pkg_scripts' : 'install/osx/scripts',
        # abs path or relative to PWD
        # cbang config pkg module uses build/pkg/root
        'root'        : client_home + '/build/pkg/root',
        # relative to root
        'sign_tools'  : ['usr/local/bin/fah-client'],
    },
    {
        'name'        : 'Uninstaller',
        'pkg_id'      : 'org.foldingathome.uninstaller.pkg',
        'home'        : un_home,
        'root'        : un_root,
        # no scripts, nothing to sign
        # bundled uninstaller pkg presumed already signed and notarized
    },
]

# Package
name = 'fah-installer'
parameters = {
    'name'               : name,
    'version'            : version,
    'maintainer'         : env['PACKAGE_AUTHOR'],
    'vendor'             : env['PACKAGE_ORG'],
    'summary'            : 'Folding@home ' + version,
    'description'        : 'Folding@home ' + version + ' Installer Package',

    'url'                : env['PACKAGE_HOMEPAGE'],
    'bug_url'            : 'https://github.com/FoldingAtHome/fah-issues',

    'pkg_type'           : 'dist',
    'distpkg_resources'  : [['install/osx/Resources', '.']],
    'distpkg_welcome'    : 'Welcome.rtf',
    'distpkg_license'    : 'License.rtf',
    'distpkg_background' : 'fah-opacity-50.png',
    'distpkg_customize'  : 'always',
    'distpkg_target'     : env.get('osx_min_ver', '10.13'),
    'distpkg_arch'       : env.get('package_arch', 'x86_64'),
    'distpkg_components' : distpkg_components,
    }
pkg = env.Packager(**parameters)

AlwaysBuild(pkg)
env.Alias('package', pkg)

Clean(pkg, ['build', 'dist', 'config.log'])
# ensure *.zip not cleaned unless distclean
NoClean(pkg, [Glob('*.zip'), 'package.txt'])
if 'distclean' in COMMAND_LINE_TARGETS:
    Clean('distclean', [
        '.sconsign.dblite', '.sconf_temp', 'config.log',
        'build', 'dist', 'package.txt', 'package-description.txt',
        Glob(name + '*.pkg'),
        Glob(name + '*.mpkg'),
        Glob(name + '*.zip'),
        ])
