Type 2 diabetes mellitus and COVID-19 in ICA - Cross-Sectional
================
Carlos Ballon-Salcedo & Kevin J. Paez

# Load packages

``` r
# Packages
pacman::p_load(rio,
               here,
               rfextras,
               reportfactory,
               tidyverse,
               ggcorrplot,
               ggsci,
               ggpubr,
               ggfortify,
               ggbiplot,
               finalfit,
               gtsummary,
               flextable,
               broom,
               performance,
               lmtest, 
               stats,
               Rtsne,
               FactoMineR, 
               factoextra,
               corrplot,
               grateful)

# My function scripts
rfextras::load_scripts()
```

# Import data

``` r
clean_data <- import(here("data", "clean_data.tsv"))
```

# Set theme

``` r
# Make my gtsummary theme
my_theme <-
  list(
    "pkgwide-fn:pvalue_fun" = function(x) style_pvalue(x, digits = 2),
    "pkgwide-fn:prependpvalue_fun" = function(x) style_pvalue(x, digits = 2, prepend_p = TRUE),

    "tbl_summary-str:continuous_stat" = "{median} ({p25}, {p75})",
    "tbl_summary-str:categorical_stat" = "{n} ({p}%)",
    
    "tbl_summary-fn:percent_fun" = function(x) style_number(x, digits = 1, scale = 100),
    
    "tbl_summary-arg:missing" = "no"
  )

# Combine themes
set_gtsummary_theme(my_theme, theme_gtsummary_compact())
theme_gtsummary_language(language = "en")
```

# Processing data

## Recode and relevel dataset

### Exposures

Possible risk factors associated with mortality from COVID-19 in
patients with type II diabetes mellitus, each factor is labeled with a
legend that indicates its clinical importance, accuracy, and number of
events.

- Clinically important with enough evidence but with small number of
  events, shown as `\####`
- Clinically important with enough evidence and enough number of events,
  shown as `\###`
- Clinically important with scarce evidence, shown as `\##`
- Inconsistent or contradictory evidence and unconfirmed accuracy
  (self-reported) `\#`

``` r
data <- clean_data |>
  mutate(edad = ff_label(edad, "Age (years)"), ###

         edad.c = case_when(edad <= 60 ~ "< 61", 
                            edad > 60 ~ ">= 61") |> 
           fct_relevel("< 61", ">= 61") |> 
           ff_label("Age (years)"),
         
         sexo = factor(sexo) |> ###
           fct_relevel("Female", "Male") |> 
           ff_label("Sex"),
         
         t_de_enfermedad = ff_label(t_de_enfermedad, 
                                    "Duration of disease (days)"), ##
         
         tabaquismo = factor(tabaquismo) |> #### 
           fct_relevel("No", "Yes") |> 
           ff_label("Smoking"),
         
         alcoholismo = factor(alcoholismo) |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Alcoholism"),
           
         obesidad = factor(obesidad) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Obesity"),
         
         asma_bronquial = factor(asma_bronquial) |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Asthma"),
         
         hta = factor(hta) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Hypertension"),
         
         dislipidemia = factor(dislipidemia) |> ##
           fct_relevel("No", "Yes") |>
           ff_label("Dyslipidemia"),
         
         ecv = factor(ecv) |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Cerebrovascular disease"),
         
         neoplasia = factor(neoplasia) |> #### 
           fct_relevel("No", "Yes") |>
           ff_label("Cancer"),
         
         vih = factor(vih) |> #### 
           fct_relevel("No", "Yes") |> 
           ff_label("HIV"),
         
         e_inmunosupresora = 
           case_when(neoplasia == "Yes" | vih == "Yes" | e_inmunosupresora == "Yes" ~ "Yes",
                     TRUE ~ "No") |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Immunesupressive disease"),
         
         erc = factor(erc) |> ####
           fct_recode("No" = "ON",) |> 
           fct_relevel("No", "Yes") |>
           ff_label("Chronic renal disease"),
         
         f_renal_aguda = factor(f_renal_aguda) |> #
           fct_relevel("No", "Yes") |> 
           ff_label("Acute kidney injury"),
      
         fiebre = factor(fiebre) |> ###
           fct_relevel("No", "Yes") |> 
           ff_label("Fever"),

         tos = factor(tos) |> ###
           fct_relevel("No", "Yes") |> 
           ff_label("Dry cought"),
         
         dolor_de_garganta = factor(dolor_de_garganta)|> #
           fct_relevel("No", "Yes") |>
           ff_label("Sore throat"),
         
         malestar_general = factor(malestar_general) |> #
           fct_relevel("No", "Yes") |>
           ff_label("General malaise"),
         
         cefalea = factor(cefalea) |> #
           fct_relevel("No", "Yes") |>
           ff_label("Headache"),
         
         astenia = factor(astenia) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Asthenia"),
         
         anosmia = factor(anosmia) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Anosmia"),
         
         disgeusia = factor(disgeusia) |> ### 
           fct_relevel("No", "Yes") |>
           ff_label("Dysgeusia"),
         
         disnea = factor(disnea) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Dyspnea"),
         
         perdida_de_peso = factor(perdida_de_peso) |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Weight loss"),
         
         estertores_pulmonares = factor(estertores_pulmonares) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Lung crackles"),
         
         diarrrea = factor(diarrrea) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Diarrhea"),

         emesis = factor(emesis) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Vomiting"),

         dolor_abdominal = factor(dolor_abdominal) |> ####
           fct_relevel("No", "Yes") |>
           ff_label("Abdominal pain"),

         poliuria = factor(poliuria) |> #### 
           fct_relevel("Yes", "No") |>
           ff_label("Polyuria"),

         polidipsia = factor(polidipsia) |> #### 
           fct_relevel("Yes", "No") |>
           ff_label("Polidipsia"),

         polifagia = factor(polifagia) |> ####
           fct_relevel("Yes", "No") |>
           ff_label("Poliphagia"),
        
         disgeusia = factor(disgeusia) |> ####
           fct_relevel("Yes", "No") |>
           ff_label("Dysgeusia"),

         taquipnea = factor(taquipnea) |> ###
           fct_relevel("No", "Yes") |>
           ff_label("Tachypnea"),

         sensorio = factor(sensorio) |> #
           fct_recode("Awake" = "DESPIERTO",
                      "Sleepy" = "SOMNOLIENTO",
                      "Drowsy" = "SOPOROSO") |> 
           fct_relevel("Awake",
                       "Sleepy",
                       "Drowsy") |>
           ff_label("Sensory"),
        
        ingreso_a_uci = factor(ingreso_a_uci) |> ####
          fct_relevel("No", "Yes") |>
          ff_label("ICU admission"),
        
        corticoides = case_when(corticoides == "No" ~ "No", 
                                TRUE ~ "Yes") |> ###
          fct_relevel("No", "Yes") |>
          ff_label("Corticosteroids"),
        
        anticoagulantes = factor(anticoagulantes) |> ###
          fct_recode("Enoxaparin" = "ENOXAPARINA") |>
          fct_relevel("No", "Enoxaparin") |>
          ff_label("Anticoagulants"),
        
        antiparasitarios = factor(antiparasitarios) |> ###
          fct_recode("Ivermectin" = "IVERMECTINA") |>
          fct_relevel("No", "Ivermectin") |>
          ff_label("Antiparasitics"),
        
        antipaludicos = factor(antipaludicos) |> ###
          fct_recode("Hydroxychloroquine" = "HIDROXICLOROQUINA") |>
          fct_relevel("No", "Hydroxychloroquine") |>
          ff_label("Antimalarials"),
        
        antibioticos = case_when(antibioticos == "No" ~ "No", 
                                 TRUE ~ "Yes") |> ###
          fct_relevel("No", "Yes") |>
          ff_label("Antibiotics"),
        
        pronacion = factor(pronacion) |> ###
          fct_relevel("Yes", "No") |>
          ff_label("Pronation"),
        
        hemodialisis = factor(hemodialisis) |> #
          fct_relevel("No", "Yes") |>
          ff_label("Hemodialysis"),
        
        sepsis = factor(sepsis) |> #
          fct_relevel("No", "Yes") |>
          ff_label("Sepsis"),
        
        shock_septico = factor(shock_septico) |> # 
          fct_relevel("No", "Yes") |>
          ff_label("Septic shock"),
        
        p_a_sistolica_ingreso = ff_label(p_a_sistolica_ingreso, 
                                         "SBP (mmHg)"), ###
        
        p_a_sistolica_ingreso.c = case_when(p_a_sistolica_ingreso < 140 ~ "< 140",
                                            TRUE ~ ">= 140") |> 
          fct_relevel("< 140",">= 140") |> 
          ff_label("SBP (mmHg)"),
        
        p_a_diastolica_ingreso = ff_label(p_a_diastolica_ingreso, 
                                          "DBP (mmHg)"), ###
        
        p_a_diastolica_ingreso.c = case_when(p_a_diastolica_ingreso < 90 ~ "< 90",
                                             TRUE ~ ">= 90") |> 
          fct_relevel("< 90", ">= 90") |> 
          ff_label("DBP (mmHg)"),
        
        hemoglobina_ingreso = ff_label(hemoglobina_ingreso, "Hemoglobin (g/dL)"), ###
      
        hemoglobina_ingreso.c = case_when(hemoglobina_ingreso <= 12 ~ "<= 12",
                                          TRUE ~ "> 12") |> 
          fct_relevel("> 12", "<= 12") |> 
          ff_label("Hemoglobin (g/dL)"),
        
        hematocrito_ingreso = ff_label(hematocrito_ingreso, "Hematocrit (%)"), ##
        
        hematocrito_ingreso.c = case_when(hematocrito_ingreso < 36 ~ "< 36",
                                          TRUE ~ ">= 36") |>
           fct_relevel(">= 36", "< 36") |> 
           ff_label("Hematocrit (%)"),
        
        mcv_ingreso = ff_label(mcv_ingreso, "MCV (mm^3)"), ##
        
        mch_ingreso = ff_label(mch_ingreso, "MCH (pg)"), ##
        
        leucocitos_ingreso = leucocitos_ingreso/1000, ###
        leucocitos_ingreso = ff_label(leucocitos_ingreso, "White-cells (×10^−9/L)"),
        
        leucocitos_ingreso.c = case_when(leucocitos_ingreso < 4 ~ "< 4",
                                         leucocitos_ingreso >= 4 & leucocitos_ingreso <= 10 ~ "4-10",
                                         leucocitos_ingreso > 10 ~ "> 10") |> 
          fct_relevel("4-10", "< 4", "> 10") |> 
          ff_label("White-cells (×10^−9/L)"),
        
        linfocitos_ingreso = linfocitos_ingreso/1000, ###
        linfocitos_ingreso = ff_label(linfocitos_ingreso, "Lymphocytes (×10^−9/L)"),

        linfocitos_ingreso.c = case_when(linfocitos_ingreso < 1 ~ "< 1",
                                         TRUE ~ ">= 1") |>
           fct_relevel(">= 1", "< 1") |> 
           ff_label("Lymphocytes (×10^−9/L)"),
        
        neutrofilos_ingreso = neutrofilos_ingreso/1000, ###
        neutrofilos_ingreso = ff_label(neutrofilos_ingreso, "Neutrophils (×10^−9/L)"),
        
        neutrofilos_ingreso.c = case_when(neutrofilos_ingreso > 6.3 ~ "> 6.3",
                                          TRUE ~ "<= 6.3") |>
          fct_relevel("<= 6.3", "> 6.3") |> 
          ff_label("Neutrophils (×10^−9/L)"),

        plaquetas_ingreso = plaquetas_ingreso/1000, ###
        plaquetas_ingreso = ff_label(plaquetas_ingreso, "Platelets (×10^−9/L)"),

        plaquetas_ingreso.c = case_when(plaquetas_ingreso < 125 ~ "< 125",
                                        TRUE ~ ">= 125") |>
          fct_relevel(">= 125", "< 125") |> 
          ff_label("Platelets (×10^−9/L)"),
        
        glucosa_ingreso = ff_label(glucosa_ingreso, "Glucose (mg/dL)"), ###
        
        urea_ingreso = ff_label(urea_ingreso, "Blood urea nitrogen (mg/dL)"), ### 
        
        urea_ingreso.c = case_when(urea_ingreso < 20 ~ "< 20",
                                   TRUE ~ ">= 20") |>
          fct_relevel("< 20", ">= 20") |> 
          ff_label("Blood Urea Nitrogen (mg/dL)"),
        
        creatinina_ingreso = ff_label(creatinina_ingreso, "Serum creatinine (mg/dL)"), ### 
      
        creatinina_ingreso.c = case_when(creatinina_ingreso < 1.3 ~ "< 1.3",
                                         TRUE ~ ">= 1.3") |>
          fct_relevel("< 1.3", ">= 1.3") |> 
          ff_label("Serum creatinine (mg/dL)"),

        ph_ingreso = ff_label(ph_ingreso, "pH"), ##
        
        ph_ingreso.c = case_when(ph_ingreso < 7.35 ~ "< 7.35",
                                 ph_ingreso >= 7.35 & ph_ingreso <= 7.45 ~ "7.35 - 7.45",
                                 ph_ingreso > 7.45 ~ "> 7.45") |>
          fct_relevel("7.35 - 7.45", "< 7.35", "> 7.45") |> 
          ff_label("pH"),
        
        frecuencia_cardiaca_ingreso = ff_label(frecuencia_cardiaca_ingreso, 
                                               "Heart rate"), ##
        
        frecuencia_cardiaca_ingreso.c = case_when(frecuencia_cardiaca_ingreso < 100 ~ "< 100",
                                                  TRUE ~ ">= 100") |> 
          fct_relevel("< 100", ">= 100") |> 
          ff_label("Heart rate"),
        
        frecuencia_respiratoria_ingreso = ff_label(frecuencia_respiratoria_ingreso,
                                                   "Respiratory rate"), ##
        
        frecuencia_respiratoria_ingreso.c = 
          case_when(frecuencia_respiratoria_ingreso < 24 ~ "< 24",
                    frecuencia_respiratoria_ingreso >= 24 & 
                      frecuencia_respiratoria_ingreso <= 30 ~ "24 - 30",
                    frecuencia_respiratoria_ingreso > 30 ~ "> 30") |> ##
          fct_relevel("24 - 30", "< 24", "> 30") |> 
          ff_label("Respiratory rate"),
      
        saturacion_de_oxigeno_ingreso = ff_label(saturacion_de_oxigeno_ingreso, 
                                                 "SaO2 (%)"), ####
        
        saturacion_de_oxigeno_ingreso.c = 
          case_when(saturacion_de_oxigeno_ingreso < 94 ~ "< 94", 
                    TRUE ~ ">= 94") |> ###
          fct_relevel(">= 94", "< 94") |> 
          ff_label("SaO2"),

        fio2_aga_ingreso = ff_label(fio2_aga_ingreso, "FiO2 (%)"), ###
        
        fio2_aga_ingreso.c = case_when(fio2_aga_ingreso > 21 ~ "> 21 (O2 therapy)",
                                       TRUE ~ "21") |>
          fct_relevel("> 21 (O2 therapy)", "21") |> 
          ff_label("FiO2 (%)"),
        
        po2_ingreso = ff_label(po2_ingreso, "PaO2 (mmHg)"), ##
      
        po2_ingreso.c = case_when(po2_ingreso < 60 ~ "< 60",
                                  TRUE ~ ">= 60") |> 
          fct_relevel(">= 60", "< 60") |> 
          ff_label("PaO2"),

        pco2_ingreso = ff_label(pco2_ingreso, 
                                "PCO (mmHg)"), ##
      
        pco2_ingreso.c = case_when(pco2_ingreso < 36 ~ "< 36", 
                                   TRUE ~ ">= 36") |> 
          fct_relevel("< 36", ">= 36") |> 
          ff_label("PCO (mmHg)"),
        
        pafi_ingreso = ff_label(pafi_ingreso, "PaO2:Fio2 ratio"), ###
        
        pafi_ingreso.c = case_when(pafi_ingreso <= 200 ~ "<= 200", 
                                   TRUE ~ "> 200") |> 
          fct_relevel("> 200", "<= 200") |> 
          ff_label("PaO2:Fio2 ratio"),
        
        hco3_ingreso = ff_label(hco3_ingreso, "HCO3 (mmol/L)"), ##
        
        hco3_ingreso.c = case_when(hco3_ingreso < 21 ~ "< 21",
                                   hco3_ingreso >= 21 & hco3_ingreso <= 28 ~ "21 - 28",
                                   hco3_ingreso > 28 ~ "> 28") |>
          fct_relevel("21 - 28", "< 21", "> 28") |> 
          ff_label("HCO3 (mEq/L)"),
        
        
        anion_gap_ingreso = ff_label(anion_gap_ingreso, "Anion Gap (mEq/L)"), ##
        
        anion_gap_ingreso.c = case_when(anion_gap_ingreso < 7 ~ "< 7",
                                        anion_gap_ingreso >= 7 & anion_gap_ingreso <= 13 ~ "7 - 13",
                                        anion_gap_ingreso > 13 ~ "> 13") |>
          fct_relevel("7 - 13", "< 7", "> 13") |> 
          ff_label("Anion Gap (mEq/L)"),
        
        sodio_ingreso = ff_label(sodio_ingreso, "Sodium (mmol/L)"), ##
        
        potasio_ingreso = ff_label(potasio_ingreso, "Potasium (mEq/L)"), ##
        
        calcio_ingreso = ff_label(calcio_ingreso, "Calcium (mmol/L)"), ##
        
        cloro_ingreso = ff_label(calcio_ingreso, "Chlorine (mmol/L)") ##
        )
```

### Outcomes

``` r
data <- data |>
  mutate(a_f = factor(a_f) |>
           fct_recode("Non-survivor" = "FALLECIDO",
                      "Survivor" = "ALTA") |>
           fct_relevel("Survivor", "Non-survivor"))
```

## Variable selection

``` r
data <- data |>
  select(# Demographics characteristics and history
         edad, edad.c, sexo, tabaquismo, alcoholismo, dislipidemia, obesidad, 
         hta, ecv, neoplasia, vih, e_inmunosupresora, erc, hemodialisis, 
         asma_bronquial, t_de_enfermedad,
         
         # Signs and symptoms
         fiebre, tos, dolor_de_garganta, malestar_general, cefalea, taquipnea,
         disnea, anosmia, disgeusia, estertores_pulmonares, diarrrea, emesis, 
         astenia, dolor_abdominal, perdida_de_peso, poliuria, polidipsia, 
         polifagia, sensorio,
         
         # Vital signs
         frecuencia_respiratoria_ingreso, frecuencia_respiratoria_ingreso.c, 
         frecuencia_cardiaca_ingreso, frecuencia_cardiaca_ingreso.c,
         p_a_sistolica_ingreso, p_a_sistolica_ingreso.c, p_a_diastolica_ingreso,
         p_a_diastolica_ingreso.c, 
         
         # Laboratory findings
         leucocitos_ingreso, leucocitos_ingreso.c, neutrofilos_ingreso, 
         neutrofilos_ingreso.c, linfocitos_ingreso, linfocitos_ingreso.c, 
         plaquetas_ingreso, plaquetas_ingreso.c, mcv_ingreso, mch_ingreso, 
         hemoglobina_ingreso, hemoglobina_ingreso.c, hematocrito_ingreso, 
         hematocrito_ingreso.c, creatinina_ingreso, creatinina_ingreso.c, 
         urea_ingreso, urea_ingreso.c, glucosa_ingreso, ph_ingreso, 
         ph_ingreso.c, anion_gap_ingreso, anion_gap_ingreso.c,
         sodio_ingreso, potasio_ingreso, cloro_ingreso, calcio_ingreso,
         
         # Blood gas findings
         saturacion_de_oxigeno_ingreso, saturacion_de_oxigeno_ingreso.c,
         fio2_aga_ingreso, fio2_aga_ingreso.c, pafi_ingreso, pafi_ingreso.c, 
         po2_ingreso, po2_ingreso.c, pco2_ingreso, pco2_ingreso.c, 
         hco3_ingreso, hco3_ingreso.c,
         
         # Treatment
         antibioticos, corticoides, anticoagulantes, antiparasitarios, 
         antipaludicos, pronacion,
         
         # outcomes
         a_f
         )
```

## Exploratory Data Analysis (EDA)

### Exploration of categorical variables

``` r
# Selection of factors
categorical <- data |> 
  dplyr::select(where(is.factor))

# Multiple tables
lapply(categorical, function(x) table(x, categorical$a_f))
```

### Exploration of numerical variables

``` r
# Selection of numerical variables and remove missing values
numerical <- data |> 
  dplyr::select(where(is.numeric), a_f) |>
  na.omit()

# Variable groups
dem_numerical <- numerical |> 
  dplyr::select(edad:p_a_diastolica_ingreso)

hemo_numerical <- numerical |> 
  dplyr::select(leucocitos_ingreso:mch_ingreso)

biochem_numerical <- numerical |> 
  dplyr::select(hemoglobina_ingreso:ph_ingreso)

electro_numerical <- numerical |> 
  dplyr::select(anion_gap_ingreso:calcio_ingreso, hco3_ingreso)

gas_numerical <- numerical |> 
  dplyr::select(saturacion_de_oxigeno_ingreso:pco2_ingreso)

# Formal and informal exploration
total_plots(hemo_numerical)
```

``` r
# Summary statistics
lapply(numerical, function(x) summary(x))
```

### Exploration of numerical variables in survivors and non-survivor

``` r
# Selection of numerical variables and remove missing values of survivors
surv_numerical <- data |> 
  dplyr::select(where(is.numeric), a_f) |>
  dplyr::filter(a_f == "Survivor") |>
  na.omit()

surv_dem_numerical <- surv_numerical |> 
  dplyr::select(edad:p_a_diastolica_ingreso)

surv_hemo_numerical <- surv_numerical |> 
  dplyr::select(leucocitos_ingreso:mch_ingreso)

surv_biochem_numerical <- surv_numerical |> 
  dplyr::select(hemoglobina_ingreso:ph_ingreso)

surv_electro_numerical <- surv_numerical |> 
  dplyr::select(anion_gap_ingreso:calcio_ingreso, hco3_ingreso)

surv_gas_numerical <- surv_numerical |> 
  dplyr::select(saturacion_de_oxigeno_ingreso:pco2_ingreso)

# Selection of numerical variables and remove missing values of non-survivors
non_numerical <- data |> 
  dplyr::select(where(is.numeric), a_f) |>
  dplyr::filter(a_f == "Non-survivor") |>
  na.omit()

non_dem_numerical <- non_numerical |> 
  dplyr::select(edad:p_a_diastolica_ingreso)

non_hemo_numerical <- non_numerical |> 
  dplyr::select(leucocitos_ingreso:mch_ingreso)

non_biochem_numerical <- non_numerical |> 
  dplyr::select(hemoglobina_ingreso:ph_ingreso)

non_electro_numerical <- non_numerical |> 
  dplyr::select(anion_gap_ingreso:calcio_ingreso, hco3_ingreso)

non_gas_numerical <- non_numerical |> 
  dplyr::select(saturacion_de_oxigeno_ingreso:pco2_ingreso)
```

``` r
# Summary statistics of survivors 
lapply(surv_numerical, function(x) summary(x))

# Summary statistics of non-survivors 
lapply(non_numerical, function(x) summary(x))
```

``` r
# Variable groups for clustering
a_dem_numerical <- numerical |> 
  dplyr::select(edad:p_a_diastolica_ingreso, a_f)

a_hemo_numerical <- numerical |> 
  dplyr::select(leucocitos_ingreso:mch_ingreso, a_f)

a_biochem_numerical <- numerical |> 
  dplyr::select(hemoglobina_ingreso:ph_ingreso, a_f)

a_electro_numerical <- numerical |> 
  dplyr::select(anion_gap_ingreso:calcio_ingreso, hco3_ingreso, a_f)

a_gas_numerical <- numerical |> 
  dplyr::select(saturacion_de_oxigeno_ingreso:pco2_ingreso, a_f)
```

``` r
# Formal and informal exploration
two_groups_plots(surv_hemo_numerical, non_hemo_numerical, a_hemo_numerical)
```

### Correlation matrix (multicollinearity)

We explore the correlation between independent variables to identify
multicollinearity. These variables by their nature are different except
for white-cells and PaO2:Fio2 ratio. In the case of using white-cells
and neutrophils or lymphocytes, it’s possible that the information
provided by white-cells may be redundant and generate biased estimates.
The same applies to PaO2:Fio2 ratio and FiO2. Considering the clinical
relevance of PaO2:Fio2 ratio, it’s better to work with the latter.

``` r
# Selection of numerical variables
data_num = data |> 
  dplyr::select(where(is.numeric)) |>
  na.omit()

# Rename variables
cor_data = rename(data_num, "Age" = edad,
             "Respiratory rate" = frecuencia_respiratoria_ingreso, 
             "Heart rate" = frecuencia_cardiaca_ingreso,
             "SBP" = p_a_sistolica_ingreso,
             "DBP" = p_a_diastolica_ingreso,
             "White-cells" = leucocitos_ingreso,
             "Neutrophils" = neutrofilos_ingreso,
             "Lymphocytes" = linfocitos_ingreso,
             "Platelets" = plaquetas_ingreso,
             "MCV" = mcv_ingreso,
             "MCH" = mch_ingreso,
             "Hemoglobin" = hemoglobina_ingreso, 
             "Hematocrit" = hematocrito_ingreso,
             "Serum creatinine" = creatinina_ingreso,
             "BUN" = urea_ingreso,
             "Glucose" = glucosa_ingreso, 
             "pH" = ph_ingreso,
             "Anion Gap" = anion_gap_ingreso,
             "Sodium" = sodio_ingreso,
             "Potasium " = potasio_ingreso, 
             "Chlorine" = cloro_ingreso,
             "Calcium" = calcio_ingreso,
             "SaO2" = saturacion_de_oxigeno_ingreso, 
             "FiO2" = fio2_aga_ingreso,
             "PaO2:Fio2 ratio" = pafi_ingreso,
             "PaO2" = po2_ingreso, 
             "PCO2" = pco2_ingreso,
             "HCO3" = hco3_ingreso,
             "Duration of disease" = t_de_enfermedad)

# Correlation matrix
corr <- round(cor(cor_data), 1)

# Visualization
my_ggcorrplor(corr) -> FS1

# View
FS1
```

# Produce outputs

## Table 1. Demographics and clinical characteristics of patients on admission

``` r
# Demographics characteristics and history
table_1.1 <- data |> 
  tbl_summary(include = c(edad:t_de_enfermedad, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1)))|> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})",
                stat_0 = "**All patients** (n = {N})",
                p.value = "**p value**") |>
  modify_spanning_header(all_stat_cols(stat_0 = FALSE) ~ "**Mortality**") |>
  modify_caption("**Table 1**. Demographics and clinical characteristics of patients on admission")

# Signs and symptoms
table_1.2 <- data |> 
  tbl_summary(include = c(fiebre:sensorio, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1))) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})") |> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05)

# Vital signs
table_1.3 <- data |> 
  tbl_summary(include = c(frecuencia_respiratoria_ingreso:p_a_diastolica_ingreso.c, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1))) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})") |> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05)

# Stack tables
table_1 = tbl_stack(
  list(table_1.1, table_1.2, table_1.3), 
  group_header = c("Demographics characteristics and history", "Signs and symtoms", "Vital signs"),
  quiet = TRUE)

# View
table_1
```

## Table 2. Laboratory findings and treatment of patients on admission

``` r
# Laboratory findings
table_2.1 <- data |> 
  tbl_summary(include = c(leucocitos_ingreso:calcio_ingreso, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1)))|> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})",
                stat_0 = "**All patients** (n = {N})",
                p.value = "**p value**") |>
  modify_column_alignment(columns = everything(), align = "left") |>
  modify_spanning_header(all_stat_cols(stat_0 = FALSE) ~ "**Mortality**") |>
  modify_caption("**Table 2**. Laboratory findings and treatment of patients on admission")

# Blood gas findings
table_2.2 <- data |> 
  tbl_summary(include = c(saturacion_de_oxigeno_ingreso:hco3_ingreso.c, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1))) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})") |> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05)

# Treatments
table_2.3 <- data |> 
  tbl_summary(include = c(antibioticos:pronacion, a_f),
              by = a_f, percent = "column",
              digits = list(all_continuous() ~ c(1, 1))) |>
  modify_header(all_stat_cols() ~ "**{level}** (n = {n})") |> 
  add_overall() |>
  add_p() |>
  bold_p(t = 0.05)

# Stack tables
table_2 = tbl_stack(
  list(table_2.1, table_2.2, table_2.3), 
  group_header = c("Laboratory findings", "Blood gas findings", "Treatment"),
  quiet = TRUE)

# View
table_2
```

## Table 3. Unadjusted and adjusted models

In the univariate analysis, self-reported or not enough event variables
were eliminated, this will help to avoid *overfitting* in the subsequent
multivariate analysis. Other variables such as plaquetas_ingreso,
plaquetas_ingreso.c, hemoglobina_ingreso, hematocrito_ingreso,
glucosa_ingreso, ph_ingreso, ph_ingreso.c, anion_gap_ingreso,
anion_gap_ingreso.c, sodio_ingreso, potasio_ingreso, cloro_ingreso, and
calcio_ingreso, fio2_aga_ingreso.c, pco2_ingreso.c, antiparasitarios,
and pronacion, showed *no differences* between groups in the bivariate
analysis (Table 2).

| Self-reported                                                                                                          | Not enough event (\<10)                                                                                                                                                                                                         |
|------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| dolor_de_garganta, malestar_general, cefalea, anosmia, disgeusia, astenia, dolor_abdominal, perdida_de_peso, sensorio. | tabaquismo, alcoholismo, dislipidemia, ecv, neoplasia, vih, e_inmunosupresora, erc, hemodialisis, asma_bronquial, anosmia, disgeusia, diarrrea, emesis, poliuria, polidipsia, polifagia, sensorio, ingreso_a_uci, antibioticos. |

Not analyzed variables

``` r
data_uv <- data |>
  dplyr::select(
         # Demographics characteristics and history
         edad.c, sexo, obesidad, hta, 
         
         # Signs and symptoms
         fiebre, tos, taquipnea,
         disnea, estertores_pulmonares,
         
         # Vital signs
         frecuencia_respiratoria_ingreso.c, frecuencia_cardiaca_ingreso.c,
         p_a_sistolica_ingreso.c, p_a_diastolica_ingreso.c, 
         
         # Laboratory findings
         leucocitos_ingreso.c, neutrofilos_ingreso.c, linfocitos_ingreso.c, 
         plaquetas_ingreso.c, mcv_ingreso, mch_ingreso, hemoglobina_ingreso.c,
         hematocrito_ingreso.c, creatinina_ingreso.c, urea_ingreso.c,

         # Blood gas findings
         saturacion_de_oxigeno_ingreso.c, fio2_aga_ingreso, pafi_ingreso.c, 
         po2_ingreso.c, pco2_ingreso, hco3_ingreso.c,
         
         # Treatment
         corticoides, anticoagulantes, antiparasitarios, antipaludicos, 
         pronacion,
         
         # outcomes
         a_f) |>
  
  na.omit() # Eliminate 82 observations
```

``` r
reset_gtsummary_theme()
theme_gtsummary_journal("jama")
theme_gtsummary_compact()
```

### Unadjusted models

> Note: All variables included in the formula are based on bivariate
> analysis.

``` r
table_3.1 <- data_uv |>
  tbl_uvregression(include = c(edad.c:pronacion),
                   y = a_f, 
                   method = glm, 
                   method.args = list(family = binomial),
                   exponentiate = TRUE,
                   conf.int = TRUE,
                   hide_n = TRUE, 
                   add_estimate_to_reference_rows = FALSE,
                   pvalue_fun = ~style_pvalue(.x, digits = 3),
                   estimate_fun = ~style_number(.x, digits = 2), 
                   label = list(edad.c ~ "Age (years)",
                                sexo ~ "Sex",
                                obesidad ~ "Obesity",
                                hta ~ "Hypertension",
                                fiebre ~ "Fever",
                                tos ~ "Dry cought",
                                taquipnea ~ "Tachypnea",
                                disnea ~ "Dyspnea",
                                estertores_pulmonares ~ "Lung crackles",
                                frecuencia_respiratoria_ingreso.c ~ "Respiratory rate",
                                frecuencia_cardiaca_ingreso.c ~ "Heart rate",
                                p_a_sistolica_ingreso.c ~ "Systolic blood pressure (mmHg)",
                                p_a_diastolica_ingreso.c ~ "Diastolic blood pressure (mmHg)",
                                leucocitos_ingreso.c ~ "White-cells",
                                neutrofilos_ingreso.c ~ "Neutrophils",
                                linfocitos_ingreso.c ~ "Lymphocytes",
                                plaquetas_ingreso.c ~ "Platelets",
                                mcv_ingreso ~ "MCV",
                                mch_ingreso ~ "MCH",
                                hemoglobina_ingreso.c ~ "Hemoglobin (g/dL)",
                                hematocrito_ingreso.c ~ "Hematocrit (%)",
                                creatinina_ingreso.c ~ "Serum creatinine (mg/dL)",
                                urea_ingreso.c ~ "BUN (mg/dL)",
                                saturacion_de_oxigeno_ingreso.c ~ "SaO2",
                                fio2_aga_ingreso ~ "FiO2 (%)",
                                pafi_ingreso.c ~ "PaO2:Fio2 ratio",
                                po2_ingreso.c ~ "PaO2 (mmHg)",
                                pco2_ingreso ~ "PCO2 (mmHg)",
                                hco3_ingreso.c ~ "HCO3 (mmol/L)",
                                corticoides ~ "Corticosteroids",
                                anticoagulantes ~ "Anticoagulants",
                                antiparasitarios ~ "Antiparasitics",
                                antipaludicos ~ "Antimalarials",
                                pronacion ~ "Pronation")) |>
  bold_labels() |>
  bold_p(t = 0.05) |>
  modify_header(estimate = "**Univariable OR (95% CI)**", 
                p.value = "**p value**")
```

### Adjusted models

#### Full multivariable model

``` r
# Model
full_multivariable <- 
  glm(a_f ~ edad.c + sexo + obesidad + hta + 
        fiebre + tos + taquipnea + disnea + estertores_pulmonares +
        frecuencia_respiratoria_ingreso.c + frecuencia_cardiaca_ingreso.c   + 
        p_a_sistolica_ingreso.c + p_a_diastolica_ingreso.c + 
        leucocitos_ingreso.c + neutrofilos_ingreso.c    + linfocitos_ingreso.c + 
        plaquetas_ingreso.c + mcv_ingreso + mch_ingreso + hemoglobina_ingreso.c +
        hematocrito_ingreso.c   + creatinina_ingreso.c + urea_ingreso.c + 
        saturacion_de_oxigeno_ingreso.c + fio2_aga_ingreso  + pafi_ingreso.c +
        po2_ingreso.c   + pco2_ingreso  + hco3_ingreso.c + 
        corticoides + anticoagulantes + antiparasitarios + antipaludicos + pronacion,
      data = data_uv, family = binomial(link = "logit")) |>
  tbl_regression(exponentiate = TRUE,
                 conf.int = TRUE,
                 pvalue_fun = ~style_pvalue(.x, digits = 3),
                 estimate_fun = ~style_number(.x, digits = 2)) |>
  bold_p(t = 0.05) |>
  
  # Add Generalized Variance Inflation Factor (GVIF)
  add_vif()
```

``` r
# Model
m1 = glm(a_f ~ edad.c + sexo + obesidad + hta + 
           fiebre + tos + taquipnea + disnea + estertores_pulmonares +
           frecuencia_respiratoria_ingreso.c + frecuencia_cardiaca_ingreso.c    + 
           p_a_sistolica_ingreso.c + p_a_diastolica_ingreso.c + 
           leucocitos_ingreso.c + neutrofilos_ingreso.c + linfocitos_ingreso.c + 
           plaquetas_ingreso.c + mcv_ingreso + mch_ingreso + hemoglobina_ingreso.c +
           hematocrito_ingreso.c    + creatinina_ingreso.c + urea_ingreso.c + 
           saturacion_de_oxigeno_ingreso.c + fio2_aga_ingreso   + pafi_ingreso.c +
           po2_ingreso.c    + pco2_ingreso  + hco3_ingreso.c + 
           corticoides + anticoagulantes + antiparasitarios + antipaludicos + pronacion, 
         data = data_uv, family = binomial(link = "logit"))

# Visual check of model assumptions
performance::check_model(m1)

# Indices of model performance
performance::model_performance(m1)

# Check for Multicollinearity
performance::check_collinearity(m1)
```

#### Reduced multivariable: step-by-step forward

``` r
mv_reg_stepbackward <- m1 |> 
  step(direction = "backward", trace = FALSE)
```

``` r
# Forward model
mv_reg_stepforward <- m1 |> 
  step(direction = "forward", trace = FALSE)

# Forward formula-based model
m2 <- glm(a_f ~ edad.c + sexo + obesidad + hta + fiebre + 
            tos + taquipnea + disnea + estertores_pulmonares + 
            frecuencia_respiratoria_ingreso.c + frecuencia_cardiaca_ingreso.c + 
            p_a_sistolica_ingreso.c + p_a_diastolica_ingreso.c + leucocitos_ingreso.c +
            neutrofilos_ingreso.c +linfocitos_ingreso.c + plaquetas_ingreso.c + 
            mcv_ingreso + mch_ingreso + hemoglobina_ingreso.c + hematocrito_ingreso.c + 
            creatinina_ingreso.c + urea_ingreso.c + saturacion_de_oxigeno_ingreso.c + 
            fio2_aga_ingreso + pafi_ingreso.c + po2_ingreso.c + pco2_ingreso + 
            hco3_ingreso.c + corticoides + anticoagulantes + antiparasitarios + 
            antipaludicos + pronacion, 
          family = binomial(link = "logit"), data = data_uv)

# Visual check of model assumptions
performance::check_model(m2)

# Indices of model performance
performance::model_performance(m2)

# Check for Multicollinearity
performance::check_collinearity(m2)
```

#### Reduced multivariable: step-by-step backward

``` r
# Backward model
mv_reg_stepbackward <- m1 |> 
  step(direction = "backward", trace = FALSE)

# Backward formula-based model
m3 <- glm(a_f ~ disnea + estertores_pulmonares + frecuencia_respiratoria_ingreso.c + 
            frecuencia_cardiaca_ingreso.c + p_a_sistolica_ingreso.c + 
            neutrofilos_ingreso.c + linfocitos_ingreso.c + mcv_ingreso + 
            mch_ingreso + hematocrito_ingreso.c + fio2_aga_ingreso + 
            po2_ingreso.c + corticoides + pronacion,
          family = binomial(link = "logit"), data = data_uv)

# Visual check of model assumptions
performance::check_model(m3)

# Indices of model performance
performance::model_performance(m3)

# Check for Multicollinearity
performance::check_collinearity(m3)
```

#### Parsimonious model

``` r
# Parsimonious formula
m4 <- glm(a_f ~ edad.c + obesidad + hta + taquipnea + disnea + estertores_pulmonares +
            frecuencia_cardiaca_ingreso.c + p_a_sistolica_ingreso.c + neutrofilos_ingreso.c +
            linfocitos_ingreso.c + mch_ingreso + hemoglobina_ingreso.c + urea_ingreso.c + 
            saturacion_de_oxigeno_ingreso.c + pafi_ingreso.c + po2_ingreso.c + 
            hco3_ingreso.c + corticoides,
          family = binomial(link = "logit"), data = data_uv)

# Visual check of model assumptions
check_model(m4)

# Indices of model performance
model_performance(m4)

# Check for Multicollinearity
check_collinearity(m4)
```

### Model comparison

``` r
# Compare performance of different models
compare_performance(m2, m3, m4, verbose = FALSE)

# Radar plot
plot(compare_performance(m2, m3, m4, rank = TRUE, verbose = FALSE))

# Likelihood Ratio Test
lmtest::lrtest(m3, m4)
```

> Likelihood Ratio Test (0.1131): There is insufficient evidence to
> conclude that the backward model is significantly better than the
> parsimonious model.

``` r
# Final model
table_3.2 <- 
  glm(a_f ~ edad.c + obesidad + hta + taquipnea + disnea + estertores_pulmonares +
            frecuencia_cardiaca_ingreso.c + p_a_sistolica_ingreso.c + neutrofilos_ingreso.c +
            linfocitos_ingreso.c + mch_ingreso + hemoglobina_ingreso.c + urea_ingreso.c + 
            saturacion_de_oxigeno_ingreso.c + pafi_ingreso.c + po2_ingreso.c + 
            hco3_ingreso.c + corticoides,
          family = binomial(link = "logit"), data = data_uv) |>
  tbl_regression(conf.int = TRUE, exponentiate = TRUE,
                 pvalue_fun = ~style_pvalue(.x, digits = 3),
                 estimate_fun = ~style_number(.x, digits = 2),
                 label = list(edad.c ~ "Age (years)", obesidad ~ "Obesity",
                              hta ~ "Hypertension", taquipnea ~ "Tachypnea",
                              disnea ~ "Dyspnea", estertores_pulmonares ~ "Lung crackles",
                              frecuencia_cardiaca_ingreso.c ~ "Heart rate",
                              p_a_sistolica_ingreso.c ~ "Systolic blood pressure (mmHg)",
                              neutrofilos_ingreso.c ~ "Neutrophils",
                              linfocitos_ingreso.c ~ "Lymphocytes",
                              mch_ingreso ~ "MCH",
                              hemoglobina_ingreso.c ~ "Hemoglobin (g/dL)",
                              urea_ingreso.c ~ "BUN (mg/dL)",
                              saturacion_de_oxigeno_ingreso.c ~ "SaO2",
                              pafi_ingreso.c ~ "PaO2:Fio2 ratio",
                              po2_ingreso.c ~ "PaO2 (mmHg)",
                              hco3_ingreso.c ~ "HCO3 (mmol/L)",
                              corticoides ~ "Corticosteroids")) |>
  bold_p(t = 0.05) |>
  add_vif() |>
  modify_header(estimate = "**Multivariable OR (95% CI)**", 
                p.value = "**p value** ")

# Merge tables
table_3 <- tbl_merge(tbls = list(table_3.1, table_3.2)) |>
  modify_spanning_header(everything() ~ NA_character_)

# View
table_3
```

## Dimensional reduction

### Principal Component Analysis (PCA)

PCA only works with numerical values. Dimensions and principal
components (PCs) are the same.

#### Variables

``` r
# Rename variables
numerical = rename(numerical, "Age" = edad,
                   "Respiratory rate" = frecuencia_respiratoria_ingreso, 
                   "Heart rate" = frecuencia_cardiaca_ingreso,
                   "SBP" = p_a_sistolica_ingreso,
                   "DBP" = p_a_diastolica_ingreso,
                   "White-cells" = leucocitos_ingreso,
                   "Neutrophils" = neutrofilos_ingreso,
                   "Lymphocytes" = linfocitos_ingreso,
                   "Platelets" = plaquetas_ingreso,
                   "MCV" = mcv_ingreso,
                   "MCH" = mch_ingreso,
                   "Hemoglobin" = hemoglobina_ingreso, 
                   "Hematocrit" = hematocrito_ingreso,
                   "Serum creatinine" = creatinina_ingreso,
                   "BUN" = urea_ingreso,
                   "Glucose" = glucosa_ingreso, 
                   "pH" = ph_ingreso,
                   "Anion Gap" = anion_gap_ingreso,
                   "Sodium" = sodio_ingreso,
                   "Potasium " = potasio_ingreso, 
                   "Chlorine" = cloro_ingreso,
                   "Calcium" = calcio_ingreso,
                   "SaO2" = saturacion_de_oxigeno_ingreso, 
                   "FiO2" = fio2_aga_ingreso,
                   "PaO2:Fio2 ratio" = pafi_ingreso,
                   "PaO2" = po2_ingreso, 
                   "PCO2" = pco2_ingreso,
                   "HCO3" = hco3_ingreso,
                   "Duration of disease" = t_de_enfermedad)
```

``` r
# Scaled (standardization)
pca_data <- stats::prcomp(numerical[1:29], scale. = TRUE, center = TRUE)

# Extract eigenvalues of dimensions
get_eigenvalue(pca_data)

# Graph eigenvalues of dimensions
fviz_eig(pca_data, addlabels = TRUE)

# Extract results for variables
var <- get_pca_var(pca_data)

# Coordinates for variables
head(var$coord, 5)

# Correlations between variables and dimensions
head(var$cor, 4)

# Quality of representation (Cos2) of variables to dimensions
head(var$cos2, 5)

# Graph of correlation matrix - Cos2 of variables
my_ggcorrplor(var$cos2)

# Graph of Cos2 of variables to dimensions 1 and 2
fviz_cos2(pca_data, choice = "var", axes = 1:2)

# Graph of 10 variables with highest cos2 to PC1 and PC2
a <- fviz_cos2(pca_data, choice = "var", axes = 1:2, top = 10)

# Graph of 10 variables with highest cos2 to PC3 and PC4
b <- fviz_cos2(pca_data, choice = "var", axes = 3:4, top = 10)

# Correlations circle and color by cos2 values 
c <- fviz_pca_var(pca_data, col.var = "cos2",
                  gradient.cols = c("#fee0d2", "#fc9272", "#de2d26"), 
                  repel = TRUE # Avoid text overlapping
                  )

# Contributions of variables to dimensions
head(var$contrib, 5)

# Graph of correlation matrix - contributions of variables
corrplot(var$contrib, is.corr=FALSE)

# Graph of contributions of top 10 variables to PC1 
fviz_contrib(pca_data, choice = "var", axes = 1, top = 10)

# Graph of contributions of top 10 variables to PC2
fviz_contrib(pca_data, choice = "var", axes = 2, top = 10)

# Graph of 10 variables with highest contribution to PC1 and PC2
d <- fviz_contrib(pca_data, choice = "var", axes = 1:2, top = 10)

# Graph of 10 variables with highest contribution to PC3 and PC4
e <- fviz_contrib(pca_data, choice = "var", axes = 3:4, top = 10)

# Correlations circle and color by contributions values 
f <- fviz_pca_var(pca_data, col.var = "contrib",
                  gradient.cols = c("#fee0d2", "#fc9272", "#de2d26"),
                  repel = TRUE)
```

``` r
# Arrange multiple plots
ggpubr::ggarrange(a, d, b, e, c, f, ncol = 2, nrow = 3, 
                  labels = c("A)", "B)", "C)", "D)", "E)", "F)"), 
                  legend = "right")
```

#### Individuals

``` r
# Extract results for individuals
ind <- get_pca_ind(pca_data)

# Coordinates for individuals
head(ind$coord, 5)

# Cos2 of individuals to dimensions
head(ind$cos2, 5)

# Graph of 30 individuals with highest cos2 to PC1 and PC2
a <- fviz_cos2(pca_data, choice = "ind", axes = 1:2, top = 30)

# Graph of 30 individuals with highest cos2 to PC3 and PC4
b <- fviz_cos2(pca_data, choice = "ind", axes = 3:4, top = 30)

# Correlations circle and color by cos2 values 
c <- fviz_pca_ind(pca_data, col.ind = "cos2", 
                  gradient.cols = c("#fee0d2", "#fc9272", "#de2d26"), 
                  repel = TRUE
                  )

# Contributions of individuals to dimensions
head(ind$contrib, 5)

# Graph of 30 individuals with highest contribution to PC1 and PC2
d <- fviz_contrib(pca_data, choice = "ind", axes = 1:2, top = 30)

# Graph of 30 individuals with highest contribution to PC3 and PC4
e <- fviz_contrib(pca_data, choice = "ind", axes = 3:4, top = 30)

# Correlations circle and color by contributions values 
f <- fviz_pca_ind(pca_data, col.ind = "contrib",
                  gradient.cols = c("#fee0d2", "#fc9272", "#de2d26")
                  )
```

``` r
# Arrange multiple plots
ggpubr::ggarrange(a, d, b, e, c, f, ncol = 2, nrow = 3, 
                  labels = c("A)", "B)", "C)", "D)", "E)", "F)"), 
                  legend = "right")
```

``` r
# Graph of individuals and color by group for PC1 and PC2
ind.graph <- fviz_pca_ind(pca_data,
                      axes = 1:2,
                      geom.ind = "point", 
                      col.ind = numerical$a_f,
                      palette = "igv",
                      addEllipses = TRUE, 
                      mean.point = FALSE, 
                      ggtheme = theme_pubr())

a <- ggpubr::ggpar(ind.graph,
                   title = element_blank(),
                   xlab = "PC1 (12.7%)", ylab = "PC2 (10.6%)",
                   legend.title = "Group", legend = "right",
                   font.legend = c(12, "black")
                   )

# Graph of individuals and color by group for PC3 and PC4
ind.graph <- fviz_pca_ind(pca_data,
                      axes = 3:4,
                      geom.ind = "point", 
                      col.ind = numerical$a_f,
                      palette = "igv",
                      addEllipses = TRUE, 
                      mean.point = FALSE, 
                      ggtheme = theme_pubr())

b <- ggpubr::ggpar(ind.graph,
                   title = element_blank(),
                   xlab = "PC3 (9.2%)", ylab = "PC2 (8.6%)",
                   legend.title = "Group", legend = "right",
                   font.legend = c(12, "black")
                   )
```

``` r
# Arrange multiple plots
ggpubr::ggarrange(a, b, ncol = 2, nrow = 1, labels = c("A)", "B)"), 
                  legend = "bottom", common.legend = TRUE)
```

#### Biplot of individuals and variables

``` r
# Contributions for PC1 and PC2
a <- fviz_pca_biplot(pca_data,
                     # Dimensions 1 and 2
                     axes = 1:2,
                     # Individuals
                     col.ind = numerical$a_f,
                     geom.ind = "point",
                     col.var = "black",
                     # Top 10 contributing variables
                     geom.var = "text", select.var = list(contrib = 10),
                     # Theme
                     palette = "Set1", 
                     addEllipses = TRUE, mean.point = FALSE, repel = TRUE,
                     ggtheme = theme_pubr())

# Graphical parameters
a1 <- ggpubr::ggpar(a,
                    title = element_blank(),
                    xlab = "PC1 (12.7%)", ylab = "PC2 (10.6%)",
                    legend.title = "Group",
                    font.legend = c(12, "black")
                    )

# Contributions for PC3 and PC4
b <- fviz_pca_biplot(pca_data,
                     # Dimensions 3 and 4
                     axes = 3:4,
                     # Individuals
                     col.ind = numerical$a_f,
                     geom.ind = "point",
                     col.var = "black",
                     # Top 10 contributing variables
                     geom.var = "text", select.var = list(contrib = 10),
                     # Theme
                     palette = "Set1", 
                     addEllipses = TRUE, mean.point = FALSE, repel = TRUE,
                     ggtheme = theme_pubr())

# Graphical parameters
b1 <- ggpubr::ggpar(b,
                    title = element_blank(),
                    xlab = "PC3 (9.2%)", ylab = "PC4 (8.6%)",
                    legend.title = "Group",
                    font.legend = c(12, "black")
                    )

# Arrange multiple plots
F1 <- ggpubr::ggarrange(a1, b1, ncol = 2, nrow = 1, labels = c("A)", "B)"), 
                        legend = "bottom", common.legend = TRUE)

# View
F1
```

### t-Distributed Stochastic Neighbor Embedding (t-SNE)

``` r
tsne_numerical <- data |> 
  dplyr::select(where(is.numeric), -t_de_enfermedad, a_f) |>
  na.omit() |>
  mutate(ID=row_number())

meta_numerical <- tsne_numerical |>
  dplyr::select(ID, a_f)

tSNE_fit <- tsne_numerical |>
  dplyr::select(where(is.numeric)) |>
  scale() |>
  Rtsne()

tSNE_df <- tSNE_fit$Y %>% 
  as.data.frame() %>%
  rename(tSNE1="V1",
         tSNE2="V2") %>%
  mutate(ID=row_number())

tSNE_df <- tSNE_df |>
  inner_join(meta_numerical, by="ID")

tSNE_df |>
  ggplot(aes(x = tSNE1, 
             y = tSNE2,
             color = a_f))+
  geom_point() +
  scale_color_igv() +
  labs(color = "Group") + 
  theme_minimal() +
  theme(legend.position = "right", 
        axis.line = element_line(color = "black"))
```

# Save outputs

``` r
# Table 1
table_1_p <- as_flex_table(table_1)

# Table 2
table_2_p <- as_flex_table(table_2)

# Table 3
table_3_p <- as_flex_table(table_3)

# Save tables
save_as_docx(table_1_p, path = "Table_1.docx", align = "center")

save_as_docx(table_2_p, path = "Table_2.docx", align = "center")

save_as_docx(table_3_p, path = "Table_3.docx", align = "center")

# Save supplementary figure 1 (EPS)
ggsave(plot = FS1, filename = "FIG_S1.eps", width = 12, height = 12,
       units = "in")

# Save figure 1 (EPS)
ggsave(plot = F1, filename = "FIG2.eps", width = 14, height = 6,
       units = "in")

# Save supplementary figure 1 (PNG)
ggsave(plot = FS1, filename = "FIG_S1.png", width = 12, height = 12,
       dpi = 300, units = "in")

# Save figure 1 (PNG)
ggsave(plot = F1, filename = "FIG2.png", width = 14, height = 6,
       dpi = 300, units = "in")
```
