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

# for rJava
sudo ln -s /usr/bin/java  /usr/lib/jvm/default-java/jre/bin/java
export JAVA_HOME=/opt/jdk/jdk1.8.0_111

```
Install R and required packages.
See this tutorial for reference https://www.digitalocean.com/community/tutorials/how-to-set-up-r-on-ubuntu-14-04 

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

