# FORTLS
![license](https://img.shields.io/badge/Licence-GPL--3-blue.svg)
[![](https://www.r-pkg.org/badges/version/FORTLS)](https://CRAN.R-project.org/package=FORTLS)
[![DOI](https://zenodo.org/badge/DOI/10.3390/IECF2020-08066.svg)](https://doi.org/10.3390/IECF2020-08066)

## Automatic Processing of TLS Point Cloud Data for Forestry Purposes
Process automation of Terrestrial Laser Scanner (TLS) point cloud data derived from single
scans. 'FORTLS' enables (i) detection of trees and estimation of diameter
at breast height (dbh), (ii) estimation of some stand variables (e.g. density,
basal area, mean and dominant height), (iii) computation of metrics related to important forest
attributes estimated in Forest Inventories (FIs) at stand level and (iv) optimization of plot design
for combining TLS data and field measured data. Documentation about 'FORTLS' is described
in Molina-Valero et al. (2020, <https://doi.org/10.3390/IECF2020-08066>).

# Install `FORTLS 1.2.0` (Beta version)

Get the latest released version of FORTLS from GitHub (included in the devel branch)

```r
remotes::install_github("Molina-Valero/FORTLS", ref = "devel", dependencies = TRUE)
```


## Normalize

This function obtains coordinates relative to the plot centre for Terrestrial Laser Scanner (TLS) and SLAM point clouds (supplied as LAS files) derived from single and multiple scans. 

```r
# Establishment of working directories (optional)
# By default here we propose the current working directory of the R process

dir.data <- getwd()
dir.result <- getwd()

# Loading example data (LAS file) to dir.data

download.file("https://www.dropbox.com/s/2c3d320o3srcawb/1.las?raw=1", destfile = file.path(dir.data, "1.las"), mode = "wb")

download.file("https://www.dropbox.com/s/2c3d320o3srcawb/2.las?raw=1", destfile = file.path(dir.data, "2.las"), mode = "wb")

download.file("https://www.dropbox.com/s/j15lkv0rv2id69e/multiple.scans.las?raw=1", destfile = file.path(dir.data, "multi.scans.las"), mode = "wb")

# Normalizing a single scan point cloud data without considering arguments

pcd.single.scan <- normalize(las = "1.las", dir.data = dir.data, dir.result = dir.result)

# Normalizing a SLAM point cloud data without considering arguments

pcd.multi.scans <- normalize(las = "multi.scans.las", multi.scans = TRUE, dir.data = dir.data, dir.result = dir.result)
```

## Tree detection
### Tree detection from Terrestrial Laser Scanning (TLS) single scans point clouds

Detects trees from TLS point clouds corresponding to a single scan. For each tree detected, the function calculates the central coordinates and estimates the diameter at 1.3 m above ground level (which is known as dbh, diameter at breast height) and classifies it as fully visible or partially occluded. Finally, the function obtains the number of points belonging to normal sections of trees (those corresponding to dbh +/- 5 cm) and estimates them for both original and reduced (with point cropping process) point clouds.

```r
# Tree detection without considering arguments
# For this case study, TLS resolution was established as:
# point.dist = 7.67 mm and tls.dist = 10 m

tree.list.tls <- tree.detection.single.scan(data = pcd.single.scan, tls.resolution = list(point.dist = 7.67, tls.dist = 10), dir.result = dir.result)
```
### Tree detection from multi scans or Simultaneous Localisation And Mapping (SLAM) point clouds

```r
tree.list.tls <- tree.detection.multi.scans(data = pcd.multi.scans, dir.result = dir.result)
```

### Tree detection from several plots
For analysis of .las files corresponding to several plots, tree.detection.several.plots is the most appropriate function as it integrates both the normalize and tree.detection functions and allows a character vector containing multiple .las file names to be included as an argument. For instance, all .las files in the dir.data directory can be jointly analyzed as follows.
```r
files <- list.files(pattern = "las$", path = dir.data)[1:2]

tree.list.tls <- tree.detection.several.plots(las.list = files,

                                              normalize.arguments = list(max.dist = 15,
                                                                         algorithm.dtm = "knnidw",
                                                                         res.dtm = 0.25),

                                              tree.detection.arguments = list(dbh.min = 7.5, dbh.max = 100,
                                                                              breaks = 1.3,
                                                                              tls.resolution = list(point.dist = 7.67,
                                                                                                    tls.dist = 10)),

                                              dir.data = dir.data, dir.result = dir.result)
```
## Distance sampling
Sampling methodologies are implemented in the `distance.sampling` function. They are designed to correct estimation bias caused by lack of detection of trees due to occlusion. Although this is an optional previous step for other  **FORTLS** functions (such as `metrics.variables` and `simulations`), its inclusion is highly recommended, especially when high rates of occlusion occur. Distance sampling methods are applied to all trees detected from 16 TLS single scans corresponding to plots located in La Rioja (a region of Spain): this is one of the examples included in the **FORTLS** package.
```{r warning=FALSE}
# Load 'Rioja.data' dataset (provided as example data in FORTLS)

data(Rioja.data)

# Select trees detected from TLS data contained in 'Rioja.data' dataset

tree.list.tls <- Rioja.data$tree.list.tls

# Apply distance sampling methods to all detected trees

tree.ds <- distance.sampling(tree.list.tls)

# Summary of distance sampling

summary(tree.ds)

# Detection probability for first two detected trees

head(tree.ds$tree, 2)
```
