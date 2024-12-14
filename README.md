## Declared crops by groups in 2024.

Target was to learn read and work with excel files. Filter relevant data, generate my own tables and visualise prepared statistics.

Project was made based on the following statistics:

https://app.powerbi.com/view?r=eyJrIjoiMTI5NWVjNzUtMWJhYi00MzBlLThhMTUtYTVjNTUzY2M4ZWQyIiwidCI6IjNjMjk2MzFmLTAyN2EtNGFlYy05OGQxLWJlMGNjODg4MzAxNiIsImMiOjl9

![R1_1](https://github.com/user-attachments/assets/c5186d95-70a7-4f6a-b279-0d60665264d0)

Note. Numbers are a bit different, since I‘ve used current date data and officialy we are providing statistics exactly after declaration.

### Step 1: 
Imported necessary libraries.

### Step 2: 
Imported data array about declared area of plants and plants classifier. 

    kodu_plotas <- read_excel("C:/R/deklaruotos_naudmenos_2022_2024.xlsx", col_names =TRUE)

    paseliu_kl <- read_excel("C:/R/paseliu_klasifikatorius.xlsx", col_names = TRUE)
    
![R1_2](https://github.com/user-attachments/assets/1a846539-2872-413a-a571-37f76415c44b)


### Step 3: 
Since I am going to use IDs to connect these two tables, deleted unecessary column about validation of plant.

    paseliai <- subset(paseliu_kl, select = -c(GALIOJA_IKI))

### Step 4:
Merged both tables by plant ID.

    plotas_merged <- kodu_plotas %>% 
    left_join(paseliai, by = c("KULTUROSID" = "ID"))

### Step 5: 
Filtered data about year 2024 and 2023 in different tables, so I could count change in declared area.

    plotas2024 <- plotas_merged %>% filter(METAI == 2024)

    plotas2023 <- plotas_merged %>% filter(METAI == 2023)


![R1_3](https://github.com/user-attachments/assets/f99823c2-b137-45f8-95d9-230ab5d84627)


### Step 6:

 Created variable with relevant plant codes.

    kulturos <- c("KVŽ", "KRŽ", "MIŽ", "RUŽ", "MIV", "KVV", "AVI", "KUK", "GRI", "KRV", 
    "RUV", "RAŽ", "CUR", "RAV", "KAN", "LIN", "ŽIR", "PUP", "LUB", "VIK")


### Step 7: 
Created variables for each group of plants.

    zieminiai <- c("KVŽ", "KRŽ", "MIŽ", "RUŽ")
    vasariniai <- c("MIV", "KVV", "AVI", "KUK", "GRI", "KRV", "RUV")
    techniniai <- c("RAŽ", "CUR", "RAV", "KAN", "LIN")
    ankstiniai <- c("ŽIR", "PUP", "LUB", "VIK")

### Step 8:
Filtered information about required information with created variable kulturos. 

    Mehtod 1: kulturos_1 <- plotas2024 %>% filter(KODAS %in% kulturos)

    Method 2: kulturos_2 <- filter(plotas2024, KODAS %in% kulturos)


### Step 9: 
Created my own table with plant groups and assigned number fore ach group.

    paseliu_grupes <- data.frame(pavadinimas = c("Žieminiai javai","Vasariniai javai","Techniniai augalai","Ankštiniai augalai"),
                             grupe = c(1,2,3,4))

### Step 10: 
Assigned number of group to each plant code with created variables from before.

    kulturos_grupes <- kulturos_1 %>% mutate(grupes_nr = case_when(KODAS %in% zieminiai ~ 1,
                                                               KODAS %in% vasariniai ~ 2,
                                                               KODAS %in% techniniai ~ 3,
                                                               KODAS %in% ankstiniai ~ 4,
                                                               TRUE ~ 5 ))

### Step 11:
 Merged two tables kulturos_grupes and paseliu_grupes and assigned group number to each plant code.

    plotas_grupiu <- kulturos_grupes %>% left_join(paseliu_grupes, by = c("grupes_nr" = "grupe"))

### Step 12: 
Grouped by plant group and summarised declared area of each group.

    groupby_grupes <- plotas_grupiu %>% group_by(pavadinimas, METAI, grupes_nr) %>% 
        summarise(sumplotas = sum(PLOTAS), .groups = 'drop') %>% 
        as.data.frame()

![R1_4](https://github.com/user-attachments/assets/c5907404-12f1-403d-b646-2880d4e4cec2)


## ggplot2()

### Step 13:
Using ggplot2 created column chart of declared area of plant‘s groups.

Used reoder, to provide descending order of bars. Picked bar colors, separated thousands by comma on y-axis, added data labels, set y-axis and x-axis names, set theme.

    groupby_grupes %>% 
    ggplot(aes(x = reorder(pavadinimas, -sumplotas), y = sumplotas)) +
    geom_bar(stat = "identity", fill = "darkgreen") +
    scale_y_continuous(labels = scales::label_comma())+
    geom_text(aes(label = sumplotas), colour = "white", position = position_stack(vjust = 0.5))+
    labs(y="Deklaruotas plotas, ha", x = "Augalų grupė", title = "Deklaruotas žemės ūkio naudmenų plotas pagal augalų grupes")+
    theme_light()


![R1_5](https://github.com/user-attachments/assets/cf6c3718-ec9a-4272-af4a-a5622deb9a27)


### Step 14: 
Using ggplot2 created column chart of declared area of plants. 
Used reoder, to provide descending order of bars. Picked bar colors, separated thousands by comma on y-axis and set interval from 0 to 1000000, added data labels and changed angle so it would be vertical, set y-axis and x-axis names, set theme.


    kulturos_grupes %>% 
    ggplot(aes(x = reorder(KODAS, -PLOTAS), y = PLOTAS), environment = parent.frame()) +
    geom_bar(stat = "identity", fill = "darkgreen", width=0.6) +
    scale_y_continuous(labels = scales::label_comma(), limits=c(0,1000000))+
    geom_text(aes(label = PLOTAS), angle = 270, hjust=1, colour = "black")+
    labs(y="Deklaruotas plotas, ha", x = "Augalų grupė", title = "Deklaruotas žemės ūkio naudmenų plotas pagal augalų grupes")+
    theme(plot.title = element_text(hjust = 0.5))+
    theme_light()

![R1_6](https://github.com/user-attachments/assets/bbf3f0fb-84e3-48e3-b603-63255dcd6ce8)


### Insights
### Raw data about declared area of all plants were taken. Relevant plants were picked and column chart about declared area was created.

### Created my own table with groups of plants and assigned each plant to group. Column chart about declared area of plant‘s groups was created.

