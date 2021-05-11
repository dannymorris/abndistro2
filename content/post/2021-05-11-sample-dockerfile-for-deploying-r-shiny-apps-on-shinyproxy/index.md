---
title: Sample Dockerfile for deploying R Shiny apps on ShinyProxy
author: Danny Morris
date: '2021-05-11'
slug: sample-dockerfile-for-deploying-r-shiny-apps-on-shinyproxy
categories:
  - R
  - Shiny
  - Data Science
tags:
  - R
  - Shiny
  - Data Science
---

This document contains an example Dockerfile configured for deploying Shiny apps on ShinyProxy. The key is to properly configure the shiny `port` and `host`.

### Sample application structure

```
app
└───Dockerfile
└───Rprofile.site
└───app.R
```

### Dockerfile

```
# Start with base image containing R 3.6.1 and tidyverse packages
# DockerHub: https://hub.docker.com/r/rocker/tidyverse
# Other images from Rocker: https://hub.docker.com/u/rocker/
FROM rocker/tidyverse:3.6.3

# Install system dependencies needed for app
RUN apt-get update && apt-get install -y \
    sudo \
    pandoc \
    pandoc-citeproc \
    libcurl4-gnutls-dev \
    libcairo2-dev \
    libxt-dev \
    libssl-dev \
    libssh2-1-dev \
    libxml2-dev
    
# Install R packages   
RUN R -e "install.packages('remotes', repos='http://cran.rstudio.com')" &&\
    R -e "remotes::install_version('shiny', version = '1.4.0', repos='http://cran.rstudio.com')" &&\
    R -e "install.packages('shinythemes', repos='http://cran.rstudio.com')" &&\
    R -e "install.packages('shinyWidgets', repos='http://cran.rstudio.com')" &&\

# Copy all contents to the root folder in the image
RUN mkdir /root/app
COPY . /root/app

# Copy Rprofile.site to the image
COPY Rprofile.site /usr/local/lib/R/etc/

EXPOSE 3838
CMD ["R", "-e", "shiny::runApp('/root/app', port=3838, host='0.0.0.0')"]
```

### Rprofile.site

```
local({
   options(shiny.port = 3838, shiny.host = "0.0.0.0")
})
```