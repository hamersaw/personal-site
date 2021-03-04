import os

# set data environment variables
os.environ['SUSTAIN_DATA_DIR'] = '/home/hamersaw/downloads/sustain-data'

# set mongodb environment variables
os.environ['SUSTAIN_MONGODB_BIN_DIR'] = '/home/hamersaw/development/tumen/bin'
os.environ['SUSTAIN_MONGODB_DATABASE'] = 'sustaindb'
os.environ['SUSTAIN_MONGODB_HOST'] = '127.0.0.1'
os.environ['SUSTAIN_MONGODB_PORT'] = '27017'

# set build directory
os.environ['ETL_BUILD_DIR'] = 'impl/'
