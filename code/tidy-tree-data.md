tidy tree data
================
------------------------------------------------------------------------
4/15/2018

``` r
# read data
trees <- read.csv("~/emergency-response-time/data-raw/tree-data-13april.csv")
```

``` r
trees <- trees %>%
  select(X,Y,Year_Plant,Family,Common_nam,Genus_spec,Functional,
         Size,Native,Edible,Nuisance,Origin,Planting_O)
```

``` r
# transform data
# make origin regex to pull out continents
origin_pat <- "^[A-Z][a-z]+ ?[A]?[a-z]+"
x <-grepl(origin_pat,trees$Origin)
# remove bad origin rows, make better origin variable
trees <- trees %>%
  filter(x) %>%
  mutate(origin_wip = grep(origin_pat,Origin, value=T) %>% str_extract(origin_pat)) %>%
  mutate(origin = ifelse(origin_wip=="China","Asia",origin_wip)) %>%
  select(-Origin,-origin_wip)

trees <- trees %>%
# filter out trees with no native val, make better "native" col
  filter(Native %in% c("Yes","No")) %>%
  mutate(native=ifelse(Native=="Yes",T,F)) %>%
  select(-Native) %>%
# mutate better "group" col
  mutate(group = ifelse(Planting_O=="BES Contractor", "contractor",
                        ifelse(Planting_O=="FoTStreet","street",
                               ifelse(Planting_O=="FoTYard","yard","park")))) %>%
  select(-Planting_O) %>%
# mutate better "nuisance" col
  mutate(nuisance=ifelse(Nuisance!="",T,F)) %>%
  select(-Nuisance) %>% 
# mutate better "edible" col
  mutate(edible=ifelse(Edible=="Yes - fruit","fruit",
         ifelse(Edible=="Yes - nuts","nuts","none"))) %>%
  select(-Edible) %>%
# mutate radius, area for canopy cover
  mutate(canopy_rad=ifelse(Size=="L", 60,
                           ifelse(Size=="M", 40, 20))) %>%
  mutate(canopy_area = pi*canopy_rad^2)

# reorder and rename cols
trees <- trees %>%
  select(X,Y,Year_Plant,Common_nam,Size,canopy_rad,canopy_area,native,
         nuisance,edible,Family,Genus_spec,origin,Functional,group)
names(trees) <- c("xcoord","ycoord","year","name","size","canopy_rad","canopy_area",
                  "native","nuisance","edible","family","species","origin",
                  "funct","group")
```

``` r
# export new tidy as csv
setwd("~/emergency-response-time/data")
write.csv(trees, "tree_tidy.csv",row.names=F)
```