from invoke import Collection, task

import glob
import os

import task.ncproj as ncproj
import task.region as region

"""
configuration variables
"""

macav2_dir = os.getenv('SUSTAIN_DATA_DIR') + '/macav2/'

county_collection = 'macav2_county'
county_index_file = macav2_dir + '/county-index.txt'

mongo_binary = os.getenv('SUSTAIN_MONGODB_BIN_DIR') + '/mongo'
mongoimport_binary = \
    os.getenv('SUSTAIN_MONGODB_BIN_DIR') + '/mongoimport'

header_str = 'gis_join,timestamp,min_specific_humidity,max_specific_humidity,min_precipitation,max_precipitation,min_surface_downwelling_shortwave_flux_in_air,max_surface_downwelling_shortwave_flux_in_air,min_min_air_temperature,max_min_air_temperature,min_max_air_temperature,max_max_air_temperature,min_eastward_wind,max_eastward_wind,min_northward_wind,max_northward_wind,min_vpd,max_vpd'

@task(ncproj.compile)
def index(context):
    """
    Compute grid index for macav2 data
    """

    # check if county index file already exists
    if os.path.isfile(county_index_file):
        print('[|] index file "' + county_index_file
            + '" already exists')
    else:
        print('[+] computing index file')

        # find netcdf file
        netcdf_filename = ''
        for filename in glob.glob(macav2_dir + '**/*.nc'):
            netcdf_filename = filename
            break

        # compute county index file
        context.run('java -cp "' + ncproj.ncproj_dir
            + '/app/build/libs/*" org.sustain.etl.ncproj.Main index'
            + ' --host=' + os.getenv('SUSTAIN_MONGODB_HOST')
            + ' --port=' + os.getenv('SUSTAIN_MONGODB_PORT')
            + ' ' + os.getenv('SUSTAIN_MONGODB_DATABASE')
            + ' ' + region.county_collection
            + ' ' + netcdf_filename + ' > ' + county_index_file)

@task(ncproj.compile, index)
def build(context):
    """
    Build macav2 data
    """

    # iterate over macav2 directories
    for directory in sorted(glob.glob(macav2_dir + '/*')):
        if not os.path.isdir(directory):
            continue

        # compute ncproj reprojection
        basename = os.path.basename(directory)
        csv_filename = macav2_dir + '/macav2-' \
            + basename + '-ncproj.csv'

        if os.path.isfile(csv_filename):
            print('[|] macav2 ncproj "' + basename
                + '" reprojection already exists')
        else:
            # compute list of netcdf files in directory
            netcdf_filenames = ''
            for filename in sorted(glob.glob(directory + '/*.nc')):
                netcdf_filenames = netcdf_filenames + ' ' + filename

            # compute reprojection
            print('[+] computing macav2 ncproj "'
                + basename + '" reprojection')

            context.run('java -cp "' + ncproj.ncproj_dir
                + '/app/build/libs/*" org.sustain.etl.ncproj.Main dump'
                + ' ' + county_index_file + ' ' + netcdf_filenames
                + ' > ' + csv_filename)

            # change header row
            context.run("sed -i '1s/^.*$/" + header_str
                + "/' " + csv_filename)

        # mongo import
        print('[+] importing mongodb collection for "' + basename + '"')
        context.run(mongoimport_binary
            + ' --host=' + os.getenv('SUSTAIN_MONGODB_HOST')
            + ' --port=' + os.getenv('SUSTAIN_MONGODB_PORT')
            + ' --db=' + os.getenv('SUSTAIN_MONGODB_DATABASE')
            + ' --collection=' + county_collection
            + ' --type=csv --headerline ' + csv_filename)

@task()
def clean(context):
    """
    Delete cached macav2 data
    """

    print("[-] deleting cached files")
    context.run('rm ' + county_index_file + ' ' + macav2_dir + '/*.csv')

namespace = Collection('macav2', build, clean, index)
