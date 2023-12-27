# Binary Leaves
The `binary_leaves` folder and .zip-file contais 282 binary images of 5 different leave types. 
The binary images are taken from the [Flavia leave dataset](https://flavia.sourceforge.net/) and feature a resolution of $170 \times 128$ pixels. The dataset is intended for use in examples for binary image processing.
Exemplary images are shown below. 

|  Japanese maple |  Chinese cinnamon  | ginkgo, maidenhair tree | Chinese tulip tree | tangerine |
| :-: | :-: | :-: | :-: | :-: |
| ![a Japanes maple leave](/binary_leaves/0/000.png) | ![a Chinese cinnamon leave](/binary_leaves/1/000.png) | ![a ginkgo, maidenhair tree leave](/binary_leaves/2/000.png) | ![a Chinese tulip tree leave](/binary_leaves/3/000.png) | ![a tangerine leave](/binary_leaves/4/000.png) |
| 56 images | 55 images | 62 images | 53 images | 56 images |

## Preparation of the dataset

The following code was used to prepare the dataset. It is included here for completeness. 

```python

# file ids and common names of the leaves taken from https://flavia.sourceforge.net/
leaves = {
    'Japanese maple': range(1268,1323+1),
    'Chinese cinnamon': range(1497,1551+1),
    'ginkgo, maidenhair tree': range(2424,2485+1),
    'Chinese tulip tree': range(3511,3563+1),
    'tangerine': range(3566,3621+1),
}


# load the leaves from ./data/flavia_leaves/Leaves
train_X, train_y = [], []
idx_to_label = {}
for i, (name, fileIds) in enumerate(leaves.items()):
    for fid in fileIds:
        img = cv2.imread(f"./data/flavia_leaves/Leaves/{fid}.jpg", cv2.IMREAD_GRAYSCALE)
        # resize images to 128x128
        # compute new size to keep aspect ratio
        h, w = img.shape
        new_h = 128
        new_w = int(w*128//h)
        
        img = cv2.resize(img, (new_w, new_h), interpolation=cv2.INTER_AREA)
        binary = (img<250).astype(np.uint8)
        # find the regions and use the largest one
        retval, labels, stats, centroids = cv2.connectedComponentsWithStats(binary.astype(np.uint8))
        # find the largest region
        largest_region_id = np.argmax(stats[1:, cv2.CC_STAT_AREA]) + 1
        binary = (labels == largest_region_id).astype(np.uint8)
        
        train_X.append(binary)
        train_y.append(i)

    idx_to_label[i] = name

train_X = np.array(train_X)
train_y = np.array(train_y)

print(f"Loaded {len(train_X)} images")

# write the binary leave images to disk
outfolder = "./binary_leaves/"

import pathlib

count_per_class = np.zeros(len(idx_to_label),dtype=int)

for i in range(len(train_X)):
    path = pathlib.Path(outfolder, str(train_y[i]))
    path.mkdir(parents=True, exist_ok=True)
    cv2.imwrite(str(path / f"{count_per_class[train_y[i]]:03d}.png"), train_X[i]*255)
    count_per_class[train_y[i]] += 1

# write json with label to index mapping
import json
with open("./binary_leaves/labels.json", "w") as f:
    json.dump(idx_to_label, f)

# print a few stats about the dataset
print(f"Number of images per class: {count_per_class}")
print(f"Total number of images: {np.sum(count_per_class)}")
```