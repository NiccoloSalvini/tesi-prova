# REST API Infrastructure {#Infrastructure}

<!--  You can label chapter and section titles using `{#label}` after them, e.g., we can reference Chapter \@ref(intro). If you do not manually label them, there will be automatic labels anyway, e.g., Chapter \@ref(methods).-->

In order to provide a fast and easy to use API service to the end user many technologies have been involved. Challenges in scraping as pointed out in section \@ref(challenges) are many and still some remains unsolved. Challenges regards not only scraping per se, but also the way the service has to interact with the users. Interactions are many and consequently are problems arised. API service has to be fast otherwise data become obsolete and so happen to the analysis that have relied on those data. Service has to be deployed so that it can be shared over a wider range of clients. Service has to be scalable at need since, due to deployment, when the number of users increases the run time performance should not decrease. Moreover from one hand service has to be continuously integrated and reviewed so that each function can be responsive to immobiliare.it changes. But on the other code behind the service has to be version controlled and freezed, so that when packages are updated service can prevent fails. API has to be also secured granting access only to the ones authorized. In the end Service has to be run at a certain given times and storing data on cloud database, so that it can be tracked back the evolution of the phenomenon under inspection.
Open source solutions for each of the requirements stated are available for back-end and front-end integration. Moreover documentations related to technologies served are able to offer flexible solutions to be embedded into the R ecosystem.
For all the requirements the idea is to provide a REST Plumber API with 4 endpoints which calls parallelized scraping functions built in section \@ref(scraping). On top of that a daily Cron Job scheduler, exposing one API endpoint, produces and later stores a .csv file in a NOSQL mongoDB Atlas could database. Containerization happens through a Linux OS (Ubuntu distr) Docker container hosted by a AWS EC2 server. API endpoints are secured with https protocols and protected with authentication by nginx reverse proxy. On a second server a Shiny App calls one endpoint with specified parameters which returns daily data from the former infrastructure.

Technologies involved are:

- GitHub version control
- Scheduler cron job, section \@ref(scheduler)
- Docker containers, section \@ref(docker)
- Plumber REST API, section \@ref(plumberapi)
- NGINX reverse proxy, section \@ref(nginx)
- AWS (Amazon Web Services) EC2 \@ref(aws)
- MongoDB Atlas
- Shiny, see chapter \@ref(application)


![complete infrastructure (Matt Dancho source)](images/prova.PNG)

As a side note each single part of this thesis has been made according to the same API inspiring criteria of reproducibility and self containerization. RMarkdown [@rmarkdown1] documents (book's chapters) are compiled and then converted into .html files. Through Bookdown [@bookdown2] the resulting documents are put together according to general .yml instruction file and are readble as gitbook.
Files are then pushed to a [Github repository](https://github.com/NiccoloSalvini/thesis). By a simple trick with GH pages, .html files are dispalyed into a Github subdomain hosted at [link](https://niccolosalvini.github.io/thesis/).  The resulting  deployed gitbook can also produce a .pdf version output through a Xelatex engine. Xelatex compiles .Rmd documents according to a .tex template which formatting rules are contained in a further .yml file. The pdf version of the thesis can be obtained by clicking the download button, then choosing pdf output version in the upper banner. For further references on the topic @bookdown2

Some of the main technologies implied will be viewed singularly, nonetheless for brevity reasosn some needs to be skipped.


## Scheduler{#scheduler}

\BeginKnitrBlock{definition}\iffalse{-91-83-99-104-101-100-117-108-101-114-93-}\fi{}
<span class="definition" id="def:scheduler"><strong>(\#def:scheduler)  \iffalse (Scheduler) \fi{} </strong></span>A Scheduler in a process is a component on a OS that allows the computer to decide which activity is going to be executed. In the context of multi-programming it is thought as a tool to keep CPU occupied as much as possible.
\EndKnitrBlock{definition}


As an example it can trigger a process while some other is still waiting to finish. There are many type of scheduler and they are based on the frequency of times they are executed considering a certain closed time neighbor.

- Short term scheduler: it can trigger and queue the "ready to go" tasks
  - with pre-emption 
  - without pre-emption

The ST scheduler selects the process and It gains control of the CPU by the dispatcher. In this context we can define latency as the time needed to stop a process and to start a new one. 

- Medium term scheduler 
- Long term scheduler

for some other useful but beyond the scope refereces, such as the scheduling algorithm the reader can refer to [@wiki:scheduler].

### Cron Jobs
\BeginKnitrBlock{definition}\iffalse{-91-67-114-111-110-106-111-98-93-}\fi{}
<span class="definition" id="def:cronjob"><strong>(\#def:cronjob)  \iffalse (Cronjob) \fi{} </strong></span>Cron job is a software utility which acts as a time-based job scheduler in Unix-like OS. Linux users that set up and maintain software environments exploit cron to schedule their day-to-day routines to run periodically at fixed times, dates, or intervals. It typically automates system maintenance but its usage is very flexible to whichever needed. It is lightweight and it is widely used since it is a common option for Linux users.
\EndKnitrBlock{definition}
The tasks by cron are driven by a crontab file, which is a configuration file that specifies a set of commands to run periodically on a given schedule. The crontab files are stored where the lists of jobs and other instructions to the cron daemon are kept.

Each line of a crontab file represents a job, and has this structure

![crontab](images/crontab.PNG)

Each line of a crontab file represents a job. This example runs a shell named scheduler.sh at 23:45 (11:45 PM) every Saturday. .sh commands can update mails and other minor routines.

45 23 * * 6 /home/oracle/scripts/scheduler.sh

Some rather unusual scheduling definitions for crontabs can be found in this reference [@wiki:cronjob]. Crontab's syntax completion can be made easier through [this](https://crontab.guru/) GUI.  

The cron job needs to be ran on scraping fucntions at 11:30 PM every single day. The get_data.R script first sources an endpoint function, then it applies the function with fixed parameters. Parameters describe the url specification, so that each time the scheduler runs the get_data.R collects data from the same source. Day after day .json files are generated and then stored into a NOSQL *mongoDB* database whose credentials are public. Data are collected on a daily basis with the explicit aim to track day-by-day changes both in the new entries an goners in rental market, and to investigate the evolution of price differentials over time. Spatio-Temporal modeling is still quite unexplored, data is saved for future used. Crontab configuration for daily 11:30 PM schedules has this appearance:

30 11 * * * /home/oracle/scripts/get_data.R

Since now the computational power comes from the machine on which the system is installed. A smarter solution takes care of it by considering run time limits and the substantial inability to share data. To a certain extent what it has been already done since now might fit for personal use: a scheduler can daily execute the scraping scripts  and  generate a .csv file. Furthermore an application can rely on those data, but evident reasons suggest that it does not suite any need. What it will do the trick would be an open source dedicated software environment or *container* that will contains scraping functions and a scheduler on cloud solving a pair of the problems arisen. This problem can be addressed with a technology that has seen a huge growth in its usage in the last few years.

## Docker{#docker}

\BeginKnitrBlock{definition}\iffalse{-91-68-111-99-107-101-114-93-}\fi{}
<span class="definition" id="def:docker"><strong>(\#def:docker)  \iffalse (Docker) \fi{} </strong></span>_Docker_ is a software tool to create and deploy applications using containers.
_Docker containers_ are a standard unit of software (i.e. software boxes) where everything needed for applications, such as libraries or dependencies can be run reliably and quickly. Furthermore they are also portable, in the sense that they can be taken from one computing environment to the following. Docker containers by default run on kernel Linux OS.
\EndKnitrBlock{definition}
Containers can be thought as an abstraction at the app layers that groups code and dependencies together. One major advantage of containers is that multiple containers can run on the same machine with the same OS. Each container can run its own isolated process in the user space, so that each task is complementary to the other. Containers are lightweight and take up less space than Virtual Machines (container images are files which can take up typically tens of MBs in size), can handle more applications and require fewer Virtual Machines and OSs.


![docker container vs VM](images/dockerVSvirtualmachines.PNG)

When containers are built _Docker container Images_ are created and can be open sourced through Docker Hub.
_Docker Hub_ is a web service provided by Docker for searching and sharing container images with other teams or developers in the community. Docker Hub can connect with GitHub behind authorization entailing an image version control tool. Once the connection is established  changes that are pushed with git to the GitHub repository are passed to Docker Hub. The push command automatically triggers the image building. Then docker image can be tagged (salvini/api-immobiliare:latest)so that on one hand it is recognizable and on the other can be reused in the future. Once the building stage is completed the DH repository can be pulled and then run locally on machine or cloud, see section \@ref(aws).
Docker building and testing images can be very time consuming. R packages can take a long time to install because code has to be compiled, especially if using R on a Linux server or in a Docker container. 
Rstudio [package manager](https://packagemanager.rstudio.com/client/#/) includes beta support for pre-compiled R packages that can be installed faster. This dramatically reduces packages time installation [@nolis_2020].
In addition to that an open source project named [rocker](https://www.rocker-project.org/images/) has narrowed the path fro developers by building custom R docker images for a wide range of usages. What can be read from their own website about the project is: "The rocker project provides a collection of containers suited for different needs. find a base image to extend or images with popular software and optimized libraries pre-installed. Get the latest version or a reproducible fixed environment." 

### Why Docker

[Indeed](https://it.indeed.com/), an employment-related search engine, released an article on 2019 displaying changing trends from 2015 to 2019 in Technology Job market. Many changes are relevant in key technologies. Two among the others technologies (i.e. docker and Azure) have experienced a huge growth and both refer to the same demand input: _containers_ .
The landscape of Data Science is changing [@Skills_Explorer] from reporting to application building:
In 2015 - Businesses reports drive better decisions
In 2020 - Businesses need apps to empower better decision making at all levels

![docker-stats](images/Inkedindeed_jobs_LI.jpg)

For all the things said what docker is bringing to business [@red_hat_customer_portal]:

- _Speed application deployment_ : containers include the minimal run time requirements of the application, reducing their size and allowing them to be deployed quickly.
- _Portability across machines_ : an application and all its dependencies can be bundled into a single container that is independent from the host version of Linux kernel, platform distribution, or deployment model. This container can be transfered to another machine that runs Docker, and executed there without compatibility issues.
- _Version control and component reuse_ : you can track successive versions of a container, inspect differences, or roll-back to previous versions. Containers reuse components from the preceding layers, which makes them noticeably lightweight. In addition due to Docker Hub it is possible to establish a connection between Git and DockerHub. Vesion
- _Sharing_ : you can use a remote repository to share your container with others. It is also possible to configure a private repository hosted on Docker Hub.
- _Lightweight footprint and minimal overhead_ : Docker images are typically very small, which facilitates rapid delivery and reduces the time to deploy new application containers.
- _Fault isolation_ :Docker reduces effort and risk of problems with application dependencies. Docker also freezes the environment to the preferred packages version so that it guarantees continuity in deployment and isolate the container from system fails coming from package version updates.

The way to tell docker which system requirements are needed in the newly born software is a _Dockerfile_.

### Dockerfile{#dockerfile}

Docker can build images automatically by reading instructions from a Dockerfile. A Dockerfile is a text document that contains all the commands/rules a generic user could call on the CLI to assemble an image. Executing the command `docker build` from shell the user can trigger the image building. That executes sequentially several command-line instructions. For thesis purposes a dockerfile is written with the specific instructions and then the file is pushed to GitHub repository. Once pushed DockerHub automatically parses the repository looking for a plain text file whose name is "Dockerfile". When It is matched then it trriggers the building of the image.

The Dockerfile used to trigger the building of the service docker container has the following set of instructions:

![dockerfile](images/dockerfile.PNG)


- `FROM rocker/tidyverse:latest` : the command imports a pre-built image by the rocker team that contains the latest (tag latest) version of base-R along with the tidyverse packages.


- `MAINTAINER Niccolo Salvini "niccolo.salvini27@gmail.com"` : The command tags the maintainer and its e-mail contact information.


- `RUN apt-get update && apt-get install -y \ libxml2-dev \ libudunits2-dev` :The command update and install Linux dependencies needed for running R packages. `rvest` requires libxml2-dev and `magrittr` needs libudunits2-dev. If they are not installed then associated libraries can not be loaded. Linux dependencies needed have been found by trial and error while building containers. Building  logs messages print errors and suggest which dependency is mandatory.


- `RUN R -e "install.packages(c('plumber','tibble','...',dependencies=TRUE)` : the command install all the packages required to execute the files (R files) containerized for the scraping. Since all the packages have their direct R dependencies the option `dependencies=TRUE` is needed. 


- `RUN R -e "install.packages('https://cran.r-project.org/.../iterators, type='source')`
`RUN R -e "install.packages('https://cran.r-project.org/.../foreach/, type='source')`
`RUN R -e "install.packages('https://cran.r-project.org/.../doParallel, type='source')`
DoParallel was not available in package manager for R version later than 4.0.0. For this reason the choice was to install a previous source version by the online repository, as well as its dependencies.


- `COPY \\` The command tells Docker copies all the files in the container.


- `EXPOSE 8000` :  the commands instructs Docker that the container listens on the specified network ports 8000 at runtime. It is possible to specify whether the port exposed listens on UDP or TCP, the default is TCP (this part needs a previous set up previous installing, for further online documentation It is recommended [@docker_documentation_2020] )

- `ENTRYPOINT ["Rscript", "main.R"]` : the command tells docker to execute the file main.R within the container that triggers the API start. In main.R it are pecified both the port and the host where API expects to be exposed (in this case port 8000). 

In order to make the system stand-alone and make the service available to a wider range of subjects a choice has to be made. The service has to have both the characteristics to be run on demand and to specify query parameters. 

## REST API 
\BeginKnitrBlock{definition}\iffalse{-91-65-80-73-93-}\fi{}
<span class="definition" id="def:api"><strong>(\#def:api)  \iffalse (API) \fi{} </strong></span>API stands for application programming interface and it is a set of definitions and protocols for building and integrating application software. APIs let a product or a service communicate with other products and services without having to know how they’re implemented.
\EndKnitrBlock{definition}
This can simplify app development, saving time and impacting positively on the budget due to resource savings. APIs are thought of as contracts, with documentation that represents an general agreement between parties.
There are many types of API that exploit different media and architectures to communicate with apps or services.
\BeginKnitrBlock{definition}\iffalse{-91-82-69-83-84-93-}\fi{}
<span class="definition" id="def:rest"><strong>(\#def:rest)  \iffalse (REST) \fi{} </strong></span>The specification REST stands for REpresentational State Transfer and is a set of architectural principles. 
\EndKnitrBlock{definition}
When a request is made through a REST API it transfers a representation of the state to the requester. This representation, is submitted in one out of the many available formats via HTTP: JSON (Javascript Object Notation), HTML, XLT, TXT. JSON is the most popular because it is language agnostic [@what_is_a_rest_api], as well as more comfortable to be read and parsed.
In order for an API to be considered RESTful, it has to conform to these criteria:

(rivedi elenco)
- A client-server architecture made up of clients, servers, and resources, with requests managed through HTTP.
- Stateless client-server communication, meaning no client information is stored between requests and each request is separate and unconnected.
- Cacheable data that streamlines client-server interactions.
- A uniform interface between components so that information is transferred in a standard form. This requires that:
  - resources requested are identifiable and separate from the representations sent to the client.
  - resources can be manipulated by the client via the representation they receive because the representation contains enough information to do so.
  - self-descriptive messages returned to the client have enough information to describe how the client should process it.
  - hypermedia, meaning that after accessing a resource the client should be able to use hyperlinks to find all other currently available actions they can take.
- A layered system that organizes each type of server (those responsible for security, load-balancing, etc.) involved the retrieval of requested information into hierarchies, invisible to the client.

REST API accepts http requests as input and elaborates them through end points. An end point identifies the operation through traditional http methods (e.g. /GET /POST) that the API caller wants to perform. Further documentation and differences between HTTP and REST API can be found to this [reference](https://docs.aws.amazon.com/it_it/apigateway/latest/developerguide/http-api-vs-rest.html).

open REST API examples: 
- BigQuery API API: A data platform for customers to create, manage, share and query data.
- YouTube Data API v3: The YouTube Data API v3 is an API that provides access to YouTube data, such as videos, playlists, and channels.
- Cloud Natural Language API: Provides natural language understanding technologies, such as sentiment analysis, entity recognition, entity sentiment analysis, and other text annotations, to developers.
- Skyscanner Flight Search API: The Skyscanner API lets you search for flights & get flight prices from Skyscanner's database of prices, as well as get live quotes directly from ticketing agencies.
- Openweathermap API: current weather data for any location on Earth including over 200,000 cities.

![API functioning](images/Rest-API.png)

### Plumber REST API{#plumberapi}

Plumber allows the user to create a REST API by adding decoration comments to the existing R code. Decorations are a special type of comments that suggests to Plumber where and when the API specifications parts are. Below a simple example extracted by the documentation:


```r
# plumber.R

#* Echo back the input
#* @param msg The message to echo
#* @get /echo
function(msg="") {
  list(msg = paste0("The message is: '", msg, "'"))
}

#* Plot a histogram
#* @serializer png
#* @get /plot
function() {
  rand = rnorm(100)
  hist(rand)
}

#* Return the sum of two numbers
#* @param a The first number to add
#* @param b The second number to add
#* @post /sum
function(a, b) {
  as.numeric(a) + as.numeric(b)
}
```

three endpoints associated to 2 /GET and 1 /POST requests are made available. Functions are made clear without names so that whenever the endpoint is called functions are directly executed.
Decorations are marked as this `#*` and they are followed by specific keywords denoted with `@`.
- the `@params` keyword refers to parameter that specifies the corpus of the HTTP request, i.e. the inputs with respect to the expected output. If default parameters are inputted then the API response is the elaboration of the functions with default parameters. As opposite endpoint function elaborates the provided parameters and returns a response.
- `#* @serializer` specifies the extension of the output file when needed.
- `#* @get`  specifies the method of HTTP request sent.
- `/echo` is the end point name.
- `@filter` decorations activates a filter layer which are used to track logs and to parse request before passing the argbody to the end points.

Many more options are available to customize plumber API but are beyond the scope, a valuable resource for further insights can be found in the dedicated package website [@an_api_generator_for_r]

### Immobiliare.it REST API  

The API service is composed by 4 endpoints */scrape* , */links*, */complete* and */get_data*:

- */scrape performs a fast scraping of the website that leverages a shortest path to directly extract 5 covariates from url. url from which data extraction takes place might be composed through parameters. By default the end point scrape data from Milan  real estate rental market. Fast scraping is reached thanks to avoiding to access to single links. It is a superficial scraping and does not contain geospatial, however it might fit for regression settings.

- */links: extracts the list of single links belonging to each of the page, looking at section \@ref(webstructure) each 25 single links for each sibling. It displays sufficient performances in terms of run time. It is propaedeutic to apply the following endpoint. 

- */complete:  both the function all.links and complete are sourced. The former with the aim to grab each single links and store it into an object. The latter to actually iterate scraping on each of the links.

- */get_data: it triggers the data extraction by sourcing the /complete endpoint and then storing .json file into the NOSQL mongoDB ATLAS


![swagger](images/swagger.PNG)

### REST API documentation{#APIdocs}


- Get FAST data, it covers 5 covariates: title, price, num of rooms, sqmeter, primarykey
```r
      GET */scrape
      @param city [chr string] the city you are interested in (e.g. "roma", "milano", "firenze"--> lowercase, without accent)
      @param npages [positive integer] number of pages to scrape, default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param macrozone [chr string] avail: Roma, Firenze, Milano, Torino; e.g. "fiera", "centro", "bellariva", "parioli" 
      content-type: application/json 
```
- Get all the links 

```r
      GET */link
      @param city [chr string] the city you are interested to extract data (lowercase without accent)
      @param npages [positive integer] number of pages to scrape default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param .thesis [logical] data used for master thesis
      content-type: application/json 
```   
      
-  Get the complete set of covariates (52) from each single links, takes a while

```r
      GET */complete
      @param city [chr string] the city you are interested to extract data (lowercase without accent)
      @param npages [positive integer] number of pages to scrape default = 10, min  = 2, max = 300
      @param type [chr string] "affitto" = rents, "vendita"  = sell 
      @param .thesis [logical] data used for master thesis
      content-type: application/json
```


## NGINX reverse proxy server{#nginx}

For analysis purposes NGINX is open source software for reverse proxying and load balancing.
Proxying is typically used to distribute the load among several servers, seamlessly show content from different websites, or pass requests for processing to application servers over protocols other than HTTP.
[...]

When NGINX proxies a request, it sends the request to a specified proxied server, fetches the response, and sends it back to the client. It is possible to proxy requests to an HTTP server (another NGINX server or any other server) or a non-HTTP server (which can run an application developed with a specific framework, such as PHP or Python) using a specified protocol. Supported protocols include FastCGI, uwsgi, SCGI, and memcached.
[...]


.conf file and installation on Linux server. Security and Authentication. 

## AWS EC2 server{#aws}

Executing REST API on a public server allows to share data with a various number of services thorugh multitude of subjects. Since it can not be specified a-priori how many times and users are going to enjoy the service a scalable solutio might fill the needs. Scalable infrastructure through a flexible cloud provider combined with nginx load balancing can offer a stable and reliable infrastructure for a relatively cheap price.
AWS offers a wide range of services each of which for a wide range of budgets and integration. Free tier servers can be rent up to a certain amount of storage and computation that nearly 0s the total bill. The cloud provider also has a dedicated webpage to configure the service needed with respect to the usage named [amazon cost manager](https://aws.amazon.com/en/aws-cost-management/).

\BeginKnitrBlock{definition}\iffalse{-91-65-87-83-32-69-67-50-93-}\fi{}
<span class="definition" id="def:aws"><strong>(\#def:aws)  \iffalse (AWS EC2) \fi{} </strong></span>Amazon Elastic Compute Cloud (EC2) is a web service that contributes to a secure, flexible computing capacity in the AWS cloud. EC2 allows to rent as many virtual servers as needed with customized capacity, security and storage.

\EndKnitrBlock{definition}
[few words still on EC2]

### Launch an EC2 instance

The preliminary step is to pick up an AMI (Amazon Machine Image). AWS AMI are already-set-up machines with stadardized specification designed to speed up the process of choosing the a customed machine. Since the project is planned to be nearly 0-cost a “Free Tier Eligible” server is chosen. By checking the Free Tier box all the available free tiers are displayed. The machine selected has this specification: t2.micro with 1 CPU and 1GB RAM and runs on a Ubuntu distribution OS. First set up settings needs to be left as-is, networking and VPC can always be updated when needed. In the "add storage" step 30 GB storage are selected, moreover 30 represent the upper limit since the server can be considered free tier. Tags windows are beyond the scope. Secondly configuration needs to account security and a new entry below SSH connection (port 22) has to be set in. New security configuration has to have TCP specification and should be associated to port 8000. Port 8000, as in dockerfile section \@ref(dockerfile), has been exposed and needs to be linked to the security port opened. 

![aws_dashboard](images/aws.PNG)

At this point instance is prepared to run and in a few minutes is deployed. Key pairs, if never done before, are generated and .pem file is saved and securely stored. Key pairs are mandatory to access to the Ubuntu server via SSH. SSH connection in Windows OS can be handled with [PuTTY](https://www.putty.org/), which is a SSH and telnet client designed for Windows. At first PuTTYgen has to convert the key pair .pem  file into a .ppk extension (otherwise Putty can not read it). Once converted .ppk is sourced in the authorization panel. If everything works and authentication is verified then the Ubuntu server CLI appears and an interaction with the server is made available. 


## Further Integrations

Pins is an r packages [this link](https://rstudio.com/resources/rstudioconf-2020/deploying-end-to-end-data-science-with-shiny-plumber-and-pins/?mkt_tok=eyJpIjoiTmprNU1USXhPVEprWXpNMSIsInQiOiJtTUhKVzlvSjVIV2hKc0NRNVU1NTRQYSsrRGd5MWMyemlTazQ5b1lHRGJXNVBLcnpScjZRaWVcL2JGUjBPNGIwV3pwY1dKTW45cnhcL2JzZUlGWndtSFNJZVNaOUcyc1ZXcEJOcnppSVJXSGZRSVU1ZUY1YUU2NWdDamoxZG5VMHZcLyJ9)













