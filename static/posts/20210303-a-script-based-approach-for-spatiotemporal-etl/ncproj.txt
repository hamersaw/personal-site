from invoke import Collection, task

import os

"""
configuration variables
"""

build_dir = os.getenv('ETL_BUILD_DIR')
ncproj_dir = build_dir + '/ncproj'

@task
def update(context):
    """
    Update the ncproj git repository
    """

    # create build directory
    if os.path.isdir(build_dir):
        print('[|] build directory exists')
    else:
        print('[+] creating build directory')
        context.run('mkdir ' + build_dir)

    if os.path.isdir(ncproj_dir):
        # update git repository
        print('[+] updating ncproj repository')
        with context.cd(ncproj_dir):
            context.run('git pull')
    else:
        # clone git repository
        print('[+] cloning ncproj repository')
        context.run('git clone https://github.com/hamersaw/ncproj '
            + ncproj_dir)

@task(update)
def compile(context):
    """
    Compile the ncproj git repository
    """

    # compie ncproj repository
    print('[+] compiling ncproj repository')
    with context.cd(ncproj_dir):
        context.run('./gradlew :app:build')

namespace = Collection('ncproj', update, compile)
