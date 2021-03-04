from invoke import Collection, task

import config
import task.all as all
import task.macav2 as macav2
import task.ncproj as ncproj
import task.region as region

namespace = Collection()
namespace.add_collection(all.namespace);
namespace.add_collection(macav2.namespace);
namespace.add_collection(ncproj.namespace);
namespace.add_collection(region.namespace);
