### Change in winter cereals declared area in 2023-2024
Target was to learn read and work with TopoJSON files. Filter relevant data, generate my own tables and make choropleth map from filtered statistics.

Shape maps were made based on the following statistics:

https://app.powerbi.com/view?r=eyJrIjoiYWQ1YjQ3ZGMtOGQxNi00MGI1LTliMzAtZjhkY2YwYTY2NzliIiwidCI6IjNjMjk2MzFmLTAyN2EtNGFlYy05OGQxLWJlMGNjODg4MzAxNiIsImMiOjl9

### Paveiksliukas 1

### Step 1: 
Imported necessary libraries.

### Step 2: 
Imported data array about declared area of plants by municipality, plants classifier and information about municipalities(coordinates).  

    kodu_plotas_SAV <- read_excel("N:/ProjektasR/deklaruotos_naudmenos_pagal_SAV_2023_2024.xlsx", col_names =TRUE)
    paseliu_kl <- read_excel("N:/ProjektasR/paseliu_klasifikatorius.xlsx", col_names = TRUE)
    koord <- read_excel("N:/ProjektasR/SAVIVALDYBIU_kord.xlsx", col_names = TRUE)

### Step 3: 

Deleted unecessary column in plants classifier.

    paseliai <- subset(paseliu_kl, select = -c(GALIOJA_IKI))

### Step 4: 

Merged both tables by plant ID.

    plotas_merged_SAV <- kodu_plotas_SAV %>% 
    left_join(paseliai, by = c("KULTUROSID" = "ID"))

### Step 5: 

Filtered data about 2024 and 2023 separately.

    plotas2024_SAV <- plotas_merged_SAV %>% filter(METAI == 2024)
    plotas2023_SAV <- plotas_merged_SAV %>% filter(METAI == 2023)

### Step 6:

Created variable to filter relevant data about winter plants.

    zieminiai <- c("KVŽ", "KRŽ", "MIŽ", "RUŽ")

### Step 7:

Filtered data about winter plants only.

    kulturos_ziem24 <- plotas2024_SAV %>% filter(KODAS %in% zieminiai)
    kulturos_ziem23 <- plotas2023_SAV %>% filter(KODAS %in% zieminiai)

### Step 8:

Grouped by both tables by name of municipality and summarised declared area.

    groupby2024 <- kulturos_ziem24 %>% group_by(SAV_PAVADINIMAS, METAI) %>% 
    summarise(sumplotas = sum(PLOTAS), .groups = 'drop') %>% 
    as.data.frame()

    groupby2023 <- kulturos_ziem23 %>% group_by(SAV_PAVADINIMAS, METAI) %>% 
    summarise(sumplotas = sum(PLOTAS), .groups = 'drop') %>% 
    as.data.frame()

### Step 9:

Merged both tables.

    pokytis <- groupby2024 %>% left_join(groupby2023, by = c("SAV_PAVADINIMAS" = "SAV_PAVADINIMAS"))

### Step 10: 

Created new column with a change of declared area in 2024 compared to 2023.

    pokytis$pokytis <- (pokytis$sumplotas.x - pokytis$sumplotas.y)

### Paveiksliukas 2

### Step 11:

 Merged table with table regarding information about municipalities.

    pokytis2 <- pokytis %>% left_join(koord, by = c("SAV_PAVADINIMAS" = "SAV_PAVADINIMAS"))


### Step 12: 

Read topoJSON files about Lithuania‘s municipalities:

    j <- sf::st_read("N:/GEOJSON IR TOPOJSON/Savivaldybes_TOPOJSON.json")
    j <- topojson_read("N:/GEOJSON IR TOPOJSON/Savivaldybes_TOPOJSON.json")

### Step 13: 

Set coordinate system to EPSG 4326

    st_crs(j) <- 4326
    st_set_crs(j, 4326)

### Step 14: 

Merged topoJSON file with information about change and municipalities.

    merged <- j %>% left_join(pokytis2, by = c("SAV_PAV" = "SAV_PAVADINIMAS"))

## ggplot2()

### Step 15: 

Using ggplot2 created choropleth map with scale fill, regarding change of winter plants.

    ggplot() +
        geom_sf(aes(fill=merged$pokytis), j, show.legend =  FALSE) +
        scale_fill_gradient( low = "blue", high = "#115a9e")
### Paveiksliukas 3

### Step 16: 

Created choropleth map with labels.

    ggplot() +
        geom_sf(aes(fill=merged$pokytis), j, show.legend = FALSE) +
        geom_label(aes(x=merged$Long, y=merged$Lat,label=merged$SAV_PAV, color = "red"), label.size = NA, size = 3, show.legend = FALSE)+
        scale_fill_gradient( low = "blue", high = "#115a9e")

### Paveiksliukas 4


## Insights:

### Raw data about declared area of all plants were taken. Information about winter plants were taken, summarised declared area in 2023 and 2024 and then change measured.

### Using topoJSON file choropleth map was created. Scale fill was used to provide information about declared area change of winter plants in 2023 and 2024.


