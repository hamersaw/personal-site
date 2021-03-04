from invoke import Collection, task

import task.macav2 as macav2
import task.ncproj as ncproj
import task.region as region

@task(macav2.build)
def build(context):
    """
    Execute all build tasks
    """

@task(macav2.clean, region.clean)
def clean(context):
    """
    Execute all clean tasks
    """

@task(ncproj.compile)
def compile(context):
    """
    Execute all compile tasks
    """

@task(region.stage)
def stage(context):
    """
    Execute all stage tasks
    """

namespace = Collection('all', build, clean, compile, stage)
