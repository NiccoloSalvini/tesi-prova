# Exploratory Analysis {#exploratory}

<!--  You can label chapter and section titles using `{#label}` after them, e.g., we can reference Chapter \@ref(intro). If you do not manually label them, there will be automatic labels anyway, e.g., Chapter \@ref(methods).-->



Data comes packed into the REST API end point `*/complete` in .json format. Data can be filtered out On the basis of the options set in the API endpoint argument body. Some of the options might regard the `city` in which it is aevaluated the real estate market, `npages` as the number of pages to be scraped, `type` as the choice between rental of selling. For further documentation on how to structure the API endpoint query refer to section \@ref(APIdocs).  Since to the analysis purposes data should come from the same source (e.g. Milan rental real estate within "circonvallazione") a dedicated endpoint boolean option `.thesis` is passed in the argument body. What the API option under the hood is doing is specifying a structured and already filtered URL to be passed to the scraping endppoint. By securing the same URL to the scraping functions data is forced to come from the same URL source. The idea behind this concept can be thought as refreshing everyday the same immobiliare.it URL. API endpoint by default also specifies 10 pages to be scraped, in this case 120 is provided leading to to 3000 data points. The `*` refers to the EC2 public DNS that is `ec2-18-224-40-67.us-east-2.compute.amazonaws.com`

`http://*/complete/120/milano/affitto/.thesis=true`

As a further source data can also be accessed through the mondgoDB credentials with the cloud ATLAS database by picking up the latest .csv file generated. For run time reasons also related to the bookdown files continuous building the API endpoint is called the day before the presentation so that the latest .csv file is available. As a consequence code chunks outputs are all cached due to heavy computation.
Interactive  maps are done with Leaflet, the result of which is a leaflet map object which can be piped to other leaflet functions. This permits multiple map layers and many control options to be added interactively (LaTex output is statically generated)

A preliminary exploratory analysis evidences 34 covariates and 250 rows. Immobiliare.it furnishes many information regarding property attributes and estate agency circumstances. Data displays many NA in some of the columns but georeference coordinates, due to the design of scraping functions, are in any case present. 


\begin{longtable}{ll}
\toprule
name & ref\\
\midrule
ID & ID of the apartements\\
LAT & latitude coordinate\\
LONG & longitude coordinate\\
LOCATION & the complete address: street name and number\\
CONDOM & the condominium monthly expenses\\
\addlinespace
BUILDAGE & the age in which the building was contructed\\
FLOOR & the floor the apartement is\\
INDIVSAPT & indipendent propeorty versus apartement\\
LOCALI & specification of the type and number of rooms\\
TPPROP & property type residential or not\\
\addlinespace
STATUS & the actual status of the house, ristrutturato, nuovo, abitabile\\
HEATING & the heating system centralized or autonomous\\
AC & air conditioning hot and cold\\
PUB\_DATE & the date of publication of the advertisement\\
CATASTINFO & land registry infromation\\
\addlinespace
APTCHAR & apartement main characteristics\\
PHOTOSNUM & number of photos displayes\\
AGE & estate agency name\\
LOWRDPRICE.originalPrice & If the price is lowered it indicates the starting price\\
LOWRDPRICE.currentPrice & If the price is lowered it indicates the current price\\
\addlinespace
LOWRDPRICE.passedDays & If the price is lowered the days passed since the price has changed\\
LOWRDPRICE.date & If the price is lowered the date passed since the price has changed\\
ENCLASS & the energy class communicated to the\\
CONTR & the type of contract\\
DISP & if it is still avaiable or not\\
\addlinespace
TOTPIANI & the total number of floors\\
PAUTO & number of parking box or garages avaibable in the property\\
REVIEW & estate agency review, long chr string\\
HASMULTI & it if has multimedia option, such as 3D house vitualization home experience or videos\\
PRICE & the monthly price <- response\\
\bottomrule
\end{longtable}

Geographic coordinates can be represented on a map in order to get a first perception of spatial autocorrelations clusters.
Leaflet maps are created with leaflet(), the result of which is a leaflet map object which can be piped to other leaflet functions. This allows multiple map layers and control settings to be added interactivelyleaflet object takes as input data in latitude and longitude UTM coordinates so no transfomation is required. Otherwise a projection to the right zone would be required and the a sp transform 


\includegraphics[width=1\linewidth]{images/leaflet_prezzi} 


##  Data preparation 

As already pointed out some data went missing since immobiliare provides data that in turn is filled by estate agencies or privates through pre compiled standard formats. Some of the missing observations can be reverse engineered by other information in the web pages e.g. given the street address it is possible to trace back the lat and long coordinates, even though this is already handled by the API through scraping functions with the mechanism found in section \@ref(ContentArchitecture). Some of the information lacking in the summary table they might be desumed and then imputed by the estate agency review which is one of the covariates and where the most of the times missing information can be found. The approach followed in this part is to prune redundant information and "over" missing covariates trying to limit the dimensionality of the dataset.


### NA Removal and Imputation 

The first problem to assess is why information are missing. As already pointed out in the introduction part and in section \@ref(ContentArchitecture) many of the presumably important covariates (i.e. price lat, long, title ,id ...) undergo to a sequence of forced step inside scraping functions with the aim to avoid to be missed.  If in the end of the sequence the covariates values are still missing, the correspondent observation is not imputed and it is left out of the scraped dataset. The author choice to follow this approach relies on empirical observation that suggest when important inframtion is missing then the rest of the covariates also do, as a consequence the observation is not useful. Moreover when the spatial component can no be reverse engineered then it is also the case of left out observation while scraping. The model needs to have a spatial component in order to be evaluted. To this purpose The missing profile is crucial since it can also suggest problems in the scraping procedure. By identifying pattern in missing observation the maintainer can take advanatge of it and then debug the part that causes missingness. In order to identify the pattern a revision of missigness is introduced by _Little and Rubin (2014) miss lit_ .randomnes can be devided into 3 categories:

- MCAR (missing completely at random) likelihood of missing is equal for all the infromation, in other words missing data are one idependetn for the other.
- MAR (missing at random) likelihood of missin is not equal.
- *NMAR* (not missing at random) data that is missing due to a specific cause, scarping can be the cause.

The last iphothesis MNAR is often the case of daily monitoring clinical studies, where patient might drop out the experiment because of death and so as a consequence all the data starting from the death time +1 is missing.
To assess pattern A _heat map_ plot fits the need: 

![(\#fig:Heatmap)Missingness Heatmap plot](07-exploratory_files/figure-latex/Heatmap-1.pdf) 

Looking at the left hand of the heat map plot \@ref(fig:Heatmap) considering from *TOTPIANI* to *AC* there are no relevant patterns and missingness can be traced back to MAR, conditioned mean imputation is applied until *CONDOM* included, the otherS are discarded. In the far right hand side *ENCLASS* and *DISP* data is completely missing, this can be addressed to a scraping fail. Further inspection of the API scraping process focused on those covariates is strongly advised. From *LOWRDPRICE.* covariates class it seems to be witnessing a missing underlining pattern NMAR which is clearer by looking are the second co_occurrence plot \@ref(fig:cooccurrence) analysis. Co-occurrence analysis might suggest frequency of missing predictor combinations and *LOWRDPRICE.* class covariates are missing all together in combination. *PAUTO* is missing where lowered price class covariates are missing but this is not true for all the observations leading to the conclusion that *PAUTO* should be treated as a low prevalence covariate, therefore PAUTO is discarded.
After a further investigation *LOWRDPRICE.* exists when the price covariates is effectively decreased, that leads to group the covariate's information and to encode it as a two levels categorical covariate. Further methods to feature engineer the lowrdprice covariates are profile data. 

![(\#fig:cooccurrence)Missingness co-occurrence plot](07-exploratory_files/figure-latex/cooccurrence-1.pdf) 


## Spatial Autocorrelation assessement 




Spatial data come packed into point reference 

- tmap 
- leaflet 
- gganimate 



## Model Specification



## Mesh building 

*PARAFRASARE*
The SPDE approach approximates the continuous Gaussian field $w_{i}$ as a discrete Gaussian Markov random field by means of a finite basis function defined on a triangulated mesh of the region of study. The spatial surface can be interpolated performing this approximation with the inla.mesh.2d() function of the R-INLA package. This function creates a Constrained Refined Delaunay Triangulation (CRDT) over the study region, that will be simply referred to as the mesh. Mesh should be intended as a trade off between the accuracy of the GMRF surface representation and the computational cost, in other words the more are the vertices, the finer is the GF approximation, leading to a computational funnel. 

![Traingularization intuition, @Krainski-Rubio source](images/triangle.jpg)

Arguments can tune triangularization through inla.mesh.2d() :

* `loc`:location coordinates that are used as initial mesh vertices
* `boundary`:object describing the boundary of the domain,
* `offset`:  argument is a numeric value (or a length two vector) and it is used
to set the automatic extension distance. If positive, it is the extension distance
in the same scale units. If negative, it is interpreted as a factor relative to the
approximate data diameter; i.e., a value of -0.10 (the default) will add a 10%
of the data diameter as outer extension.
* `cutoff`: points at a closer distance than the supplied value are replaced by a single vertex. Hence, it avoids small triangles 
* `max.edge`: A good mesh needs to have triangles as regular as possible in size and shape.
* `min.angle`argument (which can be scalar or length two vector) can be used to specify the minimum internal angles of the triangles in the inner domain and the outer extension

A convex hull is a polygon of triangles out of the domain area, in other words the extension made to avoid the boundary effect. All meshes in Figure 2.12 have been made to have a convex hull boundary. If borders are available are generally preferred, so non convex hull meshes are avoided.



### Shinyapp for mesh assessment

INLA includes a Shiny (Chang et al., 2018) application that can be used to tune the mesh params interactively




The mesh builder has a number of options to define the mesh on the left side. These include options to be passed to functions inla.nonconvex.hull() and inla.mesh.2d() and the resulting mesh displayed on the right part.

### BUilding SPDE model on mesh




## Spatial Kriging (Prediction)

QUI INCERTEZZE








