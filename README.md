## Provided applications by declared area intervals

Target was to filter and prepare relevant data which I could visualise and use for loop.

Project was made based on the following statistics:

https://app.powerbi.com/view?r=eyJrIjoiYjVkM2Y5ZDktZmM3NS00MmNjLWExYjUtMmI5YmZkMWQxYzY2IiwidCI6IjNjMjk2MzFmLTAyN2EtNGFlYy05OGQxLWJlMGNjODg4MzAxNiIsImMiOjl9

### Paveiksliukas 1

NOTE. Data only about public persons.

### Step 1: 

Imported necessary libraries.

### Step 2: 

Imported data array about declared area of each public person by municipality.

    valdu_plotas <- read_excel("N:/ProjektasR/valdu_plotas.xlsx", col_names =TRUE)

### Paveiksliukas 2

### Step 3: 

Removed unecessary column about municipalities.

    besav <- subset(valdu_plotas, select = -c(VALDOS_CENTR_SAV))

### Step 4: 

Dublicating column plotas and creating one more column, so I could use for loop for changing values.

    besav$kodas <- besav$PLOTAS

### Step 5: 

By using for loop assigning values in created column [kodas] depending on declared area.

    for (plotas in 1:nrow(besav)){
        if (besav$kodas[plotas] < 50){
        besav$kodas[plotas] <- 1
        }
            else if (besav$kodas[plotas] < 100){
            besav$kodas[plotas] <- 2
         }
            else if (besav$kodas[plotas] < 200){
            besav$kodas[plotas] <- 3
        }
            else if (besav$kodas[plotas] < 300){
            besav$kodas[plotas] <- 4
        }
            else if (besav$kodas[plotas] < 400){
            besav$kodas[plotas] <- 5
        }
            else if (besav$kodas[plotas] < 500){
            besav$kodas[plotas] <- 6
        }
            else {
            besav$kodas[plotas] <- 7
        }
    }

### Step 6: 

Group by assigned number [kodas] and counting rows of each group and summarising declaread area.

    valdu_gr <- besav %>% group_by(grupe = besav$kodas) %>% 
        summarise(paraiskos = n(), plotas = sum(PLOTAS))


### Step 7: 

Creating new column by assigned code with names of intervals.  

    intervalai <- mutate(valdu_gr, intervalas = case_when(grupe == 1 ~ "<= 50 ha",
                                        grupe == 2 ~ "50,01 - 100,00 ha",
                                        grupe == 3 ~ "100,01 - 200,00 ha",
                                        grupe == 4 ~ "200,01 - 300,00 ha",
                                        grupe == 5 ~ "300,01 - 400,00 ha",
                                        grupe == 6 ~ "400,01 - 500,00 ha",
                                        grupe == 7 ~ ">500 ha"))


### Step 8: 

Turning ‘intervalai‘ into a factor with levels in the correct order. So order in line chart would be correct.

    intervalai$intervalas <- factor(intervalai$intervalas, levels=unique(intervalai$intervalas))

### Step 9: 

Using ggplot2 created 2 lines charts.

#### Declared area change by declared area intervals:

    ggplot(data=intervalai, aes(x = intervalas, y = plotas, group = 1))+
        theme(axis.text.x = element_text(angle = 45))+
        labs(y= "Deklaruotas plotas, ha", x = "Deklaruoto ploto intervalas", title = "Deklaruoto ploto pokytis pagal deklaruoto ploto intervalus")+
        theme(plot.title = element_text(hjust = 0.5))+
        scale_y_continuous(labels = scales::label_comma(), limits=c(100000,500000))+
        geom_line(colour = "darkgreen", size = 1)+
        geom_point()+
        geom_text(aes(label = plotas), colour = "red")

### Paveiksliukas 3

#### Provided applications by declared area intervals.

    ggplot(data=intervalai, aes(x = intervalas, y = paraiskos, group = 1))+
        theme(axis.text.x = element_text(angle = 45))+
        labs(y= "Deklaruotas plotas, ha", x = "Deklaruoto ploto intervalas", title = "Pateiktų paraiškų pokytis pagal deklaruoto ploto intervalus")+
        theme(plot.title = element_text(hjust = 0.5))+
        scale_y_continuous(labels = scales::label_comma(), limits=c(0,6000))+
        geom_line(colour = "darkgreen", size = 1)+
        geom_point()+
        geom_text(aes(label = paraiskos), colour = "red")

### Paveiksliukas 4

## Insights

### Raw data about declared area by municipalities by each farmer were used. Used for loop to assign interval group to each farmer.

### Created 2 line charts which provide statistics about declared area and provided applications by each interval

