R advanced: programming and packaging
================
UQ Library
2019-11-22

## Download the project

We have an R project that already contains the relevant custom
functions. You can download it with this link:
<https://gitlab.com/stragu/DSH/-/archive/master/DSH-master.zip?path=R%2Fadvanced>

Unzip this archive, and open the .Rproj file.

## Our example ACORN data

Our data comes from the Bureau of Meteorology website. You can find more
information about it here:
<http://www.bom.gov.au/climate/data/acorn-sat/#tabs=Data-and-networks>

We will use a custom function to download the data.

This project provides temperature data for 112 Australian weather
stations. We are going to deal with the maximum daily temperature data,
which means we will have to deal with 112 separate CSV files.

First of all: how can we download the data?

We know the URL to the archive of CSV files, so we can integrate it into
a function to both download (with `download.file()`) and extract (with
`untar()`) the files:

``` r
get_acorn <- function(dest) {
  download.file(url = "ftp://ftp.bom.gov.au/anon/home/ncc/www/change/ACORN_SAT_daily/acorn_sat_v2_daily_tmax.tar.gz",
                destfile = "acorn_sat_v2_daily_tmax.tar.gz")
  if (!dir.exists(dest)) {
    dir.create(dest)
  }
  untar(tarfile = "acorn_sat_v2_daily_tmax.tar.gz",
        exdir = dest)
}
```

We can now call our new function to get the data:

``` r
get_acorn(dest = "acorn_sat_v2_daily_tmax")
```

Now, how do we import and clean one single station’s data?

> We will use several tidyverse packages, so we might as well load the
> core Tidyverse
    packages.

``` r
library(tidyverse)
```

    ## ── Attaching packages ────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.2.1     ✔ purrr   0.3.3
    ## ✔ tibble  2.1.3     ✔ dplyr   0.8.3
    ## ✔ tidyr   1.0.0     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.4.0

    ## ── Conflicts ───────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
read_csv("acorn_sat_v2_daily_tmax/tmax.001019.daily.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_date(format = ""),
    ##   `maximum temperature (degC)` = col_double(),
    ##   `site number` = col_character(),
    ##   `site name` = col_character()
    ## )

    ## # A tibble: 28,398 x 4
    ##    date       `maximum temperature (degC)` `site number` `site name`
    ##    <date>                            <dbl> <chr>         <chr>      
    ##  1 NA                                 NA   001019        KALUMBURU  
    ##  2 1941-09-01                         29.8 <NA>          <NA>       
    ##  3 1941-09-02                         29.8 <NA>          <NA>       
    ##  4 1941-09-03                         29.3 <NA>          <NA>       
    ##  5 1941-09-04                         37.1 <NA>          <NA>       
    ##  6 1941-09-05                         30.9 <NA>          <NA>       
    ##  7 1941-09-06                         NA   <NA>          <NA>       
    ##  8 1941-09-07                         NA   <NA>          <NA>       
    ##  9 1941-09-08                         32   <NA>          <NA>       
    ## 10 1941-09-09                         32.7 <NA>          <NA>       
    ## # … with 28,388 more rows

Looks like we need to remove the first row:

``` r
read_csv("acorn_sat_v2_daily_tmax/tmax.001019.daily.csv") %>%
  slice(-1)
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_date(format = ""),
    ##   `maximum temperature (degC)` = col_double(),
    ##   `site number` = col_character(),
    ##   `site name` = col_character()
    ## )

    ## # A tibble: 28,397 x 4
    ##    date       `maximum temperature (degC)` `site number` `site name`
    ##    <date>                            <dbl> <chr>         <chr>      
    ##  1 1941-09-01                         29.8 <NA>          <NA>       
    ##  2 1941-09-02                         29.8 <NA>          <NA>       
    ##  3 1941-09-03                         29.3 <NA>          <NA>       
    ##  4 1941-09-04                         37.1 <NA>          <NA>       
    ##  5 1941-09-05                         30.9 <NA>          <NA>       
    ##  6 1941-09-06                         NA   <NA>          <NA>       
    ##  7 1941-09-07                         NA   <NA>          <NA>       
    ##  8 1941-09-08                         32   <NA>          <NA>       
    ##  9 1941-09-09                         32.7 <NA>          <NA>       
    ## 10 1941-09-10                         33.4 <NA>          <NA>       
    ## # … with 28,387 more rows

We also want to remove the useless columns, and rename the maximum
temperature variable:

``` r
read_csv("acorn_sat_v2_daily_tmax/tmax.001019.daily.csv") %>%
  slice(-1) %>% 
  select(date, max.temp = `maximum temperature (degC)`)
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_date(format = ""),
    ##   `maximum temperature (degC)` = col_double(),
    ##   `site number` = col_character(),
    ##   `site name` = col_character()
    ## )

    ## # A tibble: 28,397 x 2
    ##    date       max.temp
    ##    <date>        <dbl>
    ##  1 1941-09-01     29.8
    ##  2 1941-09-02     29.8
    ##  3 1941-09-03     29.3
    ##  4 1941-09-04     37.1
    ##  5 1941-09-05     30.9
    ##  6 1941-09-06     NA  
    ##  7 1941-09-07     NA  
    ##  8 1941-09-08     32  
    ##  9 1941-09-09     32.7
    ## 10 1941-09-10     33.4
    ## # … with 28,387 more rows

This is probably what we want to end up with.

## Creating a new custom function

Let’s make this import/clean pipeline into a function, so we can reuse
it easily later on.

We have to use `function()` to define it, and also provide which
arguments can be used with this function (in our case, only the path to
the file we want to process):

``` r
read_station <- function(file) {
  read_csv(file) %>% 
    slice(-1) %>% 
    select(date, max.temp = `maximum temperature (degC)`)
}
```

We also want to add an extra step to store the file name, so this
information is available when we merge all the datasets. We can do that
with the `mutate()` function:

``` r
read_station <- function(file) {
  read_csv(file) %>% 
    slice(-1) %>% 
    select(date, max.temp = `maximum temperature (degC)`) %>% 
    mutate(filename = file)
}
```

Functions will by default return the last evaluated element.

We can now test it on our first file:

``` r
read_station("acorn_sat_v2_daily_tmax/tmax.002012.daily.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_date(format = ""),
    ##   `maximum temperature (degC)` = col_double(),
    ##   `site number` = col_character(),
    ##   `site name` = col_character()
    ## )

    ## # A tibble: 39,811 x 3
    ##    date       max.temp filename                                     
    ##    <date>        <dbl> <chr>                                        
    ##  1 1910-01-01     40.5 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  2 1910-01-02     40.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  3 1910-01-03     40.5 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  4 1910-01-04     36.2 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  5 1910-01-05     40.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  6 1910-01-06     36.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  7 1910-01-07     40.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  8 1910-01-08     36.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ##  9 1910-01-09     37.6 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ## 10 1910-01-10     36.7 acorn_sat_v2_daily_tmax/tmax.002012.daily.csv
    ## # … with 39,801 more rows

We now want to iterate over all the files, and create a single merged
dataframe.

We can start with finding all the relevant files:

``` r
files <- list.files(path = "acorn_sat_v2_daily_tmax",
           pattern = "tmax*",
           full.names = TRUE)
```

We can then apply our custom function iteratively to each file. For
that, purrr’s `map_` function family is very useful. Because we want to
end up with a dataframe, we will use `map_dfr()`:

We now have close to 4 million rows in our final dataframe.

We might also want to create a function from the two previous steps, so
we only have to provide the name of the directory where the files are
located:

``` r
merge_acorn <- function(dir) {
  files <- list.files(path = dir,
           pattern = "tmax*",
           full.names = TRUE)
  map_dfr(files, read_station)
}
```

Let’s try our new function:

## Making functions more resilient

We can make our functions more resilient by adding `stop()` and
`warning()` calls.

For example, what if our `merge_acorn()` function is not provided with a
valid path?

``` r
merge_acorn("blah")
```

We could improve our function with the following:

``` r
merge_acorn <- function(dir) {
  if (!dir.exists(dir)) {
    stop("the directory does not exist. Please provide a valid path as a string.")
  }
  files <- list.files(path = dir,
           pattern = "tmax*",
           full.names = TRUE)
  map_dfr(files, read_station)
}
```

Now, let’s see what happens if we don’t provide a valid directory:

``` r
merge_acorn("bleh")
```

This will neatly stop our function and provide an error message if the
path is not found.

We now want to share our useful functions with the World\!

## Packaging functions

> Useful packages for package development: devtools, usethis, roxygen2

To prepare for packaging our functions, we can create a different script
for each function. The name of each script can be the name of the
function it defines: “get\_acorn.R”, “read\_station.R” and
“merge\_acorn.R”.

> It is often useful to define functions in a separate script to the
> data analysis script. Know that you can then “source” those custom
> functions at the beginning of the analysis script thanks to the
> `source()` function.

Now, let’s create a new project for our package, to keep things tidy:
File \> New Project… \> New Directory \> R Package.

Let’s name our package “acornucopia”.

We can pick the two function scripts we created before as the base for
our package.

We end up with a basic package structure:

  - DESCRIPTION
  - man
  - NAMESPACE
  - R

Let’s go through those main components.

### 1\. Description

This is the general description of the what the package does, who
developed and maintains it, what it needs to work, and what licence it
is released under.

The file already contains a template for us to fill in. We can update
the fields with relevant information, in particular for `Title`,
`Author`, `Maintainer`, `Description` and `License`:

    Package: acornucopia
    Type: Package
    Title: Process ACORN data
    Version: 0.1.0
    Author: Your Name
    Maintainer: Your Name <yourself@somewhere.net>
    Depends: readr, dplyr, purrr
    Description: Functions to import, cleanup and merge temperature data from ACORN stations
    License: GPL-3
    Encoding: UTF-8
    LazyData: true

Notice that we added the `Depends:` field. This allows us to specify
what extra packages are needed for our package to work.

#### Which licence?

GPL or MIT are common licences for R packages. They are Open Source and
allow others to reuse your code, which is the norm in the R ecosystem.

### 2\. NAMESPACE

The NAMESPACE file lists the functions available to the user when the
package is loaded with `library()`.

By default, it uses a regular expression that will match anything with a
name that starts with one or more letters (in the R directory).

### 3\. R

This is where our function definitions go. If you haven’t imported the
two scripts when creating the package project, copy and paste them in
now.

#### Documenting functions

We are using the package roxygen2 to document each function.

With a function’s R script open in the source pane, go to Code \> Insert
Roxygen Skeleton. This will add a template before your function, which
can be used to generate the function’s documentation. You can see that
it is adapted to your code. For example, for the `read_station()`
function:

    #' Read a station's CSV file
    #'
    #' This function imports and cleans a single ACORN station CSV file.
    #'
    #' @param file Path and filename of CSV file to read (as a string)
    #'
    #' @return A clean tibble
    #' @export
    #' @import dplyr
    #' @importFrom readr read_csv
    #' @examples
    #' \dontrun{
    #' read_station("path/to/file.csv")
    #' }

  - up the top, the first sentence is the title, and the following
    paragraph gives a description. A third section can be used to give
    more details.
  - `@export` can be used as is to populate the NAMESPACE file with this
    function
  - we added the `@import` and `@importFrom` tags to specify precisely
    what package or functions need to be imported for our function to
    work.
  - `\dontrun{}` can be used for examples that will not work, or that
    shouldn’t be executed for a variety of reasons.

> Pressing return inside the Roxygen skeleton will automatically prepend
> the necessary comment characters `#'`

We can then generate and view the help file with:

``` r
devtools::document()
?read_station
```

Notice that we get a warning about NAMESPACE not being generated by
roxygen2. If we want roxygen2 to take care of this file, we can first
delete it, and then run the `document()` function again.

NAMESPACE is now populated with the exports and imports defined in the
Roxygen documentation.

#### Challenge 1: document a function

Use roxygen2 to create the documentation for `merge_acorn()`. Try
generating the documentation again, and check your NAMESPACE.

### Testing the package

At any time, we can load the package to test its functions:

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>L</kbd>

To check our package for issues:

``` r
devtools::check()
```

This function does a very thorough check, and will take some time to go
through all the steps.

### Building and installing

We can also install our package on our system. First, we need to build
the package:

  - On Windows: Build \> Build Binary Package
  - On Linux or macOS: Build \> Build Source Package

This is what you can share with colleagues and friends who want to try
your package on their computer.

To install it (the name of the archive will depend on your system):

``` r
install.packages("../acornucopia_0.1.0.tar.gz", repos = NULL)
```

> We have to set `repos` to `NULL` so R doesn’t look for the package on
> CRAN.

We now have our package listed in our packages list, ready to be loaded
whenever we want to use its functions.

### Publishing

We can then try to publish on CRAN, or other repositories, making sure
that you follow the guidelines / requirements.

### Best practice

As a general rule, it is best to stick to a minimum of dependencies, so
the package:

  - requires less maintenance
  - is lighter to install

For function names, try to avoid dots, and use underscores instead
(tidystyle).

In order to publish on CRAN, we have to follow more stringet policies:
<https://cran.r-project.org/web/packages/policies.html>

### Using version control

When developing software, it is important to keep track of versions, and
if you collaborate with others, of authors too.

It also allows you to roll back to a previous version if needed.

RStudio integrates Git, the most popular version control system for
software development. To setup a Git repository for our package, we can
use “Tools \> Version Control \> Project Setup…”, and create a new Git
repository. We can then use the Git tab to save snapshots of our files.

You can then host your code on Gitlab or GitHub to make it accessible to
others. See for example: <https://github.com/Lchiffon/wordcloud2>

The package usethis provides many useful functions to setup packages,
including a `use_github()` function to quickly create a remote
repository, and a `use_readme_*()` to add a readme file for the project.

Others will then be able to install your package from your remote
repository with:

``` r
devtools::install_github("username/myPackage")
devtools::install_gitlab("username/myPackage")
```

## Shiny webapps

Shiny is a package that allows to create a web application with R code.

A Shiny app requires two main elements:

  - a user interface (UI)
  - a server

Let’s build an app from scratch, using our ACORN data and functions.

What we want to create is a small webapp that visualises the ACORN data
and gives the user a bit of control over the visualisation.

### Setting up

In our original project (that contains the ACORN data), let’s create a
new app with “File \> New File \> Shiny Web App…”. We will stick to
“single file”, and the current project directory as the location.

In our files, we can now see a “myApp” directory that contains an
“app.R” script.

The app is currently an example app. We can run it with the “Run App”
button.

#### Creating a minimal skeleton

For our app to work, we need to:

  - define a UI
  - define a server
  - define how the app is run

We can start with this empty skeleton:

``` r
# UI
ui <- fluidPage()

# Server
server <- function(input, output) {}

# Run the application 
shinyApp(ui = ui, server = server)
```

Running it will show a blank page. Let’s add a title:

``` r
# UI
ui <- fluidPage(
  titlePanel("ACORN data explorer")
)

# Server
server <- function(input, output) {}

# Run the application 
shinyApp(ui = ui, server = server)
```

Now, let’s make sure we have the data ready to be used in our app. We
don’t want to do the merging and summarising of our data every time we
run the app, so let’s save the finished product into an RDS file. In our
process.R script:

``` r
library(acornucopia)
dat <- merge_acorn("acorn_sat_v2_daily_tmax")
# process for monthly average
library(lubridate)
monthly <- dat %>% 
    group_by(month = month(date),
             year = year(date)) %>% 
    summarise(mean.max = mean(max.temp, na.rm = TRUE))
# export into app directory
saveRDS(monthly, "myApp/monthly.RDS")
```

We can now read that data file into our app, process it, and present it
in an interactive table:

``` r
# Import data
monthly <- readRDS("monthly.RDS")

# Define UI
ui <- fluidPage(
    titlePanel("ACORN data explorer"),
    dataTableOutput("dt")
)

# Define server logic
server <- function(input, output) {
    output$dt <- renderDataTable({
        monthly
    })
}

# Run the application 
shinyApp(ui = ui, server = server)
```

Notice that we had to define an output in the server section (with a
“render” function), and use that output in a UI function (with an
“output” function).

Now, for a different kind of output, let’s add a plot:

``` r
# Import data
monthly <- readRDS("monthly.RDS")

# Load necessary packages
library(ggplot2)

# Define UI
ui <- fluidPage(
    titlePanel("ACORN data explorer"),
    plotOutput("plot"),
    dataTableOutput("dt")
)

# Define server logic
server <- function(input, output) {
    output$dt <- renderDataTable({
        monthly
    })
    
    output$plot <- renderPlot({
            ggplot(monthly,
               aes(x = year, y = month, fill = mean.max)) +
            geom_tile() +
            scale_fill_distiller(palette = "RdYlBu")
    })
}

# Run the application 
shinyApp(ui = ui, server = server)
```

How can we add some interaction? We could give the user control over
which month they want to visualise by adding a slider:

``` r
# Import data
monthly <- readRDS("monthly.RDS")

# Load necessary packages
library(ggplot2)
library(dplyr)

# Define UI
ui <- fluidPage(
    titlePanel("ACORN data explorer"),
    # input slider for months
    sliderInput("month",
                "Pick a month:",
                min = 1,
                max = 12,
                value = 1),
    plotOutput("plot"),
    dataTableOutput("dt")
)

# Define server logic
server <- function(input, output) {
    output$dt <- renderDataTable({
        monthly
    })
    
    output$plot <- renderPlot({
        monthly %>% 
            filter(month == input$month) %>% 
            ggplot(aes(x = year, y = month, fill = mean.max)) +
            geom_tile() +
            scale_fill_distiller(palette = "RdYlBu")
    })
}

# Run the application 
shinyApp(ui = ui, server = server)
```

## Challenge 2: restore an “all months” option?

How could we give the option to go back to the full-year view?

Hint: have a look at `?selectInput`, or find other ideas on this list:
<https://shiny.rstudio.com/tutorial/written-tutorial/lesson3/>

One solution could be:

``` r
# import data
monthly <- readRDS("monthly.RDS")

# load necessary packages
library(ggplot2)
library(dplyr)

# Define UI for application that draws a histogram
ui <- fluidPage(
    titlePanel("ACORN data explorer"),
    # input slider for months
    selectInput("month",
                "Pick one or more months:",
                1:12,
                multiple = TRUE),
    plotOutput("plot"),
    dataTableOutput("dt")
)

# Define server logic required to draw a histogram
server <- function(input, output) {
    output$dt <- renderDataTable({
        monthly
    })
    
    output$plot <- renderPlot({
        monthly %>% 
            filter(month %in% input$month) %>% 
            ggplot(aes(x = year, y = month, fill = mean.max)) +
            geom_tile() +
            scale_fill_distiller(palette = "RdYlBu")
    })
}

# Run the application 
shinyApp(ui = ui, server = server)
```

## Publishing a Shiny app

You can use ShinyApps.io, which offers free or paid accounts.

We also have access to Nectar (National eResearch Collaboration Tools
and Resources project), in which we can request a virtual machine and
deploy a Shiny server: <https://nectar.org.au/>

## Useful links

  - *R Packages*, by Hadley Wickham: <http://r-pkgs.had.co.nz/>
  - Full official guide for packaging:
    <https://cran.r-project.org/doc/manuals/r-release/R-exts.html>
  - What to lookout for when publishing to CRAN:
    <https://cran.r-project.org/web/packages/policies.html>
  - Package development cheatsheet:
    <https://github.com/rstudio/cheatsheets/raw/master/package-development.pdf>
  - Official Shiny tutorial: <https://shiny.rstudio.com/tutorial/>
  - Shiny examples:
      - <https://shiny.rstudio.com/gallery/>
      - <https://www.showmeshiny.com/>
  - Shiny cheatsheet:
    <https://github.com/rstudio/cheatsheets/raw/master/shiny.pdf>

## Extras

These two topics are important when developing custom functions, but can
not fit in this session. They are described here for reference, if
needed.

### Quasiquotation

Tidyverse packages use **quasiquotation** and **lazy evaluation** to
save us a lot of typing.

For example, we can do:

``` r
mtcars %>% select(disp)
```

… but `disp` is not an existing object in our environment.

`quo_name()` quotes, whereas `sym()` gets the symbol.

Try `eval(sym(something))`.

This might lead to issues, like:

``` r
selector <- function(d, col) {
  d %>% select(col)
}
```

We need to quote-unquote:

``` r
selector <- function(d, col) {
  col <- enquo(col) # do not evaluate yet!
  d %>% select(!!col) # evaluate here only
}
```

For multiple arguments:

``` r
selector <- function(d, ...) {
  col <- enquos(...)
  d %>% select(!!!col)
}
```

### Classes

We can assign new classes to objects:

``` r
obj <- "Hello world!"
class(obj) <- "coolSentence"
attributes(obj)
```

The `structure()` function is useful for that too.

We can then define methods for this specific class, for example a
`coolSentence.print()` function.