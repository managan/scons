#
# SConstruct file to build scons packages during development.
#
# See the README file for an overview of how SCons is built and tested.
#

#
# Copyright (c) 2001, 2002 Steven Knight
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY
# KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
# WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
# LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
# OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
# WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

import distutils.util
import os
import os.path
import socket
import stat
import string
import sys
import time

project = 'scons'
default_version = '0.09'

Default('.')

#
# An internal "whereis" routine to figure out if a given program
# is available on this system.
#
def whereis(file):
    for dir in string.split(os.environ['PATH'], os.pathsep):
        f = os.path.join(dir, file)
        if os.path.isfile(f):
            try:
                st = os.stat(f)
            except:
                continue
            if stat.S_IMODE(st[stat.ST_MODE]) & 0111:
                return f
    return None

#
# We let the presence or absence of various utilities determine
# whether or not we bother to build certain pieces of things.
# This should allow people to still do SCons work even if they
# don't have Aegis or RPM installed, for example.
#
aegis = whereis('aegis')
aesub = whereis('aesub')
dh_builddeb = whereis('dh_builddeb')
fakeroot = whereis('fakeroot')
gzip = whereis('gzip')
rpm = whereis('rpm')
unzip = whereis('unzip')
zip = whereis('zip')

#
# Now grab the information that we "build" into the files.
#
try:
    date = ARGUMENTS['date']
except:
    date = time.strftime("%Y/%m/%d %H:%M:%S", time.localtime(time.time()))
    
if ARGUMENTS.has_key('developer'):
    developer = ARGUMENTS['developer']
elif os.environ.has_key('USERNAME'):
    developer = os.environ['USERNAME']
elif os.environ.has_key('LOGNAME'):
    developer = os.environ['LOGNAME']
elif os.environ.has_key('USER'):
    developer = os.environ['USER']

if ARGUMENTS.has_key('build_system'):
    build_system = ARGUMENTS['build_system']
else:
    build_system = string.split(socket.gethostname(), '.')[0]

if ARGUMENTS.has_key('version'):
    revision = ARGUMENTS['version']
elif aesub:
    revision = os.popen(aesub + " \\$version", "r").read()[:-1]
else:
    revision = default_version

a = string.split(revision, '.')
arr = [a[0]]
for s in a[1:]:
    if len(s) == 1:
        s = '0' + s
    arr.append(s)
revision = string.join(arr, '.')

# Here's how we'd turn the calculated $revision into our package $version.
# This makes it difficult to coordinate with other files (debian/changelog
# and rpm/scons.spec) that hard-code the version number, so just go with
# the flow for now and hard code it here, too.
#if len(arr) >= 2:
#    arr = arr[:-1]
#def xxx(str):
#    if str[0] == 'C' or str[0] == 'D':
#        str = str[1:]
#    while len(str) > 2 and str[0] == '0':
#        str = str[1:]
#    return str
#arr = map(lambda x, xxx=xxx: xxx(x), arr)
#version = string.join(arr, '.')
version = default_version

build_id = string.replace(revision, version + '.', '')

if ARGUMENTS.has_key('change'):
    change = ARGUMENTS['change']
elif aesub:
    change = os.popen(aesub + " \\$change", "r").read()[:-1]
else:
    change = default_version

python_ver = sys.version[0:3]

platform = distutils.util.get_platform()

ENV = { 'PATH' : os.environ['PATH'] }
for key in ['AEGIS_PROJECT', 'PYTHONPATH']:
    if os.environ.has_key(key):
        ENV[key] = os.environ[key]

lib_project = os.path.join("lib", project)

cwd_build = os.path.join(os.getcwd(), "build")

test_deb_dir        = os.path.join(cwd_build, "test-deb")
test_rpm_dir        = os.path.join(cwd_build, "test-rpm")
test_tar_gz_dir     = os.path.join(cwd_build, "test-tar-gz")
test_src_tar_gz_dir = os.path.join(cwd_build, "test-src-tar-gz")
test_zip_dir        = os.path.join(cwd_build, "test-zip")
test_src_zip_dir    = os.path.join(cwd_build, "test-src-zip")

unpack_tar_gz_dir   = os.path.join(cwd_build, "unpack-tar-gz")
unpack_zip_dir      = os.path.join(cwd_build, "unpack-zip")

if platform == "win32":
    tar_hflag = ''
    python_project_subinst_dir = None
    project_script_subinst_dir = 'Scripts'
else:
    tar_hflag = 'h'
    python_project_subinst_dir = lib_project
    project_script_subinst_dir = 'bin'


zcat = 'gzip -d -c'
    
#
# Figure out if we can handle .zip files.
#
zipit = None
unzipit = None
try:
    import zipfile

    def zipit(env, target, source):
        print "Zipping %s:" % str(target[0])
        def visit(arg, dirname, names):
            for name in names:
                path = os.path.join(dirname, name)
                if os.path.isfile(path):
                    arg.write(path)
        zf = zipfile.ZipFile(str(target[0]), 'w')
        os.chdir('build')
        os.path.walk(env['PSV'], visit, zf)
        os.chdir('..')
        zf.close()

    def unzipit(env, target, source):
        print "Unzipping %s:" % str(source[0])
        zf = zipfile.ZipFile(str(source[0]), 'r')
        for name in zf.namelist():
            dest = os.path.join(env['UNPACK_ZIP_DIR'], name)
            dir = os.path.dirname(dest)
            try:
                os.makedirs(dir)
            except:
                pass
            print dest,name
            # if the file exists, then delete it before writing
            # to it so that we don't end up trying to write to a symlink:
            if os.path.isfile(dest) or os.path.islink(dest):
                os.unlink(dest)
            if not os.path.isdir(dest):
                open(dest, 'w').write(zf.read(name))

except:
    if unzip and zip:
        zipit = "cd build && $ZIP $ZIPFLAGS dist/${TARGET.file} $PSV"
        unzipit = "$UNZIP $UNZIPFLAGS $SOURCES"

def SCons_revision(target, source, env):
    """Interpolate specific values from the environment into a file.
    
    This is used to copy files into a tree that gets packaged up
    into the source file package.
    """
    t = str(target[0])
    s = source[0].rstr()
    # Note:  We don't use $VERSION from the environment so that
    # this routine will change when the version number changes
    # and things will get rebuilt properly.
    global version
    print "SCons_revision() < %s > %s" % (s, t)
    inf = open(s, 'rb')
    outf = open(t, 'wb')
    for line in inf.readlines():
        # Note:  We construct the __*__ substitution strings here
        # so that they don't get replaced when this file gets
        # copied into the tree for packaging.
        line = string.replace(line, '__BUILD'     + '__', env['BUILD'])
        line = string.replace(line, '__BUILDSYS'  + '__', env['BUILDSYS'])
        line = string.replace(line, '__DATE'      + '__', env['DATE'])
        line = string.replace(line, '__DEVELOPER' + '__', env['DEVELOPER'])
        line = string.replace(line, '__FILE'      + '__', s)
        line = string.replace(line, '__REVISION'  + '__', env['REVISION'])
        line = string.replace(line, '__VERSION'   + '__',  version)
        outf.write(line)
    inf.close()
    outf.close()
    os.chmod(t, os.stat(s)[0])

revbuilder = Builder(action = SCons_revision)

env = Environment(
                   ENV                 = ENV,
 
                   BUILD               = build_id,
                   BUILDSYS            = build_system,
                   DATE                = date,
                   DEVELOPER           = developer,
                   REVISION            = revision,
                   VERSION             = version,
                   DH_COMPAT           = 2,

                   TAR_HFLAG           = tar_hflag,

                   ZIP                 = zip,
                   ZIPFLAGS            = '-r',
                   UNZIP               = unzip,
                   UNZIPFLAGS          = '-o -d $UNPACK_ZIP_DIR',

                   ZCAT                = zcat,

                   TEST_DEB_DIR        = test_deb_dir,
                   TEST_RPM_DIR        = test_rpm_dir,
                   TEST_SRC_TAR_GZ_DIR = test_src_tar_gz_dir,
                   TEST_SRC_ZIP_DIR    = test_src_zip_dir,
                   TEST_TAR_GZ_DIR     = test_tar_gz_dir,
                   TEST_ZIP_DIR        = test_zip_dir,

                   UNPACK_TAR_GZ_DIR   = unpack_tar_gz_dir,
                   UNPACK_ZIP_DIR      = unpack_zip_dir,

                   BUILDERS            = { 'SCons_revision' : revbuilder },

                   PYTHON              = sys.executable
                 )

#
# Define SCons packages.
#
# In the original, more complicated packaging scheme, we were going
# to have separate packages for:
#
#	python-scons	only the build engine
#	scons-script	only the script
#	scons		the script plus the build engine
#
# We're now only delivering a single "scons" package, but this is still
# "built" as two sub-packages (the build engine and the script), so
# the definitions remain here, even though we're not using them for
# separate packages.
#

python_scons = {
        'pkg'           : 'python-' + project,
        'src_subdir'    : 'engine',
        'inst_subdir'   : os.path.join('lib', 'python1.5', 'site-packages'),

        'debian_deps'   : [
                            'debian/changelog',
                            'debian/control',
                            'debian/copyright',
                            'debian/dirs',
                            'debian/docs',
                            'debian/postinst',
                            'debian/prerm',
                            'debian/rules',
                          ],

        'files'         : [ 'LICENSE.txt',
                            'README.txt',
                            'setup.cfg',
                            'setup.py',
                          ],

        'filemap'       : {
                            'LICENSE.txt' : '../LICENSE.txt'
                          },
}

#
# The original packaging scheme would have have required us to push
# the Python version number into the package name (python1.5-scons,
# python2.0-scons, etc.), which would have required a definition
# like the following.  Leave this here in case we ever decide to do
# this in the future, but note that this would require some modification
# to src/engine/setup.py before it would really work.
#
#python2_scons = {
#        'pkg'          : 'python2-' + project,
#        'src_subdir'   : 'engine',
#        'inst_subdir'  : os.path.join('lib', 'python2.1', 'site-packages'),
#
#        'debian_deps'  : [
#                            'debian/changelog',
#                            'debian/control',
#                            'debian/copyright',
#                            'debian/dirs',
#                            'debian/docs',
#                            'debian/postinst',
#                            'debian/prerm',
#                            'debian/rules',
#                          ],
#
#        'files'        : [
#                            'LICENSE.txt',
#                            'README.txt',
#                            'setup.cfg',
#                            'setup.py',
#                          ],
#        'filemap'      : {
#                            'LICENSE.txt' : '../LICENSE.txt',
#                          },
#}
#

scons_script = {
        'pkg'           : project + '-script',
        'src_subdir'    : 'script',
        'inst_subdir'   : 'bin',

        'debian_deps'   : [
                            'debian/changelog',
                            'debian/control',
                            'debian/copyright',
                            'debian/dirs',
                            'debian/docs',
                            'debian/postinst',
                            'debian/prerm',
                            'debian/rules',
                          ],

        'files'         : [
                            'LICENSE.txt',
                            'README.txt',
                            'setup.cfg',
                            'setup.py',
                          ],

        'filemap'       : {
                            'LICENSE.txt' : '../LICENSE.txt',
                            'scons'       : 'scons.py',
                           }
}

scons = {
        'pkg'           : project,

        'debian_deps'   : [ 
                            'debian/changelog',
                            'debian/control',
                            'debian/copyright',
                            'debian/dirs',
                            'debian/docs',
                            'debian/postinst',
                            'debian/prerm',
                            'debian/rules',
                          ],

        'files'         : [ 
                            'CHANGES.txt',
                            'LICENSE.txt',
                            'README.txt',
                            'RELEASE.txt',
                            'os_spawnv_fix.diff',
                            'scons.1',
                            'script/scons.bat',
                            'setup.cfg',
                            'setup.py',
                          ],

        'filemap'       : {
                            'scons.1' : '../doc/man/scons.1',
                          },

        'subpkgs'	: [ python_scons, scons_script ],

        'subinst_dirs'	: {
                             'python-' + project : python_project_subinst_dir,
                             project + '-script' : project_script_subinst_dir,
                           },
}

src_deps = []
src_files = []

for p in [ scons ]:
    #
    # Initialize variables with the right directories for this package.
    #
    pkg = p['pkg']
    pkg_version = "%s-%s" % (pkg, version)

    src = 'src'
    if p.has_key('src_subdir'):
        src = os.path.join(src, p['src_subdir'])

    build = os.path.join('build', pkg)

    tar_gz = os.path.join(build, 'dist', "%s.tar.gz" % pkg_version)
    platform_tar_gz = os.path.join(build,
                                   'dist',
                                   "%s.%s.tar.gz" % (pkg_version, platform))
    zip = os.path.join(build, 'dist', "%s.zip" % pkg_version)
    platform_zip = os.path.join(build,
                                'dist',
                                "%s.%s.zip" % (pkg_version, platform))
    win32_exe = os.path.join(build, 'dist', "%s.win32.exe" % pkg_version)

    #
    # Update the environment with the relevant information
    # for this package.
    #
    # We can get away with calling setup.py using a directory path
    # like this because we put a preamble in it that will chdir()
    # to the directory in which setup.py exists.
    #
    setup_py = os.path.join(build, 'setup.py')
    env.Update(PKG = pkg,
               PKG_VERSION = pkg_version,
               SETUP_PY = setup_py)
    Local(setup_py)

    #
    # Read up the list of source files from our MANIFEST.in.
    # This list should *not* include LICENSE.txt, MANIFEST,
    # README.txt, or setup.py.  Make a copy of the list for the
    # destination files.
    #
    manifest_in = File(os.path.join(src, 'MANIFEST.in')).rstr()
    src_files = map(lambda x: x[:-1],
                    open(manifest_in).readlines())
    dst_files = src_files[:]

    MANIFEST_in_list = []

    if p.has_key('subpkgs'):
        #
        # This package includes some sub-packages.  Read up their
        # MANIFEST.in files, and add them to our source and destination
        # file lists, modifying them as appropriate to add the
        # specified subdirs.
        #
        for sp in p['subpkgs']:
            ssubdir = sp['src_subdir']
            isubdir = p['subinst_dirs'][sp['pkg']]
            MANIFEST_in = File(os.path.join(src, ssubdir, 'MANIFEST.in')).rstr()
            MANIFEST_in_list.append(MANIFEST_in)
            f = map(lambda x: x[:-1], open(MANIFEST_in).readlines())
            src_files.extend(map(lambda x, s=ssubdir: os.path.join(s, x), f))
            if isubdir:
                f = map(lambda x, i=isubdir: os.path.join(i, x), f)
            dst_files.extend(f)
            for k in sp['filemap'].keys():
                f = sp['filemap'][k]
                if f:
                    k = os.path.join(sp['src_subdir'], k)
                    p['filemap'][k] = os.path.join(sp['src_subdir'], f)

    #
    # Now that we have the "normal" source files, add those files
    # that are standard for each distribution.  Note that we don't
    # add these to dst_files, because they don't get installed.
    # And we still have the MANIFEST to add.
    #
    src_files.extend(p['files'])

    #
    # Now run everything in src_file through the sed command we
    # concocted to expand __FILE__, __VERSION__, etc.
    #
    for b in src_files:
        s = p['filemap'].get(b, b)
        env.SCons_revision(os.path.join(build, b), os.path.join(src, s))

    #
    # NOW, finally, we can create the MANIFEST, which we do
    # by having Python spit out the contents of the src_files
    # array we've carefully created.  After we've added
    # MANIFEST itself to the array, of course.
    #
    src_files.append("MANIFEST")
    MANIFEST_in_list.append(os.path.join(src, 'MANIFEST.in'))

    def copy(target, source, **kw):
        global src_files
	src_files.sort()
        f = open(str(target[0]), 'wb')
        for file in src_files:
            f.write(file + "\n")
        f.close()
        return 0
    env.Command(os.path.join(build, 'MANIFEST'), MANIFEST_in_list, copy)

    #
    # Now go through and arrange to create whatever packages we can.
    #
    build_src_files = map(lambda x, b=build: os.path.join(b, x), src_files)
    apply(Local, build_src_files, {})

    distutils_formats = []

    distutils_targets = [ win32_exe ]

    install_targets = distutils_targets[:]

    if gzip:

        distutils_formats.append('gztar')

        src_deps.append(tar_gz)

        distutils_targets.extend([ tar_gz, platform_tar_gz ])
        install_targets.extend([ tar_gz, platform_tar_gz ])

        #
        # Unpack the tar.gz archive created by the distutils into
        # build/unpack-tar-gz/scons-{version}.
        #
        # We'd like to replace the last three lines with the following:
        #
        #	tar zxf $SOURCES -C $UNPACK_TAR_GZ_DIR
        #
        # but that gives heartburn to Cygwin's tar, so work around it
        # with separate zcat-tar-rm commands.
        #
        unpack_tar_gz_files = map(lambda x, u=unpack_tar_gz_dir, pv=pkg_version:
                                         os.path.join(u, pv, x),
                                  src_files)
        env.Command(unpack_tar_gz_files, tar_gz, [
                    "rm -rf %s" % os.path.join(unpack_tar_gz_dir, pkg_version),
                    "$ZCAT $SOURCES > .temp",
                    "tar xf .temp -C $UNPACK_TAR_GZ_DIR",
                    "rm -f .temp",
        ])
    
        #
        # Run setup.py in the unpacked subdirectory to "install" everything
        # into our build/test subdirectory.  The runtest.py script will set
        # PYTHONPATH so that the tests only look under build/test-{package},
        # and under etc (for the testing modules TestCmd.py, TestSCons.py,
        # and unittest.py).  This makes sure that our tests pass with what
        # we really packaged, not because of something hanging around in
        # the development directory.
        #
        # We can get away with calling setup.py using a directory path
        # like this because we put a preamble in it that will chdir()
        # to the directory in which setup.py exists.
        #
        dfiles = map(lambda x, d=test_tar_gz_dir: os.path.join(d, x), dst_files)
        env.Command(dfiles, unpack_tar_gz_files, [
            "rm -rf %s" % os.path.join(unpack_tar_gz_dir, pkg_version, 'build'),
            "rm -rf $TEST_TAR_GZ_DIR",
            "$PYTHON %s install --prefix=$TEST_TAR_GZ_DIR" % \
                os.path.join(unpack_tar_gz_dir, pkg_version, 'setup.py'),
        ])

    if zipit:

        distutils_formats.append('zip')

        src_deps.append(zip)

        distutils_targets.extend([ zip, platform_zip ])
        install_targets.extend([ zip, platform_zip ])

        #
        # Unpack the zip archive created by the distutils into
        # build/unpack-zip/scons-{version}.
        #
        unpack_zip_files = map(lambda x, u=unpack_zip_dir, pv=pkg_version:
                                      os.path.join(u, pv, x),
                               src_files)

        env.Command(unpack_zip_files, zip, unzipit)

        #
        # Run setup.py in the unpacked subdirectory to "install" everything
        # into our build/test subdirectory.  The runtest.py script will set
        # PYTHONPATH so that the tests only look under build/test-{package},
        # and under etc (for the testing modules TestCmd.py, TestSCons.py,
        # and unittest.py).  This makes sure that our tests pass with what
        # we really packaged, not because of something hanging around in
        # the development directory.
        #
        # We can get away with calling setup.py using a directory path
        # like this because we put a preamble in it that will chdir()
        # to the directory in which setup.py exists.
        #
        dfiles = map(lambda x, d=test_zip_dir: os.path.join(d, x), dst_files)
        env.Command(dfiles, unpack_zip_files, [
            "rm -rf %s" % os.path.join(unpack_zip_dir, pkg_version, 'build'),
            "rm -rf $TEST_ZIP_DIR",
            "$PYTHON %s install --prefix=$TEST_ZIP_DIR" % \
                os.path.join(unpack_zip_dir, pkg_version, 'setup.py'),
        ])

    if rpm:
        topdir = os.path.join(os.getcwd(), build, 'build',
                              'bdist.' + platform, 'rpm')

	BUILDdir = os.path.join(topdir, 'BUILD', pkg + '-' + version)
	RPMSdir = os.path.join(topdir, 'RPMS', 'noarch')
	SOURCESdir = os.path.join(topdir, 'SOURCES')
	SPECSdir = os.path.join(topdir, 'SPECS')
	SRPMSdir = os.path.join(topdir, 'SRPMS')

        specfile = os.path.join(SPECSdir, "%s-1.spec" % pkg_version)
        sourcefile = os.path.join(SOURCESdir, "%s.tar.gz" % pkg_version);
        rpm = os.path.join(RPMSdir, "%s-1.noarch.rpm" % pkg_version)
        src_rpm = os.path.join(SRPMSdir, "%s-1.src.rpm" % pkg_version)

        env.InstallAs(specfile, os.path.join('rpm', "%s.spec" % pkg))
        env.InstallAs(sourcefile, tar_gz)

        targets = [ rpm, src_rpm ]
        cmd = "rpm --define '_topdir $(%s$)' -ba $SOURCES" % topdir
        if not os.path.isdir(BUILDdir):
            cmd = ("$( mkdir -p %s; $)" % BUILDdir) + cmd
        env.Command(targets, specfile, cmd)
        env.Depends(targets, sourcefile)

        install_targets.extend(targets)

        dfiles = map(lambda x, d=test_rpm_dir: os.path.join(d, 'usr', x),
                     dst_files)
        env.Command(dfiles,
                    rpm,
                    "rpm2cpio $SOURCES | (cd $TEST_RPM_DIR && cpio -id)")

    if dh_builddeb and fakeroot:
        # Our Debian packaging builds directly into build/dist,
        # so we don't need to add the .debs to install_targets.
        deb = os.path.join('build', 'dist', "%s_%s-1_all.deb" % (pkg, version))
        for d in p['debian_deps']:
            b = env.SCons_revision(os.path.join(build, d), d)
            env.Depends(deb, b)
            Local(b)
        env.Command(deb, build_src_files, [
            "cd %s && fakeroot make -f debian/rules PYTHON=$PYTHON BUILDDEB_OPTIONS=--destdir=../../build/dist binary" % build,
                    ])

        old = os.path.join('lib', 'scons', '')
        new = os.path.join('lib', 'python2.1', 'site-packages', '')
        def xxx(s, old=old, new=new):
            if s[:len(old)] == old:
                s = new + s[len(old):]
            return os.path.join('usr', s)
        dfiles = map(lambda x, t=test_deb_dir: os.path.join(t, x),
                     map(xxx, dst_files))
        env.Command(dfiles,
                    deb,
                    "dpkg --fsys-tarfile $SOURCES | (cd $TEST_DEB_DIR && tar -xf -)")

    #
    # Use the Python distutils to generate the appropriate packages.
    #
    commands = [
        "rm -rf %s" % os.path.join(build, 'build', 'lib'),
        "rm -rf %s" % os.path.join(build, 'build', 'scripts'),
    ]

    if distutils_formats:
        commands.append("rm -rf %s" % os.path.join(build,
                                                   'build',
                                                   'bdist.' + platform,
                                                   'dumb'))
        for format in distutils_formats:
            commands.append("$PYTHON $SETUP_PY bdist_dumb -f %s" % format)

        commands.append("$PYTHON $SETUP_PY sdist --formats=%s" %  \
                            string.join(distutils_formats, ','))

    commands.append("$PYTHON $SETUP_PY bdist_wininst")

    env.Command(distutils_targets, build_src_files, commands)

    #
    # And, lastly, install the appropriate packages in the
    # appropriate subdirectory.
    #
    b_d_files = env.Install(os.path.join('build', 'dist'), install_targets)
    Local(b_d_files)

#
#
#
Export('env')

SConscript('etc/SConscript')

#
# Documentation.
#
BuildDir('build/doc', 'doc')

Export('env', 'whereis')

SConscript('build/doc/SConscript')

#
# If we're running in the actual Aegis project, pack up a complete
# source archive from the project files and files in the change,
# so we can share it with helpful developers who don't use Aegis.
#

if change:
    df = []
    cmd = "aegis -list -unf -c %s cf 2>/dev/null" % change
    for line in map(lambda x: x[:-1], os.popen(cmd, "r").readlines()):
        a = string.split(line)
        if a[1] == "remove":
            df.append(a[-1])

    cmd = "aegis -list -terse pf 2>/dev/null"
    pf = map(lambda x: x[:-1], os.popen(cmd, "r").readlines())
    cmd = "aegis -list -terse cf 2>/dev/null"
    cf = map(lambda x: x[:-1], os.popen(cmd, "r").readlines())
    u = {}
    for f in pf + cf:
        u[f] = 1
    for f in df:
        del u[f]
    sfiles = filter(lambda x: x[-9:] != '.aeignore' and x[-9:] != '.sconsign',
                    u.keys())

    if sfiles:
        ps = "%s-src" % project
        psv = "%s-%s" % (ps, version)
        b_ps = os.path.join('build', ps)
        b_psv = os.path.join('build', psv)
        b_psv_stamp = b_psv + '-stamp'

        src_tar_gz = os.path.join('build', 'dist', '%s.tar.gz' % psv)
        src_zip = os.path.join('build', 'dist', '%s.zip' % psv)

        Local(src_tar_gz, src_zip)

        for file in sfiles:
            env.SCons_revision(os.path.join(b_ps, file), file)

        b_ps_files = map(lambda x, d=b_ps: os.path.join(d, x), sfiles)
        cmds = [
            "rm -rf %s" % b_psv,
            "cp -rp %s %s" % (b_ps, b_psv),
            "find %s -name .sconsign -exec rm {} \\;" % b_psv,
            "touch $TARGET",
        ]

        env.Command(b_psv_stamp, src_deps + b_ps_files, cmds)

        apply(Local, b_ps_files, {})

        if gzip:

            env.Command(src_tar_gz, b_psv_stamp,
                        "tar cz${TAR_HFLAG} -f $TARGET -C build %s" % psv)
    
            #
            # Unpack the archive into build/unpack/scons-{version}.
            #
            unpack_tar_gz_files = map(lambda x, u=unpack_tar_gz_dir, psv=psv:
                                             os.path.join(u, psv, x),
                                      sfiles)
    
            #
            # We'd like to replace the last three lines with the following:
            #
            #	tar zxf $SOURCES -C $UNPACK_TAR_GZ_DIR
            #
            # but that gives heartburn to Cygwin's tar, so work around it
            # with separate zcat-tar-rm commands.
            env.Command(unpack_tar_gz_files, src_tar_gz, [
                "rm -rf %s" % os.path.join(unpack_tar_gz_dir, psv),
                "$ZCAT $SOURCES > .temp",
                "tar xf .temp -C $UNPACK_TAR_GZ_DIR",
                "rm -f .temp",
            ])
    
            #
            # Run setup.py in the unpacked subdirectory to "install" everything
            # into our build/test subdirectory.  The runtest.py script will set
            # PYTHONPATH so that the tests only look under build/test-{package},
            # and under etc (for the testing modules TestCmd.py, TestSCons.py,
            # and unittest.py).  This makes sure that our tests pass with what
            # we really packaged, not because of something hanging around in
            # the development directory.
            #
            # We can get away with calling setup.py using a directory path
            # like this because we put a preamble in it that will chdir()
            # to the directory in which setup.py exists.
            #
            dfiles = map(lambda x, d=test_src_tar_gz_dir: os.path.join(d, x),
                            dst_files)
            ENV = env.Dictionary('ENV')
            ENV['SCONS_LIB_DIR'] = os.path.join(unpack_tar_gz_dir, psv, 'src', 'engine')
            ENV['USERNAME'] = developer
            env.Copy(ENV = ENV).Command(dfiles, unpack_tar_gz_files, [
                "rm -rf %s" % os.path.join(unpack_tar_gz_dir,
                                           psv,
                                           'build',
                                           'scons',
                                           'build'),
                "rm -rf $TEST_SRC_TAR_GZ_DIR",
                "cd %s && $PYTHON %s %s" % \
                    (os.path.join(unpack_tar_gz_dir, psv),
                     os.path.join('src', 'script', 'scons.py'),
                     os.path.join('build', 'scons')),
                "$PYTHON %s install --prefix=$TEST_SRC_TAR_GZ_DIR" % \
                    os.path.join(unpack_tar_gz_dir,
                                 psv,
                                 'build',
                                 'scons',
                                 'setup.py'),
            ])

        if zipit:

            env.Copy(PSV = psv).Command(src_zip, b_psv_stamp, zipit)
    
            #
            # Unpack the archive into build/unpack/scons-{version}.
            #
            unpack_zip_files = map(lambda x, u=unpack_zip_dir, psv=psv:
                                             os.path.join(u, psv, x),
                                      sfiles)
    
            env.Command(unpack_zip_files, src_zip, unzipit)
    
            #
            # Run setup.py in the unpacked subdirectory to "install" everything
            # into our build/test subdirectory.  The runtest.py script will set
            # PYTHONPATH so that the tests only look under build/test-{package},
            # and under etc (for the testing modules TestCmd.py, TestSCons.py,
            # and unittest.py).  This makes sure that our tests pass with what
            # we really packaged, not because of something hanging around in
            # the development directory.
            #
            # We can get away with calling setup.py using a directory path
            # like this because we put a preamble in it that will chdir()
            # to the directory in which setup.py exists.
            #
            dfiles = map(lambda x, d=test_src_zip_dir: os.path.join(d, x),
                            dst_files)
            ENV = env.Dictionary('ENV')
            ENV['SCONS_LIB_DIR'] = os.path.join(unpack_zip_dir, psv, 'src', 'engine')
            ENV['USERNAME'] = developer
            env.Copy(ENV = ENV).Command(dfiles, unpack_zip_files, [
                "rm -rf %s" % os.path.join(unpack_zip_dir,
                                           psv,
                                           'build',
                                           'scons',
                                           'build'),
                "rm -rf $TEST_SRC_ZIP_DIR",
                "cd %s && $PYTHON %s %s" % \
                    (os.path.join(unpack_zip_dir, psv),
                     os.path.join('src', 'script', 'scons.py'),
                     os.path.join('build', 'scons')),
                "$PYTHON %s install --prefix=$TEST_SRC_ZIP_DIR" % \
                    os.path.join(unpack_zip_dir,
                                 psv,
                                 'build',
                                 'scons',
                                 'setup.py'),
            ])
