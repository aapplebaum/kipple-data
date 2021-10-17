# kipple-data
This repository houses the data associated with the _kipple_ project. It has two primary folders:

* _data_, which contains files of **zipped** memmap'd feature arrays for adversarial malware, and
* _records_, which contains a list of the associated md5/sha256 value (more below) for each dat file.

Note that for each dat file in _data_ there is an associated txt file in _records_ with the latter listing the md5/sha256 values encoded in the array.

In total, there are 13 data stores, matching the following table:

| Name      | Description | Count|
| ------------| ------------| ------------|
| msf_normal      | Randomly generated implants from msfvenom, no added-code parameter       | 5884|
| msf_sorel      | Randomly generated implants from msfvenom, added-code from the SoReL dataset       | 33633|
| msf_vs      | Randomly generated implants from msfvenom, added-code from VirusShare       | 7614|
| sorel_malware_rl | Adversarial malware generated using Malware RL over the SoReL dataset | 37553 |
| sorel_sml_gamma | Adversarial malware generated using the GAMMA attack from SecML Malware on the SoReL dataset | 5167|
| sorel_small_pad | Adversarial malware generated using the padding attack with a small pad from SecML Malware on the SoReL dataset | 225|
| sorel_large_pad | Adversarial malware generated using the padding attack with a large pad from SecML Malware on the SoReL dataset | 277|
| sorel_header_ev | Adversarial malware generated using the DOS Header attack from SecML Malware on the SoReL dataset | 2590 |
| vs_malware_rl | Adversarial malware generated using Malware RL over malware from VirusShare | 24581 |
| vs_sml_gamma | Adversarial malware generated using the GAMMA attack from SecML Malware on malware from VirusShare | 5629|
| vs_small_pad | Adversarial malware generated using the padding attack with a small pad from SecML Malware on malware from VirusShare | 2347|
| vs_large_pad | Adversarial malware generated using the padding attack with a large pad from SecML Malware on malware from VirusShare | 2815|
| vs_header_ev | Adversarial malware generated using the DOS Header attack from SecML Malware on malware from VirusShare | 2814 |

# Pre-requisites #
This data is **zipped**. The main kipple repo assumes you will unzip it -- we _strongly_ recommend unzipping once you download the repo. The zip is only to make sure we're in line with file size requirements.

# Usage #
**Assuming you've already unzipped**, the following code would be an example of running a classifier over the kipple data:
```
import ember
import os
from ember.features import PEFeatureExtractor
import lightgbm as lgb
import gzip
import numpy as np

# Load EMBER feature extractor + number of dimensions
extractor=PEFeatureExtractor(feature_version=2, print_feature_warning=False)
ndim = extractor.dim

# Load the data in the array we want to use
target_data="msf_normal"
num_entries=sum(1 for line in open("records/" + target_data + ".txt"))
malware_data = np.memmap("data/" + target_data + ".dat", dtype=np.float32, mode="r", shape=(5884, ndim))

# Load a local model
model_location="/exes/kipple_repo/kipple/models/initial.txt.gz"
with gzip.open(model_location,"rb") as f:
    md=f.read().decode('ascii')
mdl=lgb.Booster(model_str=md)

num_correct=0
for i in range (0, num_entries):
    if mdl.predict([malware_data[i]])[0] > .85:
        num_correct=num_correct+1
print(num_correct/num_entries)

```
There are more examples in the primary kipple directory.

# Useful References #
* Malware RL: https://github.com/bfilar/malware_rl
* SoReL 20M: https://github.com/sophos-ai/SOREL-20M
* SecML Malware: https://github.com/pralab/secml_malware/
* msfvenom: https://www.offensive-security.com/metasploit-unleashed/msfvenom/
* EMBER: https://github.com/elastic/ember
* VirusShare: https://virusshare.com/
