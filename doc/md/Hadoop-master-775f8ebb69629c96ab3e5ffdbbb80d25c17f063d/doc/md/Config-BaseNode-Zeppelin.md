# Zeppelin / R Base Node

Zeppelin and R applications, such a rServe, need R installed on the host container.

## Java / Scala Support

The Hadoop baseline node has Java installed.


## R Support

The R libraries need to be installed in the container.

Add OS level dependencies the R packages may require.
```
sudo apt-get install curl
sudo apt-get install libcurl4-openssl-dev
sudo apt-get -y install libcurl4-gnutls-dev libxml2-dev libssl-dev
sudo apt-get install libmariadb-client-lgpl-dev
sudo apt-get install unixODBC*

# for rJava
sudo ln -s /usr/bin/java  /usr/lib/jvm/default-java/jre/bin/java
export JAVA_HOME=/opt/jdk/jdk1.8.0_111

```
Install R and required packages.
See this tutorial for reference https://www.digitalocean.com/community/tutorials/how-to-set-up-r-on-ubuntu-14-04 

Install base and devtools packagaes.

```
# replace version with trusty (14.04) or xenial (16.04) as appropriate
sudo sh -c 'echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" >> /etc/apt/sources.list'

gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install r-base
sudo R -e "install.packages('devtools', repos = 'http://cran.us.r-project.org')"
sudo R -e "install.packages('knitr', repos = 'http://cran.us.r-project.org')"
sudo R -e "install.packages('ggplot2', repos = 'http://cran.us.r-project.org')"
sudo R -e "install.packages(c('devtools','mplot', 'googleVis'), repos = 'http://cran.us.r-project.org'); require(devtools); install_github('ramnathv/rCharts')"

```
Install additional requested R packages.

```
sudo R -e "install.packages(c( \
'car', \
'caret', \
'data.table', \
'devtools', \
'diagrammer', \
'DiagrammeR', \
'dplyr', \
'DT', \
'dygraphs', \
'foreign', \
'ggmap', \
'ggplot2', \
'ggvis', \
'gimnet', \
'htmlwidgets', \
'httr', \
'jsonlite', \
'Knitr', \
'knitr', \
'leaflet', \
'lme4', \
'lubridate', \
'maps', \
'multcomp', \
'network3D', \
'nime', \
'nycflights13', \
'parallel', \
'quantmod', \
'randomForest', \
'Rcurl', \
'RCurl', \
'rgi', \
'RMySQL', \
'RODBC', \
'roxygen2', \
'RSQLite', \
'shiney', \
'shiny', \
'sqldf', \
'stringr', \
'tensorflow', \
'testthat', \
'threeJS', \
'threejs', \
'tidyr', \
'vcd', \
'XLConnect', \
'xlsx', \
'XML', \
'xts', \
'zoo' \
), repos = 'http://cran.us.r-project.org')"

```

## Installed Packages

```
               Package          LibPath                         Version
BH             "BH"             "/usr/local/lib/R/site-library" "1.62.0-1"
DBI            "DBI"            "/usr/local/lib/R/site-library" "0.7"
DT             "DT"             "/usr/local/lib/R/site-library" "0.2"
DiagrammeR     "DiagrammeR"     "/usr/local/lib/R/site-library" "0.9.0"
MatrixModels   "MatrixModels"   "/usr/local/lib/R/site-library" "0.4-1"
ModelMetrics   "ModelMetrics"   "/usr/local/lib/R/site-library" "1.1.0"
NMF            "NMF"            "/usr/local/lib/R/site-library" "0.20.6"
R6             "R6"             "/usr/local/lib/R/site-library" "2.2.2"
RColorBrewer   "RColorBrewer"   "/usr/local/lib/R/site-library" "1.1-2"
RCurl          "RCurl"          "/usr/local/lib/R/site-library" "1.95-4.8"
RJSONIO        "RJSONIO"        "/usr/local/lib/R/site-library" "1.3-0"
RMySQL         "RMySQL"         "/usr/local/lib/R/site-library" "0.10.12"
RODBC          "RODBC"          "/usr/local/lib/R/site-library" "1.3-15"
RSQLite        "RSQLite"        "/usr/local/lib/R/site-library" "2.0"
Rcpp           "Rcpp"           "/usr/local/lib/R/site-library" "0.12.11"
RcppEigen      "RcppEigen"      "/usr/local/lib/R/site-library" "0.3.3.3.0"
RgoogleMaps    "RgoogleMaps"    "/usr/local/lib/R/site-library" "1.4.1"
Rook           "Rook"           "/usr/local/lib/R/site-library" "1.1-1"
SparseM        "SparseM"        "/usr/local/lib/R/site-library" "1.77"
TH.data        "TH.data"        "/usr/local/lib/R/site-library" "1.0-8"
TTR            "TTR"            "/usr/local/lib/R/site-library" "0.23-1"
XLConnect      "XLConnect"      "/usr/local/lib/R/site-library" "0.2-13"
XLConnectJars  "XLConnectJars"  "/usr/local/lib/R/site-library" "0.2-13"
XML            "XML"            "/usr/local/lib/R/site-library" "3.98-1.9"
assertthat     "assertthat"     "/usr/local/lib/R/site-library" "0.2.0"
backports      "backports"      "/usr/local/lib/R/site-library" "1.1.0"
base64enc      "base64enc"      "/usr/local/lib/R/site-library" "0.1-3"
bestglm        "bestglm"        "/usr/local/lib/R/site-library" "0.36"
bindr          "bindr"          "/usr/local/lib/R/site-library" "0.1"
bindrcpp       "bindrcpp"       "/usr/local/lib/R/site-library" "0.2"
bit            "bit"            "/usr/local/lib/R/site-library" "1.1-12"
bit64          "bit64"          "/usr/local/lib/R/site-library" "0.9-7"
bitops         "bitops"         "/usr/local/lib/R/site-library" "1.0-6"
blob           "blob"           "/usr/local/lib/R/site-library" "1.1.0"
brew           "brew"           "/usr/local/lib/R/site-library" "1.0-6"
car            "car"            "/usr/local/lib/R/site-library" "2.1-5"
caret          "caret"          "/usr/local/lib/R/site-library" "6.0-76"
chron          "chron"          "/usr/local/lib/R/site-library" "2.3-50"
colorspace     "colorspace"     "/usr/local/lib/R/site-library" "1.3-2"
commonmark     "commonmark"     "/usr/local/lib/R/site-library" "1.2"
crayon         "crayon"         "/usr/local/lib/R/site-library" "1.3.2"
crosstalk      "crosstalk"      "/usr/local/lib/R/site-library" "1.0.0"
curl           "curl"           "/usr/local/lib/R/site-library" "2.7"
data.table     "data.table"     "/usr/local/lib/R/site-library" "1.10.4"
debugme        "debugme"        "/usr/local/lib/R/site-library" "1.0.2"
desc           "desc"           "/usr/local/lib/R/site-library" "1.1.0"
devtools       "devtools"       "/usr/local/lib/R/site-library" "1.13.2"
dichromat      "dichromat"      "/usr/local/lib/R/site-library" "2.0-0"
digest         "digest"         "/usr/local/lib/R/site-library" "0.6.12"
doParallel     "doParallel"     "/usr/local/lib/R/site-library" "1.0.10"
dplyr          "dplyr"          "/usr/local/lib/R/site-library" "0.7.1"
dygraphs       "dygraphs"       "/usr/local/lib/R/site-library" "1.1.1.4"
evaluate       "evaluate"       "/usr/local/lib/R/site-library" "0.10.1"
foreach        "foreach"        "/usr/local/lib/R/site-library" "1.4.3"
foreign        "foreign"        "/usr/local/lib/R/site-library" "0.8-69"
geosphere      "geosphere"      "/usr/local/lib/R/site-library" "1.5-5"
ggmap          "ggmap"          "/usr/local/lib/R/site-library" "2.6.1"
ggplot2        "ggplot2"        "/usr/local/lib/R/site-library" "2.2.1"
ggvis          "ggvis"          "/usr/local/lib/R/site-library" "0.4.3"
git2r          "git2r"          "/usr/local/lib/R/site-library" "0.18.0"
glmnet         "glmnet"         "/usr/local/lib/R/site-library" "2.0-10"
glmulti        "glmulti"        "/usr/local/lib/R/site-library" "1.0.7"
glue           "glue"           "/usr/local/lib/R/site-library" "1.1.1"
googleVis      "googleVis"      "/usr/local/lib/R/site-library" "0.6.2"
gridBase       "gridBase"       "/usr/local/lib/R/site-library" "0.4-7"
gridExtra      "gridExtra"      "/usr/local/lib/R/site-library" "2.2.1"
grpreg         "grpreg"         "/usr/local/lib/R/site-library" "3.1-1"
gsubfn         "gsubfn"         "/usr/local/lib/R/site-library" "0.6-6"
gtable         "gtable"         "/usr/local/lib/R/site-library" "0.2.0"
highr          "highr"          "/usr/local/lib/R/site-library" "0.6"
htmltools      "htmltools"      "/usr/local/lib/R/site-library" "0.3.6"
htmlwidgets    "htmlwidgets"    "/usr/local/lib/R/site-library" "0.8"
httpuv         "httpuv"         "/usr/local/lib/R/site-library" "1.3.3"
httr           "httr"           "/usr/local/lib/R/site-library" "1.2.1"
igraph         "igraph"         "/usr/local/lib/R/site-library" "1.0.1"
influenceR     "influenceR"     "/usr/local/lib/R/site-library" "0.1.0"
irlba          "irlba"          "/usr/local/lib/R/site-library" "2.2.1"
iterators      "iterators"      "/usr/local/lib/R/site-library" "1.0.8"
jpeg           "jpeg"           "/usr/local/lib/R/site-library" "0.1-8"
jsonlite       "jsonlite"       "/usr/local/lib/R/site-library" "1.5"
knitr          "knitr"          "/usr/local/lib/R/site-library" "1.16"
labeling       "labeling"       "/usr/local/lib/R/site-library" "0.3"
lazyeval       "lazyeval"       "/usr/local/lib/R/site-library" "0.2.0"
leaflet        "leaflet"        "/usr/local/lib/R/site-library" "1.1.0"
leaps          "leaps"          "/usr/local/lib/R/site-library" "3.0"
lme4           "lme4"           "/usr/local/lib/R/site-library" "1.1-13"
lmtest         "lmtest"         "/usr/local/lib/R/site-library" "0.9-35"
lubridate      "lubridate"      "/usr/local/lib/R/site-library" "1.6.0"
magrittr       "magrittr"       "/usr/local/lib/R/site-library" "1.5"
mapproj        "mapproj"        "/usr/local/lib/R/site-library" "1.2-5"
maps           "maps"           "/usr/local/lib/R/site-library" "3.2.0"
markdown       "markdown"       "/usr/local/lib/R/site-library" "0.8"
memoise        "memoise"        "/usr/local/lib/R/site-library" "1.1.0"
mime           "mime"           "/usr/local/lib/R/site-library" "0.5"
minqa          "minqa"          "/usr/local/lib/R/site-library" "1.2.4"
mplot          "mplot"          "/usr/local/lib/R/site-library" "0.7.9"
multcomp       "multcomp"       "/usr/local/lib/R/site-library" "1.4-6"
munsell        "munsell"        "/usr/local/lib/R/site-library" "0.4.3"
mvtnorm        "mvtnorm"        "/usr/local/lib/R/site-library" "1.0-6"
nloptr         "nloptr"         "/usr/local/lib/R/site-library" "1.0.4"
nycflights13   "nycflights13"   "/usr/local/lib/R/site-library" "0.2.2"
openssl        "openssl"        "/usr/local/lib/R/site-library" "0.9.6"
pbkrtest       "pbkrtest"       "/usr/local/lib/R/site-library" "0.4-7"
pkgconfig      "pkgconfig"      "/usr/local/lib/R/site-library" "2.0.1"
pkgmaker       "pkgmaker"       "/usr/local/lib/R/site-library" "0.22"
plogr          "plogr"          "/usr/local/lib/R/site-library" "0.1-1"
plyr           "plyr"           "/usr/local/lib/R/site-library" "1.8.4"
png            "png"            "/usr/local/lib/R/site-library" "0.1-7"
praise         "praise"         "/usr/local/lib/R/site-library" "1.0.0"
processx       "processx"       "/usr/local/lib/R/site-library" "2.0.0"
proto          "proto"          "/usr/local/lib/R/site-library" "1.0.0"
quantmod       "quantmod"       "/usr/local/lib/R/site-library" "0.4-10"
quantreg       "quantreg"       "/usr/local/lib/R/site-library" "5.33"
rCharts        "rCharts"        "/usr/local/lib/R/site-library" "0.4.5"
rJava          "rJava"          "/usr/local/lib/R/site-library" "0.9-8"
randomForest   "randomForest"   "/usr/local/lib/R/site-library" "4.6-12"
raster         "raster"         "/usr/local/lib/R/site-library" "2.5-8"
registry       "registry"       "/usr/local/lib/R/site-library" "0.3"
reshape2       "reshape2"       "/usr/local/lib/R/site-library" "1.4.2"
reticulate     "reticulate"     "/usr/local/lib/R/site-library" "0.9"
rgexf          "rgexf"          "/usr/local/lib/R/site-library" "0.15.3"
rjson          "rjson"          "/usr/local/lib/R/site-library" "0.2.15"
rlang          "rlang"          "/usr/local/lib/R/site-library" "0.1.1"
rngtools       "rngtools"       "/usr/local/lib/R/site-library" "1.2.4"
roxygen2       "roxygen2"       "/usr/local/lib/R/site-library" "6.0.1"
rprojroot      "rprojroot"      "/usr/local/lib/R/site-library" "1.2"
rstudioapi     "rstudioapi"     "/usr/local/lib/R/site-library" "0.6"
sandwich       "sandwich"       "/usr/local/lib/R/site-library" "2.3-4"
scales         "scales"         "/usr/local/lib/R/site-library" "0.4.1"
shiny          "shiny"          "/usr/local/lib/R/site-library" "1.0.3"
shinydashboard "shinydashboard" "/usr/local/lib/R/site-library" "0.6.1"
sourcetools    "sourcetools"    "/usr/local/lib/R/site-library" "0.1.6"
sp             "sp"             "/usr/local/lib/R/site-library" "1.2-5"
sqldf          "sqldf"          "/usr/local/lib/R/site-library" "0.4-11"
stringi        "stringi"        "/usr/local/lib/R/site-library" "1.1.5"
stringr        "stringr"        "/usr/local/lib/R/site-library" "1.2.0"
tensorflow     "tensorflow"     "/usr/local/lib/R/site-library" "0.9"
testthat       "testthat"       "/usr/local/lib/R/site-library" "1.0.2"
threejs        "threejs"        "/usr/local/lib/R/site-library" "0.2.2"
tibble         "tibble"         "/usr/local/lib/R/site-library" "1.3.3"
tidyr          "tidyr"          "/usr/local/lib/R/site-library" "0.6.3"
vcd            "vcd"            "/usr/local/lib/R/site-library" "1.4-3"
viridis        "viridis"        "/usr/local/lib/R/site-library" "0.4.0"
viridisLite    "viridisLite"    "/usr/local/lib/R/site-library" "0.2.0"
visNetwork     "visNetwork"     "/usr/local/lib/R/site-library" "2.0.0"
whisker        "whisker"        "/usr/local/lib/R/site-library" "0.3-2"
withr          "withr"          "/usr/local/lib/R/site-library" "1.0.2"
xlsx           "xlsx"           "/usr/local/lib/R/site-library" "0.5.7"
xlsxjars       "xlsxjars"       "/usr/local/lib/R/site-library" "0.6.1"
xml2           "xml2"           "/usr/local/lib/R/site-library" "1.1.1"
xtable         "xtable"         "/usr/local/lib/R/site-library" "1.8-2"
xts            "xts"            "/usr/local/lib/R/site-library" "0.9-7"
yaml           "yaml"           "/usr/local/lib/R/site-library" "2.1.14"
zoo            "zoo"            "/usr/local/lib/R/site-library" "1.8-0"
KernSmooth     "KernSmooth"     "/usr/lib/R/library"            "2.23-15"
MASS           "MASS"           "/usr/lib/R/library"            "7.3-47"
Matrix         "Matrix"         "/usr/lib/R/library"            "1.2-10"
base           "base"           "/usr/lib/R/library"            "3.4.0"
boot           "boot"           "/usr/lib/R/library"            "1.3-19"
class          "class"          "/usr/lib/R/library"            "7.3-14"
cluster        "cluster"        "/usr/lib/R/library"            "2.0.6"
codetools      "codetools"      "/usr/lib/R/library"            "0.2-15"
compiler       "compiler"       "/usr/lib/R/library"            "3.4.0"
datasets       "datasets"       "/usr/lib/R/library"            "3.4.0"
foreign        "foreign"        "/usr/lib/R/library"            "0.8-69"
grDevices      "grDevices"      "/usr/lib/R/library"            "3.4.0"
graphics       "graphics"       "/usr/lib/R/library"            "3.4.0"
grid           "grid"           "/usr/lib/R/library"            "3.4.0"
lattice        "lattice"        "/usr/lib/R/library"            "0.20-35"
methods        "methods"        "/usr/lib/R/library"            "3.4.0"
mgcv           "mgcv"           "/usr/lib/R/library"            "1.8-17"
nlme           "nlme"           "/usr/lib/R/library"            "3.1-131"
nnet           "nnet"           "/usr/lib/R/library"            "7.3-12"
parallel       "parallel"       "/usr/lib/R/library"            "3.4.0"
rpart          "rpart"          "/usr/lib/R/library"            "4.1-11"
spatial        "spatial"        "/usr/lib/R/library"            "7.3-11"
splines        "splines"        "/usr/lib/R/library"            "3.4.0"
stats          "stats"          "/usr/lib/R/library"            "3.4.0"
stats4         "stats4"         "/usr/lib/R/library"            "3.4.0"
survival       "survival"       "/usr/lib/R/library"            "2.41-3"
tcltk          "tcltk"          "/usr/lib/R/library"            "3.4.0"
tools          "tools"          "/usr/lib/R/library"            "3.4.0"
utils          "utils"          "/usr/lib/R/library"            "3.4.0"

```
