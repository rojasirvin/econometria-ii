---
title: "Respuestas a la tarea 3"
lang: es
---


# Respuestas


::: {.cell}

```{.r .cell-code}
knitr::opts_chunk$set(echo = TRUE,
                      warning = FALSE,
                      message = FALSE,
                      indent = "   ")
library(tidyverse)
library(janitor)
library(sandwich)
library(clubSandwich)
library(plm)
library(stargazer)
library(lmtest)
library(AER)
library(Rxtsum)
```
:::



## Pregunta 1

En este ejercicio continuaremos usando los datos del estudio de Card (1993) para estudiar los rendimientos a la educación. Los datos *ingresos_iv.dta* contiene una muestra de hombres de entre 24 y 36 años de edad. **lwage** es el logaritmo del ingreso y **educ** es la educación acumulada.

a. [3 puntos] Estime una regresión por MCO para explicar el logaritmo del salario (**lwage**) en función de la educación **educ** y los siguientes controles: **exper**, **expersq**, **black**, **south**, **smsa**, **reg661**, **reg662**, **reg663**, **reg664**, **reg665**, **reg666**, **reg667**, **reg668** y **smsa66**. Reporte errores clásicos y errores robustos. ¿Qué problema encuentra en la estimación de esta relación? ¿El coeficiente sobre **educ** tiene una interpretación causal del efecto de la educación en el salario?

   *Estimamos por MCO la relación entre salarios y educación, controlando por un conjunto de regresores:*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   data.ingresos<-read_csv("../files/ingresos_iv.csv",
                           locale = locale(encoding = "latin1"))
   
   regmco <- lm(lwage ~ educ + exper + expersq + black + south + smsa + reg661 +
                 reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
               data = data.ingresos)
   
   stargazer(regmco, regmco,
             type = 'text',
             se = list(NULL,
                       sqrt(diag(vcovHC(regmco, type = "HC0")))),
             keep = 'educ')
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ============================================================
                                       Dependent variable:     
                                   ----------------------------
                                              lwage            
                                        (1)            (2)     
   ------------------------------------------------------------
   educ                               0.075***      0.075***   
                                      (0.003)        (0.004)   
                                                               
   ------------------------------------------------------------
   Observations                        3,010          3,010    
   R2                                  0.300          0.300    
   Adjusted R2                         0.296          0.296    
   Residual Std. Error (df = 2994)     0.372          0.372    
   F Statistic (df = 15; 2994)       85.476***      85.476***  
   ============================================================
   Note:                            *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::


   *Hay una relación de 7.4% mayor ingreso por cada año de educación adicional. Sin embargo, esta no es una relación causal pues es muy probable que la educación no sea exógena en la ecuación de salarios. Esto puede deberse, por ejemplo, a una variable omitida de habilidad que afecta tanto al número de años de educación alcanzados como al desempeño en el mercado de trabajo.*
    
a. [3 puntos] Se propone usar una variable dicotómica que indica si el individuo vivía cerca de una universidad cuando era niño, como instrumento de los años de educación. ¿Qué condiciones debe cumplir la variable propuesta para funcionar como instrumento válido?

   *El instrumento debe cumplir dos condiciones:*
    
   *Exogeneidad: el instrumento no debe pertenecer a la ecuación de salarios. Es decir, el haber crecido cerca de una universidad no debe afectar el salario contemporáneo de forma directa.*
    
   *Relevancia: el instrumento debe estar correlacionado con la variable endógena. En este caso, haber crecido cerca de una universidad debe estar correlacionado con el número de años de educación completados.*

a. [4 puntos] ¿Cómo juzga la propuesta de usar la variable antes descrita como instrumento?

   *Este argumento fue usado por [Card (1995)](https://www.nber.org/papers/w4483) para mostrar que los rendimientos a la educación están subestimados por un estimador de MCO. Card muestra que al usar variables instrumentales, el efecto estimado es de 25 a 60% más grande.*
    
   *No hay una respuesta correcta o incorrecta. Quiero leer sus argumentos.*

a. [3 puntos] Estime la relación entre el logaritmo del salario y la educación usando la variable dicotómica de acceso a una universidad, **nearc4**, como instrumento. Emplee las mismas variables de control que en el modelo de MCO. Reporte errores clásicos y errores robustos. 


   ::: {.cell}
   
   ```{.r .cell-code}
   regvi <- ivreg(lwage ~ educ + exper + expersq + black + south + smsa + reg661 +
                    reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66  |
                    nearc4 + exper + expersq + black + south + smsa + reg661 +
                    reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
                  data=data.ingresos)
   
   stargazer(regmco, regmco, regvi, regvi,
             type = 'text',
             se = list(NULL,
                       sqrt(diag(vcovHC(regmco, type = "HC0"))),
                       NULL,
                       sqrt(diag(vcovHC(regvi, type = "HC0")))),
             keep = 'educ')
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ===================================================================
                                           Dependent variable:        
                                   -----------------------------------
                                                  lwage               
                                           OLS          instrumental  
                                                          variable    
                                      (1)       (2)      (3)     (4)  
   -------------------------------------------------------------------
   educ                            0.075***  0.075***  0.132** 0.132**
                                    (0.003)   (0.004)  (0.055) (0.054)
                                                                      
   -------------------------------------------------------------------
   Observations                      3,010     3,010    3,010   3,010 
   R2                                0.300     0.300    0.238   0.238 
   Adjusted R2                       0.296     0.296    0.234   0.234 
   Residual Std. Error (df = 2994)   0.372     0.372    0.388   0.388 
   F Statistic (df = 15; 2994)     85.476*** 85.476***                
   ===================================================================
   Note:                                   *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::

    
a. [4 puntos] Interprete la primera etapa en términos del coeficiente sobre el instrumento. Obtenga el estadístico $F$ del instrumento excluido e interprete su magnitud.

   *En la primera etapa, haber vivido cerca de una universidad incrementa en 0.32 los años de escolaridad acumulados. Este efecto estadísticamente significativo al 1%. El estadístico F es de una magnitud de 13.256, por encima de 10, la regla de dedo comúnmente empleada para juzgar la presencia de instrumentos débiles.*


   ::: {.cell}
   
   ```{.r .cell-code}
   regpe <- lm(educ ~ nearc4 + exper + expersq + black + south + smsa + reg661 +
                 reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
               data=data.ingresos)
   
   stargazer(regpe,
             type = 'text',
             se = list(sqrt(diag(vcovHC(regpe, type = "HC0")))),
             keep = 'nearc4')
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ===============================================
                           Dependent variable:    
                       ---------------------------
                                  educ            
   -----------------------------------------------
   nearc4                       0.320***          
                                 (0.085)          
                                                  
   -----------------------------------------------
   Observations                   3,010           
   R2                             0.477           
   Adjusted R2                    0.474           
   Residual Std. Error      1.941 (df = 2994)     
   F Statistic         182.129*** (df = 15; 2994) 
   ===============================================
   Note:               *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   
   ```{.r .cell-code}
   linearHypothesis(regpe, "nearc4=0")
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   Linear hypothesis test
   
   Hypothesis:
   nearc4 = 0
   
   Model 1: restricted model
   Model 2: educ ~ nearc4 + exper + expersq + black + south + smsa + reg661 + 
       reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + 
       smsa66
   
     Res.Df   RSS Df Sum of Sq      F    Pr(>F)    
   1   2995 11324                                  
   2   2994 11274  1    49.917 13.256 0.0002763 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


a. [3 puntos] Interprete el coeficiente sobre la variable de educación en el modelo estructural. Compare la magnitud del efecto estimado con el resultado de MCO.

   *El coeficiente estimado sobre los años de educación indica que un año adicional de escolaridad incrementa en 13.15% el salario. Este efecto es casi el doble del estimado por MCO y estadísticamente significativo al 5%.*


   ::: {.cell}
   
   ```{.r .cell-code}
   stargazer(regmco, regvi,
             type = 'text',
             title="Comparación de estimadores de MCO y VI", 
             se = list(sqrt(diag(vcovHC(regmco, type = "HC0"))),
                       sqrt(diag(vcovHC(regvi, type = "HC0")))),
             keep = 'educ')
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   Comparación de estimadores de MCO y VI
   ======================================================================
                                            Dependent variable:          
                                   --------------------------------------
                                                   lwage                 
                                              OLS            instrumental
                                                               variable  
                                              (1)                (2)     
   ----------------------------------------------------------------------
   educ                                    0.075***            0.132**   
                                            (0.004)            (0.054)   
                                                                         
   ----------------------------------------------------------------------
   Observations                              3,010              3,010    
   R2                                        0.300              0.238    
   Adjusted R2                               0.296              0.234    
   Residual Std. Error (df = 2994)           0.372              0.388    
   F Statistic                     85.476*** (df = 15; 2994)             
   ======================================================================
   Note:                                      *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::



a. [3 puntos] Realice ahora el siguiente procedimiento. Primero, estime la primera etapa usando una regresión por MCO. Obtenga los valores ajustados de educación y llámelos **educ_hat**. Luego, estime la segunda etapa empleando **educ_hat** como variable independiente, además del resto de variables de control. ¿Cómo cambian sus resultados en comparación con la parte d.?

   *La magnitud de los coeficientes estimados es la misma. Esto es lo que esperábamos pues sabemos que el estimador de MC2E puede entenderse como un procedimiento donde primero se estiman los valores ajustados de la variable endógena usando el instrumento y las variables de control y luego se usan estos valores ajustados en la ecuación estructural. En cambio, los errores estándar son algo distintos.*


   ::: {.cell}
   
   ```{.r .cell-code}
   data.ingresos <- data.ingresos %>% 
     mutate(educ_hat = predict(regpe, .))
   
   reg2e <- lm(lwage ~ educ_hat + exper + expersq + black + south + smsa + reg661 +
                 reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
               data=data.ingresos)
   
   #Comparamos
   stargazer(regvi, reg2e,   
             title="Comparación de VI con la función ivreg y el estimador a mano",
             type="text", 
             keep = c("educ", "educ_hat"),
             df=FALSE, digits=4)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   Comparación de VI con la función ivreg y el estimador a mano
   ================================================
                           Dependent variable:     
                       ----------------------------
                                  lwage            
                        instrumental       OLS     
                          variable                 
                             (1)           (2)     
   ------------------------------------------------
   educ                   0.1315**                 
                          (0.0550)                 
                                                   
   educ_hat                              0.1315**  
                                         (0.0565)  
                                                   
   ------------------------------------------------
   Observations             3,010         3,010    
   R2                      0.2382         0.1947   
   Adjusted R2             0.2343         0.1907   
   Residual Std. Error     0.3883         0.3993   
   F Statistic                          48.2537*** 
   ================================================
   Note:                *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::


a. [3 puntos] ¿A qué se deben las discrepancias que encuentra? ¿Cuál de las dos estrategias prefiere para estimar el modelo de variables instrumentales?

   *Los coeficientes estimados son exactamente iguales, pero los errores estándar no. El problema es que nuestro procedimiento de MC2E a mano no toma en cuenta que en la ecuación estructural estamos usando valores ajustados de la variable endógena. Las funciones en la mayoría de los paquetes utilizados en econometría calculan los errores estándar de manera correcta. Preferimos usar las funciones previamente ya programadas cuando sea posible, aunque este ejercicio nos ayuda a reforzar la intuición del estimador de MC2E.*

a. [3 puntos] Reestime el modelo de variables instrumentales añadiendo un segundo instrumento, **nearc2**, y reporte errores robustos (no es necesario usar *gmm*, puede seguir con *ivreg*, por lo que no estaría obteniendo el estimador de MGM óptimo). ¿Cómo cambian sus resultados para la ecuación estructural con respecto al caso exactamente identificado?

   *El efecto estimado es significativo al 1% y en magnitud se incrementa ligeramente hasta 0.157.*


   ::: {.cell}
   
   ```{.r .cell-code}
   regvi2 <- ivreg(lwage ~ educ + exper + expersq + black + south + smsa + reg661 +
                    reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66  |
                    nearc4 + nearc2 + exper + expersq + black + south + smsa + reg661 +
                    reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
                  data=data.ingresos)
   
   stargazer(regmco, regvi, regvi2,
             type = 'text',
             se = list(sqrt(diag(vcovHC(regmco, type = "HC0"))),
                       sqrt(diag(vcovHC(regvi, type = "HC0"))),
                       sqrt(diag(vcovHC(regvi2, type = "HC0")))),
             keep = 'educ')
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ==========================================================================
                                              Dependent variable:            
                                   ------------------------------------------
                                                     lwage                   
                                              OLS              instrumental  
                                                                 variable    
                                              (1)              (2)     (3)   
   --------------------------------------------------------------------------
   educ                                    0.075***          0.132** 0.157***
                                            (0.004)          (0.054) (0.052) 
                                                                             
   --------------------------------------------------------------------------
   Observations                              3,010            3,010   3,010  
   R2                                        0.300            0.238   0.170  
   Adjusted R2                               0.296            0.234   0.166  
   Residual Std. Error (df = 2994)           0.372            0.388   0.405  
   F Statistic                     85.476*** (df = 15; 2994)                 
   ==========================================================================
   Note:                                          *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::

    
a. [3 puntos] Con el objeto que resulta de la estimación del modelo sobreidentificado, realice *summary(OBJETO, vcov = sandwich, diagnostics = TRUE)* para obtener las tres pruebas diagnóstico más usadas en variables instrumentales: prueba de instrumentos débiles, prueba de Hausman y prueba de Sargan. Interprete cada una de las pruebas.


   ::: {.cell}
   
   ```{.r .cell-code}
   summary(regvi2, vcov = sandwich, diagnostics = TRUE)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   Call:
   ivreg(formula = lwage ~ educ + exper + expersq + black + south + 
       smsa + reg661 + reg662 + reg663 + reg664 + reg665 + reg666 + 
       reg667 + reg668 + smsa66 | nearc4 + nearc2 + exper + expersq + 
       black + south + smsa + reg661 + reg662 + reg663 + reg664 + 
       reg665 + reg666 + reg667 + reg668 + smsa66, data = data.ingresos)
   
   Residuals:
        Min       1Q   Median       3Q      Max 
   -1.93841 -0.25068  0.01932  0.26519  1.46998 
   
   Coefficients:
                 Estimate Std. Error t value Pr(>|t|)    
   (Intercept)  3.3396868  0.8909170   3.749 0.000181 ***
   educ         0.1570594  0.0524127   2.997 0.002753 ** 
   exper        0.1188149  0.0228905   5.191 2.24e-07 ***
   expersq     -0.0023565  0.0003674  -6.414 1.64e-10 ***
   black       -0.1232778  0.0514904  -2.394 0.016718 *  
   south       -0.1431945  0.0301873  -4.744 2.20e-06 ***
   smsa         0.1007530  0.0313621   3.213 0.001329 ** 
   reg661      -0.1029760  0.0425755  -2.419 0.015637 *  
   reg662      -0.0002286  0.0345230  -0.007 0.994716    
   reg663       0.0469556  0.0335252   1.401 0.161435    
   reg664      -0.0554084  0.0408927  -1.355 0.175529    
   reg665       0.0515041  0.0506274   1.017 0.309085    
   reg666       0.0699968  0.0534531   1.309 0.190466    
   reg667       0.0390596  0.0514309   0.759 0.447639    
   reg668      -0.1980371  0.0522335  -3.791 0.000153 ***
   smsa66       0.0150626  0.0211120   0.713 0.475616    
   
   Diagnostic tests:
                     df1  df2 statistic  p-value    
   Weak instruments    2 2993     8.366 0.000238 ***
   Wu-Hausman          1 2993     2.978 0.084509 .  
   Sargan              1   NA     1.248 0.263905    
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   
   Residual standard error: 0.4053 on 2994 degrees of freedom
   Multiple R-Squared: 0.1702,	Adjusted R-squared: 0.166 
   Wald test: 51.65 on 15 and 2994 DF,  p-value: < 2.2e-16 
   ```
   :::
   :::

   *La prueba de instrumentos débiles rechaza la $H_0$ de que los instrumentos son débiles, por lo que tenemos primera etapa.*
    
   *La prueba de Hausman rechaza al 10% que los estimadores de VI y de MCO sean iguales, por lo que se prefiere el de VI.*
    
   *La prueba de Sargan no rechaza la $H_0$ de que el modelo esté mal especificado.*

a. [4 puntos] Considere la primera etapa del modelo sobreidentificado. Compruebe que si realiza una prueba de significancia conjunta para los instrumentos obtiene la prueba de instrumentos débiles que se reporta en el resumen que obtuvo con *summary*.

   *En el apartado anterior, obtenemos un valor $p=0.000238$ para la prueba de instrumentos débiles. Noten que se especificó una matriz de varianzas robustas. Entonces, tenemos que usar la misma matriz en la prueba $F$:*


   ::: {.cell}
   
   ```{.r .cell-code}
   regpe2 <- lm(educ ~ nearc4 + nearc2 + exper + expersq + black + south + smsa + reg661 +
                 reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66,
               data=data.ingresos)
   
   linearHypothesis(regpe2, c("nearc4=0", "nearc2=0"), white.adjust = "hc0")
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   Linear hypothesis test
   
   Hypothesis:
   nearc4 = 0
   nearc2 = 0
   
   Model 1: restricted model
   Model 2: educ ~ nearc4 + nearc2 + exper + expersq + black + south + smsa + 
       reg661 + reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + 
       reg668 + smsa66
   
   Note: Coefficient covariance matrix supplied.
   
     Res.Df Df      F    Pr(>F)    
   1   2995                        
   2   2993  2 8.3662 0.0002381 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


   *Obtenemos el mismo valor $p$ y maginitud del estadístico $F$.*
    
a. [4 puntos] Compruebe que si realiza el procedimiento de regresión auxiliar para la prueba de Hausman obtiene el mismo valor $p$ que se reporta en el resumen que obtuvo con *summary*.

   *De la primera etapa, obtenemos los residuales:*


   ::: {.cell}
   
   ```{.r .cell-code}
       data.ingresos <- data.ingresos %>% 
         mutate(vhat = resid(regpe2))
   ```
   :::

    
   *Corremos la regresión auxiliar con los residuales:*
    

   ::: {.cell}
   
   ```{.r .cell-code}
       regaux <- lm(lwage ~ educ + exper + expersq + black + south + smsa + reg661 +
                    reg662 + reg663 + reg664 + reg665 + reg666 + reg667 + reg668 + smsa66 + vhat,
                  data=data.ingresos)
   
   stargazer(regaux,
             type = 'text',
             se = list(sqrt(diag(vcovHC(regaux, type = "HC0")))),
             keep = 'vhat',
             report = c("c*s*p"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ===============================================
                           Dependent variable:    
                       ---------------------------
                                  lwage           
   -----------------------------------------------
                                 -0.083*          
                                (0.048)*          
                                p = 0.085         
                                                  
   -----------------------------------------------
   Observations                   3,010           
   R2                             0.301           
   Adjusted R2                    0.297           
   Residual Std. Error      0.372 (df = 2993)     
   F Statistic          80.368*** (df = 16; 2993) 
   ===============================================
   Note:               *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::

    
   *El valor $p$ es idéntico al obtenido con summary.*


## Pregunta 2

Considere los datos *comportamiento_wide.csv*, que contienen información individual de niñas y niños, incluyendo su género, edad, raza e información de sus madres. Además, se incluye una medida auto reportada de autoestima (**self**) y una evaluación de comportamiento antisocial (**anti**). Se quiere conocer cómo influye la autoestima en el comportamiento antisocial. Para cada niño o niña hay tres observaciones en el tiempo. Se busca explicar el comportamiento antisocial en función de la autoestima y la condición de pobreza (**pov**):

$$anti_{it}=\alpha_i+\beta_1 self_{it}+\beta_2 pov_{it}+\varepsilon_{it}$$

a. [2 puntos] La base se encuentra en formato *wide*. Ponga la base en formato *long*, donde haya una columna para cada variable y donde las filas representen a un individuo en un periodo.

   *Hay muchas formas de hacer esto. Podemos usar las funciones pivot_longer y pivot_wider, por ejemplo.*


   ::: {.cell}
   
   ```{.r .cell-code}
   data.comp <-read_csv("../files/comportamiento_wide.csv",
                         locale = locale(encoding = "latin1")) %>%
     pivot_longer(c(anti90:anti94,self90:self94,pov90:pov94),
                  names_to = c("measure", "year"),
                  names_pattern = "(.*)(..)")  %>%
     pivot_wider(names_from = measure,
                 values_from = value)
       
   colnames(data.comp)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
    [1] "id"       "momage"   "gender"   "childage" "hispanic" "black"   
    [7] "momwork"  "married"  "year"     "anti"     "self"     "pov"     
   ```
   :::
   :::



a. [2 puntos] Estime la ecuación de comportamiento antisocial empleando MCO *pooled*. ¿Cuáles son los supuestos que se deben cumplir para que $\hat{\beta}_1^{MCO}$ sea consistente?


   ::: {.cell}
   
   ```{.r .cell-code}
   summary(m.mco <- plm( anti ~ self + pov,
                         data=data.comp,
                         model="pooling",
                         index = c("id", "year")))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   Pooling Model
   
   Call:
   plm(formula = anti ~ self + pov, data = data.comp, model = "pooling", 
       index = c("id", "year"))
   
   Balanced Panel: n = 581, T = 3, N = 1743
   
   Residuals:
       Min.  1st Qu.   Median  3rd Qu.     Max. 
   -2.65689 -1.29476 -0.33138  0.98912  4.77034 
   
   Coefficients:
                Estimate Std. Error t-value  Pr(>|t|)    
   (Intercept)  2.792098   0.231110 12.0812 < 2.2e-16 ***
   self        -0.065102   0.011083 -5.8739 5.089e-09 ***
   pov          0.515809   0.078730  6.5516 7.476e-11 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   
   Total Sum of Squares:    4333.1
   Residual Sum of Squares: 4140.6
   R-Squared:      0.044433
   Adj. R-Squared: 0.043335
   F-statistic: 40.4544 on 2 and 1740 DF, p-value: < 2.22e-16
   ```
   :::
   :::


   *La variable self tiene un efecto negativo y estadísticamente significativo sobre anti. La variable pov tiene un efecto positivo y estadísticamente significativo. El estimador de MCO será consistente solo si las variables self y pov no están correlacionadas con el error. Además, para estimar este modelo, asumimos que la heterogeneidad no observada $\alpha_i$ puede escribirse simplemente como $\alpha$. Otra forma de pensar sobre este modelo es si el mismo modelo es válido para todos los periodos como para asumir una ordenada al origen y una pendiente común. El modelo pooled ignora la naturaleza en panel de los datos. Sin embargo, como tenemos a los mismos individuos en varios puntos del tiempo, los errores están agrupados, así que se deben de estimar errores con esta estructura. En este caso, al tomar en cuenta esta correlación entre grupos, los errores estándar son más grandes, pero los resultados siguen siendo significativos. En muchos casos, no tomar en cuenta la estructura agrupada de los errores puede llevar a rechazar hipótesis nulas que son ciertas.*


   ::: {.cell}
   
   ```{.r .cell-code}
   coeftest(m.mco,
            vcov = vcovHC(m.mco, type = "HC1", cluster="group"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   t test of coefficients:
   
                Estimate Std. Error t value  Pr(>|t|)    
   (Intercept)  2.792098   0.293380  9.5170 < 2.2e-16 ***
   self        -0.065102   0.013687 -4.7565 2.132e-06 ***
   pov          0.515809   0.104963  4.9142 9.753e-07 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::



a. [3 puntos] Estime la ecuación de comportamiento antisocial empleando el estimador *within*. ¿Cuáles son los supuestos que se deben cumplir para que $\hat{\beta}_1^{FE}$ sea consistente?


   *Si asumimos que la heterogeneidad no observada y el error están potencialmente correlacionados, entonces podemos usar un estimador de efectos fijos para deshacernos de la heterogeneidad no observada y estimar consistentemente los parámetros sobre self y pov.*


   ::: {.cell}
   
   ```{.r .cell-code}
   m.fe <- plm( anti ~ self + pov,
                data=data.comp,
                model="within",
                index = c("id", "year"))
   
   coeftest(m.fe,
            vcov = vcovHC(m.fe, type = "HC1", cluster="group"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   t test of coefficients:
   
         Estimate Std. Error t value  Pr(>|t|)    
   self -0.051495   0.011308 -4.5540 5.818e-06 ***
   pov   0.104899   0.099188  1.0576    0.2905    
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


a. [3 puntos] Estime la ecuación de comportamiento antisocial empleando efectos aleatorios. ¿Cuáles son los supuestos que se deben cumplir para que $\hat{\beta}_1^{RE}$ sea consistente?

   *Si estamos dispuestos a asumir que la heterogeneidad no observada y el error son independientes, podemos emplear el estimador de efectos aleatorios. MCO pooled también es consistente pero no es eficiente.*


   ::: {.cell}
   
   ```{.r .cell-code}
   m.re <- plm( anti ~ self + pov,
                data=data.comp,
                model="random",
                index = c("id", "year"))
   
   coeftest(m.re,
            vcov = vcovHC(m.re, type = "HC1", cluster="group"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   t test of coefficients:
   
                Estimate Std. Error t value  Pr(>|t|)    
   (Intercept)  2.695210   0.222607 12.1075 < 2.2e-16 ***
   self        -0.056732   0.010216 -5.5534 3.234e-08 ***
   pov          0.292407   0.081956  3.5679 0.0003696 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


a. [3 puntos] Se desea incorporar en el análisis el género (**gender**) y una variable dicotómica para los hispanos (**hispanic**). Indique qué modelo usaría y estime dicho modelo.


   *No es posible estimar los coeficientes sobre variables que no varían en el tiempo usando efectos fijos, por lo que este modelo queda descartado. Podríamos usar MCO pooled, que impone supuestos muy fuertes. La otra alternativa es un modelo de efectos aleatorios, que asume que la heterogeneidad no observada y el error no están correlacionados.*


   ::: {.cell}
   
   ```{.r .cell-code}
   m.sex <- plm( anti ~ self + pov + gender,
                 data=data.comp,
                 model="random",
                 index = c("id", "year"))
       
   coeftest(m.sex,
            vcov = vcovHC(m.sex, type = "HC1", cluster="group"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   t test of coefficients:
   
                Estimate Std. Error t value  Pr(>|t|)    
   (Intercept)  2.970534   0.231591 12.8267 < 2.2e-16 ***
   self        -0.058558   0.010223 -5.7278 1.197e-08 ***
   pov          0.304997   0.081486  3.7429 0.0001878 ***
   gender      -0.480468   0.107126 -4.4851 7.766e-06 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


a. [2 puntos] Regrese al modelo que incluye solo la autoestima y el estado de pobreza como covariables. Realice una prueba de Hausman para determinar si se prefiere un modelo de efectos fijos o uno de efectos aleatorios.

   *La implementación de la prueba de Hausman indica que se rechaza la H0 de que los coeficientes estimados son iguales (y que el modelo de efectos aleatorios es el adecuado). Hay evidencia de que se prefiere un modelo de efectos fijos, aunque tendremos que vivir con el hecho de no poder estimar el coeficiente asociado a las variables que no varían en el tiempo en este caso.*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   phtest(m.fe, m.re)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   	Hausman Test
   
   data:  anti ~ self + pov
   chisq = 13.578, df = 2, p-value = 0.001126
   alternative hypothesis: one model is inconsistent
   ```
   :::
   :::


## Pregunta 3

Retome los datos de la pregunta 2 y el modelo del comportamiento antisocial en función de la autoestima y la pobreza. En esta pregunta mostraremos la equivalencia del estimador within con otros estimadores.

a. [3 puntos] Compruebe que el estimador de efectos fijos es equivalente a MCO con dummies de individuos.

    *Comprobamos:*


   ::: {.cell}
   
   ```{.r .cell-code}
   m.fe <- plm( anti ~ self + pov,
                data=data.comp,
                model="within",
                index = c("id", "year"))
       
   m.dummy <- lm(anti ~ self + pov + factor(id),
                 data=data.comp)
   
   stargazer(m.fe, m.dummy, keep=c("self", "pov"), type="text")
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ======================================================================
                                      Dependent variable:                
                       --------------------------------------------------
                                              anti                       
                                panel                      OLS           
                                linear                                   
                                 (1)                       (2)           
   ----------------------------------------------------------------------
   self                       -0.051***                 -0.051***        
                               (0.011)                   (0.011)         
                                                                         
   pov                          0.105                     0.105          
                               (0.094)                   (0.094)         
                                                                         
   ----------------------------------------------------------------------
   Observations                 1,743                     1,743          
   R2                           0.021                     0.731          
   Adjusted R2                  -0.470                    0.596          
   Residual Std. Error                              1.002 (df = 1160)    
   F Statistic         12.551*** (df = 2; 1160) 5.417*** (df = 582; 1160)
   ======================================================================
   Note:                                      *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::


a. [2 puntos] Compruebe que en un modelo de efectos fijos las características que no varían en el tiempo no pueden ser identificadas. Añada la variable **black** para comprobarlo.

   *Comprobamos que la variable simplemente es omitida del análisis:*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   summary(plm( anti ~ self + pov + black,
                data=data.comp,
                model="within",
                index = c("id", "year")))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   Oneway (individual) effect Within Model
   
   Call:
   plm(formula = anti ~ self + pov + black, data = data.comp, model = "within", 
       index = c("id", "year"))
   
   Balanced Panel: n = 581, T = 3, N = 1743
   
   Residuals:
         Min.    1st Qu.     Median    3rd Qu.       Max. 
   -3.7868224 -0.4706542 -0.0012721  0.4534891  3.2646729 
   
   Coefficients:
         Estimate Std. Error t-value  Pr(>|t|)    
   self -0.051495   0.010530 -4.8902 1.149e-06 ***
   pov   0.104899   0.093880  1.1174    0.2641    
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   
   Total Sum of Squares:    1190.7
   Residual Sum of Squares: 1165.4
   R-Squared:      0.021182
   Adj. R-Squared: -0.46991
   F-statistic: 12.5514 on 2 and 1160 DF, p-value: 4.0471e-06
   ```
   :::
   :::


a. [5 puntos] Compruebe que el estimador de efectos fijos es equivalente a MCO sobre el modelo en diferencias con respecto a la media. Para esto, conserve dos periodos consecutivos de datos y solo observaciones que tengan datos para las variables dependientes e independientes en los dos años que elija. Luego estime por MCO el modelo con variables transformadas.

   *Nos quedamos con un subconjunto de datos:*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   data.comp.sub <- data.comp %>% 
     dplyr::select(id, year, anti, self, pov) %>% 
     filter(year==90 | year==92)
   
   #Nos quedamos con los que no son NA
   data.comp.sub <- data.comp.sub[complete.cases(data.comp.sub), ]
   ```
   :::


   *Creamos las variables como diferencias respecto a la media y estimamos el modelo within y el modelo de MCO en las variables transformadas:*


   ::: {.cell}
   
   ```{.r .cell-code}
   data.comp.sub <- data.comp.sub %>%
     group_by(id) %>% 
     mutate(m.anti = mean(anti),
            m.self = mean(self),
            m.pov = mean(pov)) %>% 
     mutate(dm.anti = anti - m.anti,
            dm.self = self - m.self,
            dm.pov = pov - m.pov)
   
   m.fe.sub <- plm( anti ~ self + pov,
                    data=data.comp.sub,
                    model="within",
                    index = c("id", "year"))
    
       
   m.demean <- lm(dm.anti ~ dm.self + dm.pov,
                  data.comp.sub)
   
   stargazer(m.fe.sub, m.demean,
             keep=c("self", "pov", "dm.self","dm.pov"),
             type="text")
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ==================================================================
                                    Dependent variable:              
                       ----------------------------------------------
                                anti                  dm.anti        
                               panel                    OLS          
                               linear                                
                                (1)                     (2)          
   ------------------------------------------------------------------
   self                      -0.038***                               
                              (0.014)                                
                                                                     
   pov                         0.195                                 
                              (0.133)                                
                                                                     
   dm.self                                           -0.038***       
                                                      (0.010)        
                                                                     
   dm.pov                                             0.195**        
                                                      (0.094)        
                                                                     
   ------------------------------------------------------------------
   Observations                1,162                   1,162         
   R2                          0.016                   0.016         
   Adjusted R2                 -0.972                  0.015         
   Residual Std. Error                           0.641 (df = 1159)   
   F Statistic         4.807*** (df = 2; 579) 9.621*** (df = 2; 1159)
   ==================================================================
   Note:                                  *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::


a. [5 puntos] Compruebe que el estimador de efectos fijos es equivalente a MCO sobre el modelo en primeras diferencias. Parta de la muestra con dos años de la parte d. para estimar por MCO el modelo con variables transformadas.

   *Usando el mismo subconjunto, calculamos ahora las primeras diferencias y estimamos:*


   ::: {.cell}
   
   ```{.r .cell-code}
   data.comp.sub <- data.comp.sub %>%
     group_by(id) %>% 
     mutate(d.anti = anti-dplyr::lag(anti, order_by = year),
            d.self = self-dplyr::lag(self, order_by = year),
            d.pov = pov-dplyr::lag(pov, order_by = year)) %>% 
     ungroup()
   
   
   m.difs <- lm(d.anti ~ -1 + d.self + d.pov,
                data=data.comp.sub)
   
   stargazer(m.fe.sub, m.demean, keep=c("self", "pov", "d.self","d.pov"), type="text")
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   ==================================================================
                                    Dependent variable:              
                       ----------------------------------------------
                                anti                  dm.anti        
                               panel                    OLS          
                               linear                                
                                (1)                     (2)          
   ------------------------------------------------------------------
   self                      -0.038***                               
                              (0.014)                                
                                                                     
   pov                         0.195                                 
                              (0.133)                                
                                                                     
   dm.self                                           -0.038***       
                                                      (0.010)        
                                                                     
   dm.pov                                             0.195**        
                                                      (0.094)        
                                                                     
   ------------------------------------------------------------------
   Observations                1,162                   1,162         
   R2                          0.016                   0.016         
   Adjusted R2                 -0.972                  0.015         
   Residual Std. Error                           0.641 (df = 1159)   
   F Statistic         4.807*** (df = 2; 579) 9.621*** (df = 2; 1159)
   ==================================================================
   Note:                                  *p<0.1; **p<0.05; ***p<0.01
   ```
   :::
   :::


## Pregunta 4

Considere los datos *mlbook1.csv* con información sobre 2287 estudiantes en 131 escuelas. Nos interesa la relación entre una medida de aptitud verbal,  (**iq_vert**) y el resultado de un examen de inglés (**langpost**). Las variables **schoolnr** y **pupilnr** identifican a las escuelas y los estudiantes, respectivamente. El modelo a estimar es el siguiente: 

$$langpost_{i}=\alpha+\beta iqvert_{i}+BX_{i}+\varepsilon_{i}$$
donde $i$ indexa y $X_i$ son tres características usadas como control: el sexo, **sex**, si el estudiante es de una población minoritaria, **minority** y el número de años repetidos, **repeatgr**.

a. [3 puntos] ¿Por qué es posible que estemos frente a una situación de errores agrupados?

   *Los datos están agrupados a nivel escuela. Los estudiantes en una misma escuela comparten características observadas y no observadas que hacen altamente probable que los factores no observables estén correlacionados entre los individuos, rompiendo el supuesto de independencia.*


a. [2 puntos] Estime la ecuación de calificación usando MCO ignorando la agrupación de datos. ¿Qué concluye respecto a la relación entre la aptitud verbal y la prueba de inglés?

   *Se concluye que una hora más en la prueba de aptitud incrementa en 2.49 puntos la calificación del examen. El error estándar es 0.072.*


   ::: {.cell}
   
   ```{.r .cell-code}
   data.examen<-read_csv("../files/mlbook1.csv",
                         locale = locale(encoding = "latin1")) 
   
   
   summary(m.mco <- lm(langpost ~ iq_verb + sex + minority + repeatgr, data=data.examen))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   Call:
   lm(formula = langpost ~ iq_verb + sex + minority + repeatgr, 
       data = data.examen)
   
   Residuals:
        Min       1Q   Median       3Q      Max 
   -28.0192  -4.2255   0.5218   4.8017  24.1421 
   
   Coefficients:
               Estimate Std. Error t value Pr(>|t|)    
   (Intercept) 10.93980    0.90504  12.088   <2e-16 ***
   iq_verb      2.48635    0.07233  34.374   <2e-16 ***
   sex          2.42228    0.28871   8.390   <2e-16 ***
   minority    -0.03701    0.62762  -0.059    0.953    
   repeatgr    -4.40860    0.43222 -10.200   <2e-16 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   
   Residual standard error: 6.853 on 2282 degrees of freedom
   Multiple R-squared:  0.4217,	Adjusted R-squared:  0.4207 
   F-statistic: 416.1 on 4 and 2282 DF,  p-value: < 2.2e-16
   ```
   :::
   :::


a. [3 puntos] Estime ahora los errores robustos a heteroscedasticidad del tipo HC1. ¿Qué cambia y por qué en la interpretación de la relación entre la prueba de aptitud y el examen?

   *El coeficiente estimado es el mismo. La fórmula empleada para calcular la varianza es una en forma de sándwich, que toma en cuenta la posible heterocedasticidad. El error estándar es apromximadamente 5% más grande, 0.076.*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   coeftest(m.mco, vcov = vcovHC(m.mco, type = "HC1"))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   t test of coefficients:
   
                Estimate Std. Error t value Pr(>|t|)    
   (Intercept) 10.939796   0.985476 11.1010   <2e-16 ***
   iq_verb      2.486350   0.075871 32.7709   <2e-16 ***
   sex          2.422279   0.288525  8.3954   <2e-16 ***
   minority    -0.037006   0.612455 -0.0604   0.9518    
   repeatgr    -4.408605   0.448615 -9.8271   <2e-16 ***
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   ```
   :::
   :::


a. [2 puntos] Estime la ecuación de calificación usando MCO y efectos fijos de escuela. ¿Qué resuelve este procedimiento?

   *Al incluir efectos fijos a nivel escuela controlamos por características no observadas a nivel escuela. Estas diferencias se incorporan en el modelo como desplazamientos de la ordenada al origen. Este procedimiento no tiene nada que ver con la agrupación de errores.*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   summary(m.mco.ef <- lm(langpost ~ iq_verb + sex + minority + repeatgr + factor(schoolnr),
                          data=data.examen))
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
   
   Call:
   lm(formula = langpost ~ iq_verb + sex + minority + repeatgr + 
       factor(schoolnr), data = data.examen)
   
   Residuals:
        Min       1Q   Median       3Q      Max 
   -24.8792  -3.6779   0.2729   4.0985  19.6755 
   
   Coefficients:
                       Estimate Std. Error t value Pr(>|t|)    
   (Intercept)         12.50218    1.54290   8.103 8.89e-16 ***
   iq_verb              2.25997    0.07093  31.860  < 2e-16 ***
   sex                  2.41900    0.26850   9.009  < 2e-16 ***
   minority             0.23262    0.71306   0.326 0.744280    
   repeatgr            -4.43503    0.40938 -10.834  < 2e-16 ***
   factor(schoolnr)2   -8.29681    2.67144  -3.106 0.001923 ** 
   factor(schoolnr)10  -7.28322    3.06053  -2.380 0.017412 *  
   factor(schoolnr)12  -4.56719    2.06924  -2.207 0.027407 *  
   factor(schoolnr)15  -7.41516    2.54948  -2.909 0.003669 ** 
   factor(schoolnr)16   2.53410    2.54854   0.994 0.320173    
   factor(schoolnr)18  -6.80332    1.81757  -3.743 0.000187 ***
   factor(schoolnr)21  -0.53872    1.99103  -0.271 0.786746    
   factor(schoolnr)24   4.20444    1.81839   2.312 0.020862 *  
   factor(schoolnr)26  -0.36076    1.96000  -0.184 0.853984    
   factor(schoolnr)27  -5.46850    1.87909  -2.910 0.003649 ** 
   factor(schoolnr)29   3.19018    2.35758   1.353 0.176147    
   factor(schoolnr)33  -0.34896    2.84408  -0.123 0.902359    
   factor(schoolnr)35   1.59577    2.08305   0.766 0.443718    
   factor(schoolnr)36  -3.99135    1.83661  -2.173 0.029873 *  
   factor(schoolnr)38   3.05382    1.75466   1.740 0.081931 .  
   factor(schoolnr)40   0.09036    1.67651   0.054 0.957022    
   factor(schoolnr)41  -2.94006    2.13296  -1.378 0.168226    
   factor(schoolnr)42  -4.99799    2.10828  -2.371 0.017845 *  
   factor(schoolnr)44  -1.49855    2.04728  -0.732 0.464268    
   factor(schoolnr)47  -5.73732    2.52469  -2.272 0.023156 *  
   factor(schoolnr)48  -1.83632    2.25583  -0.814 0.415717    
   factor(schoolnr)49  -4.14690    2.41410  -1.718 0.085979 .  
   factor(schoolnr)52  -3.60350    1.87627  -1.921 0.054918 .  
   factor(schoolnr)54   2.91142    1.72017   1.693 0.090691 .  
   factor(schoolnr)55   1.49383    1.73053   0.863 0.388111    
   factor(schoolnr)57   0.79682    1.80093   0.442 0.658210    
   factor(schoolnr)60  -1.79256    1.83760  -0.975 0.329426    
   factor(schoolnr)61   0.39402    1.89130   0.208 0.834990    
   factor(schoolnr)62   2.39062    1.88568   1.268 0.205016    
   factor(schoolnr)65   5.66979    1.85875   3.050 0.002314 ** 
   factor(schoolnr)66   1.09325    2.28956   0.477 0.633058    
   factor(schoolnr)67  -5.59969    1.78486  -3.137 0.001728 ** 
   factor(schoolnr)68  -1.22319    1.90619  -0.642 0.521140    
   factor(schoolnr)76  -1.14607    2.02551  -0.566 0.571577    
   factor(schoolnr)78   0.20198    1.99000   0.101 0.919167    
   factor(schoolnr)79   2.10981    2.10998   1.000 0.317463    
   factor(schoolnr)80   3.31569    1.87290   1.770 0.076810 .  
   factor(schoolnr)86   1.45319    1.81855   0.799 0.424322    
   factor(schoolnr)87  -0.20740    1.88076  -0.110 0.912200    
   factor(schoolnr)88   1.25972    2.35398   0.535 0.592604    
   factor(schoolnr)90   4.45169    2.06677   2.154 0.031356 *  
   factor(schoolnr)94   3.05367    2.05599   1.485 0.137621    
   factor(schoolnr)95  -0.76530    1.94512  -0.393 0.694027    
   factor(schoolnr)97   1.48348    1.98190   0.749 0.454232    
   factor(schoolnr)98  -1.40960    1.89893  -0.742 0.457977    
   factor(schoolnr)101  4.26255    1.83026   2.329 0.019955 *  
   factor(schoolnr)103 -4.54699    3.34961  -1.357 0.174775    
   factor(schoolnr)106 -2.63664    2.33290  -1.130 0.258518    
   factor(schoolnr)107 -4.50332    1.99406  -2.258 0.024023 *  
   factor(schoolnr)108 -1.87255    2.44170  -0.767 0.443222    
   factor(schoolnr)109 -4.40708    1.87743  -2.347 0.018995 *  
   factor(schoolnr)110  3.08759    1.99160   1.550 0.121215    
   factor(schoolnr)111  1.23407    1.94637   0.634 0.526125    
   factor(schoolnr)112 -0.24997    2.67934  -0.093 0.925678    
   factor(schoolnr)115 -0.13189    1.72755  -0.076 0.939152    
   factor(schoolnr)116 -0.96627    1.93966  -0.498 0.618421    
   factor(schoolnr)118 -4.19606    2.42987  -1.727 0.084335 .  
   factor(schoolnr)119 -0.37636    2.21669  -0.170 0.865195    
   factor(schoolnr)121 -3.16182    2.35853  -1.341 0.180196    
   factor(schoolnr)123  2.76021    3.34170   0.826 0.408902    
   factor(schoolnr)124  3.69157    1.89801   1.945 0.051909 .  
   factor(schoolnr)125  1.79787    1.76911   1.016 0.309622    
   factor(schoolnr)130  5.61009    2.16507   2.591 0.009629 ** 
   factor(schoolnr)132  6.28128    1.87651   3.347 0.000830 ***
   factor(schoolnr)136  3.87282    2.03048   1.907 0.056610 .  
   factor(schoolnr)137  4.40378    1.95919   2.248 0.024693 *  
   factor(schoolnr)141  1.14473    1.90727   0.600 0.548441    
   factor(schoolnr)142  5.13270    1.82046   2.819 0.004855 ** 
   factor(schoolnr)147  7.45698    1.85918   4.011 6.26e-05 ***
   factor(schoolnr)148  2.09875    1.74675   1.202 0.229682    
   factor(schoolnr)149  2.53123    1.88015   1.346 0.178351    
   factor(schoolnr)150  1.56758    1.80115   0.870 0.384221    
   factor(schoolnr)151  0.81632    1.83732   0.444 0.656871    
   factor(schoolnr)152  6.20977    1.99227   3.117 0.001852 ** 
   factor(schoolnr)155  4.06995    1.69447   2.402 0.016394 *  
   factor(schoolnr)156  1.03255    2.21442   0.466 0.641061    
   factor(schoolnr)159  2.82606    1.71234   1.650 0.099006 .  
   factor(schoolnr)160  3.52358    1.80662   1.950 0.051261 .  
   factor(schoolnr)161  5.47210    1.70824   3.203 0.001378 ** 
   factor(schoolnr)164  4.69601    1.81787   2.583 0.009853 ** 
   factor(schoolnr)167  3.85744    1.79977   2.143 0.032201 *  
   factor(schoolnr)170  1.92686    1.78368   1.080 0.280143    
   factor(schoolnr)175  4.18579    2.10407   1.989 0.046785 *  
   factor(schoolnr)176  6.59497    1.83757   3.589 0.000339 ***
   factor(schoolnr)177  1.42445    1.99027   0.716 0.474249    
   factor(schoolnr)179 -7.16566    2.82071  -2.540 0.011143 *  
   factor(schoolnr)182  1.78603    2.41396   0.740 0.459456    
   factor(schoolnr)183  0.85895    1.70516   0.504 0.614499    
   factor(schoolnr)184  0.99731    1.74622   0.571 0.567974    
   factor(schoolnr)188 -0.58465    2.19526  -0.266 0.790015    
   factor(schoolnr)189  3.22324    1.98341   1.625 0.104287    
   factor(schoolnr)192 -1.80905    2.55487  -0.708 0.478973    
   factor(schoolnr)193  6.24675    2.21572   2.819 0.004857 ** 
   factor(schoolnr)195  0.62668    1.90527   0.329 0.742247    
   factor(schoolnr)196  2.79306    1.77499   1.574 0.115735    
   factor(schoolnr)197  1.31643    1.90977   0.689 0.490700    
   factor(schoolnr)198 -2.54503    2.66624  -0.955 0.339919    
   factor(schoolnr)199 -4.10925    1.69994  -2.417 0.015719 *  
   factor(schoolnr)204  0.98462    1.81798   0.542 0.588149    
   factor(schoolnr)206  3.79180    2.28001   1.663 0.096446 .  
   factor(schoolnr)209  2.44276    1.83923   1.328 0.184272    
   factor(schoolnr)210 -0.40756    2.02750  -0.201 0.840706    
   factor(schoolnr)212  1.37659    1.80057   0.765 0.444637    
   factor(schoolnr)214 -3.50270    1.85506  -1.888 0.059136 .  
   factor(schoolnr)215  2.42072    2.21308   1.094 0.274155    
   factor(schoolnr)216  3.25053    2.68121   1.212 0.225516    
   factor(schoolnr)217  3.17962    2.55021   1.247 0.212604    
   factor(schoolnr)218  5.14348    1.79490   2.866 0.004203 ** 
   factor(schoolnr)219  5.03571    2.10754   2.389 0.016962 *  
   factor(schoolnr)222  1.27021    1.81917   0.698 0.485106    
   factor(schoolnr)224  0.16178    2.15792   0.075 0.940246    
   factor(schoolnr)226  0.43861    2.55025   0.172 0.863464    
   factor(schoolnr)227  3.11551    1.95897   1.590 0.111895    
   factor(schoolnr)228  7.82086    1.88038   4.159 3.32e-05 ***
   factor(schoolnr)231  4.99594    1.87985   2.658 0.007928 ** 
   factor(schoolnr)233 -3.62877    2.44368  -1.485 0.137700    
   factor(schoolnr)234  4.45798    2.13964   2.084 0.037322 *  
   factor(schoolnr)235  4.18463    2.35443   1.777 0.075653 .  
   factor(schoolnr)237  1.69607    2.06447   0.822 0.411422    
   factor(schoolnr)240 -0.21368    2.21446  -0.096 0.923139    
   factor(schoolnr)241  1.82195    1.83630   0.992 0.321219    
   factor(schoolnr)242  5.11955    2.27771   2.248 0.024698 *  
   factor(schoolnr)243  4.71818    2.54942   1.851 0.064352 .  
   factor(schoolnr)244 -0.81472    2.15710  -0.378 0.705695    
   factor(schoolnr)246  3.94809    2.21570   1.782 0.074911 .  
   factor(schoolnr)249  0.18111    1.81893   0.100 0.920695    
   factor(schoolnr)250 -0.69331    2.06691  -0.335 0.737332    
   factor(schoolnr)252 -0.36370    2.27756  -0.160 0.873140    
   factor(schoolnr)256 -8.25432    2.35336  -3.507 0.000462 ***
   factor(schoolnr)258 -8.24124    2.68111  -3.074 0.002140 ** 
   ---
   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
   
   Residual standard error: 6.186 on 2152 degrees of freedom
   Multiple R-squared:  0.5557,	Adjusted R-squared:  0.528 
   F-statistic: 20.08 on 134 and 2152 DF,  p-value: < 2.2e-16
   ```
   :::
   :::


a. [5 puntos] Estime la ecuación de calificación usando MCO y con errores agrupados a nivel escuela (sin efectos fijos de escuela). ¿Qué resuelve este procedimiento?

   *Al estimar los errores agrupados y robustos a heterocedasticidad se toma en cuenta la correlación que existe en los errores dentro de cada escuela. Los errores agrupados estimados con la opción cluster asumen correlación de errores dentro del grupo, pero no entre grupos. Con respecto a las partes b. y c., el error estándar asociado al tiempo dedicado a la tarea es aproximadamente 20% mayor. Este es un ejemplo típico en el que los errores agrupados se inflan con respecto a los errores de MCO clásicos y los errores robustos.*
    
   *Nota: es posible que los errores agrupados sean menores que los errores de MCO. Para ver eso, considere un modelo simple con datos agrupados de la forma siguiente: $$y_{ig=\alpha+\beta x_{ig}+u_{ig}}$$ donde $x_{ig}$ es un regresor escalar.*
    
   *Se asume que el tamaño promedio de los grupos es $\bar{N}_g$. Moulton (1990) muestra que el error estándar de MCO esta sesgado hacia abajo por una cantidad igual a la raíz de $\tau \approx 1 +\rho_x \rho_u (\bar{N}_g-1)$, donde $\rho_x$ es la correlación dentro de los grupos de $x$ y $\rho_u$ es la correlación dentro de los grupos de los errores. Esto implica que para obtener el error correcto que toma en cuenta la agrupación hay que multiplicar el error de MCO por la raíz de $\tau$. Sin embargo, note que dependiendo del signo y la magnitud de $\rho_x$ y $\rho_u$, la raíz de $\tau$ puede llegar a ser menor que 1 y, por tanto, el error agrupado puede llegar a ser menor que el de MCO. $\tau$ se conoce como el factor de Moulton y puede ser extendido para un modelo más complicado. La intuición funciona de manera similar para un modelo más complicado: todo depende de las correlaciones entre grupos de los regresores y la correlación de los errores.*
    

   ::: {.cell}
   
   ```{.r .cell-code}
   coef_test(m.mco, vcov = "CR1S", cluster = data.examen$schoolnr)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
          Coef. Estimate     SE   t-stat d.f. (Satt) p-val (Satt) Sig.
    (Intercept)   10.940 1.2559   8.7109        95.2       <0.001  ***
        iq_verb    2.486 0.0899  27.6464        94.5       <0.001  ***
            sex    2.422 0.2846   8.5110       110.0       <0.001  ***
       minority   -0.037 0.8547  -0.0433        20.0        0.966     
       repeatgr   -4.409 0.3958 -11.1381        81.8       <0.001  ***
   ```
   :::
   :::


a. [5 puntos] Estime la ecuación de calificación usando MCO, variables indicadoras de escuela y con errores agrupados a nivel escuela. ¿Qué resuelve este procedimiento?

   *Al controlar por características no observadas de las escuelas empleando efectos fijos por escuela y además estimando los errores que toman en cuenta la estructura agrupada de los errores obtenemos un coeficiente estimado de 2.26, pero con un error estándar mayor, 0.0915.*


   ::: {.cell}
   
   ```{.r .cell-code}
   coef_test(m.mco.ef, vcov = "CR1S", cluster = data.examen$schoolnr)
   ```
   
   ::: {.cell-output .cell-output-stdout}
   ```
                  Coef. Estimate     SE  t-stat d.f. (Satt) p-val (Satt) Sig.
            (Intercept)  12.5022 1.1668  10.715        86.2      < 0.001  ***
                iq_verb   2.2600 0.0915  24.695        94.0      < 0.001  ***
                    sex   2.4190 0.2836   8.529       106.5      < 0.001  ***
               minority   0.2326 0.7172   0.324        32.0      0.74778     
               repeatgr  -4.4350 0.4083 -10.862        82.3      < 0.001  ***
      factor(schoolnr)2  -8.2968 0.3795 -21.863        42.8      < 0.001  ***
     factor(schoolnr)10  -7.2832 0.4270 -17.058        32.6      < 0.001  ***
     factor(schoolnr)12  -4.5672 0.4565 -10.005        37.0      < 0.001  ***
     factor(schoolnr)15  -7.4152 0.4211 -17.611        31.8      < 0.001  ***
     factor(schoolnr)16   2.5341 0.4252   5.960        30.8      < 0.001  ***
     factor(schoolnr)18  -6.8033 0.4225 -16.101        30.3      < 0.001  ***
     factor(schoolnr)21  -0.5387 0.4188  -1.286        31.1      0.20787     
     factor(schoolnr)24   4.2044 0.4187  10.042        31.3      < 0.001  ***
     factor(schoolnr)26  -0.3608 0.4253  -0.848        31.4      0.40275     
     factor(schoolnr)27  -5.4685 0.4170 -13.113        30.4      < 0.001  ***
     factor(schoolnr)29   3.1902 0.4536   7.033        34.2      < 0.001  ***
     factor(schoolnr)33  -0.3490 0.4205  -0.830        30.8      0.41305     
     factor(schoolnr)35   1.5958 0.2760   5.783        31.3      < 0.001  ***
     factor(schoolnr)36  -3.9913 0.4175  -9.560        30.5      < 0.001  ***
     factor(schoolnr)38   3.0538 0.4171   7.322        31.0      < 0.001  ***
     factor(schoolnr)40   0.0904 0.4264   0.212        32.0      0.83352     
     factor(schoolnr)41  -2.9401 0.2796 -10.515        33.9      < 0.001  ***
     factor(schoolnr)42  -4.9980 0.4168 -11.992        31.1      < 0.001  ***
     factor(schoolnr)44  -1.4985 0.3315  -4.520        30.9      < 0.001  ***
     factor(schoolnr)47  -5.7373 0.2712 -21.152        81.0      < 0.001  ***
     factor(schoolnr)48  -1.8363 0.2637  -6.963        47.5      < 0.001  ***
     factor(schoolnr)49  -4.1469 0.2037 -20.359        44.2      < 0.001  ***
     factor(schoolnr)52  -3.6035 0.3986  -9.041        34.0      < 0.001  ***
     factor(schoolnr)54   2.9114 0.4389   6.634        33.4      < 0.001  ***
     factor(schoolnr)55   1.4938 0.4230   3.532        32.1      0.00127   **
     factor(schoolnr)57   0.7968 0.3384   2.354        32.6      0.02472    *
     factor(schoolnr)60  -1.7926 0.3208  -5.588        30.8      < 0.001  ***
     factor(schoolnr)61   0.3940 0.3533   1.115        33.0      0.27283     
     factor(schoolnr)62   2.3906 0.4618   5.177        34.7      < 0.001  ***
     factor(schoolnr)65   5.6698 0.4298  13.192        31.4      < 0.001  ***
     factor(schoolnr)66   1.0933 0.4601   2.376        40.1      0.02237    *
     factor(schoolnr)67  -5.5997 0.4214 -13.289        31.4      < 0.001  ***
     factor(schoolnr)68  -1.2232 0.4360  -2.806        32.6      0.00841   **
     factor(schoolnr)76  -1.1461 0.4266  -2.687        31.5      0.01142    *
     factor(schoolnr)78   0.2020 0.4157   0.486        30.7      0.63054     
     factor(schoolnr)79   2.1098 0.4214   5.006        32.2      < 0.001  ***
     factor(schoolnr)80   3.3157 0.3850   8.613        31.6      < 0.001  ***
     factor(schoolnr)86   1.4532 0.4241   3.427        30.8      0.00175   **
     factor(schoolnr)87  -0.2074 0.4152  -0.500        31.8      0.62085     
     factor(schoolnr)88   1.2597 0.4166   3.024        31.5      0.00493   **
     factor(schoolnr)90   4.4517 0.4252  10.470        33.8      < 0.001  ***
     factor(schoolnr)94   3.0537 0.3997   7.641        41.1      < 0.001  ***
     factor(schoolnr)95  -0.7653 0.3455  -2.215        32.8      0.03383    *
     factor(schoolnr)97   1.4835 0.3802   3.902        30.6      < 0.001  ***
     factor(schoolnr)98  -1.4096 0.3875  -3.637        33.2      < 0.001  ***
    factor(schoolnr)101   4.2626 0.3876  10.997        31.6      < 0.001  ***
    factor(schoolnr)103  -4.5470 0.4170 -10.903        85.5      < 0.001  ***
    factor(schoolnr)106  -2.6366 0.2861  -9.217        35.1      < 0.001  ***
    factor(schoolnr)107  -4.5033 0.4439 -10.144        33.2      < 0.001  ***
    factor(schoolnr)108  -1.8726 0.4256  -4.399        30.5      < 0.001  ***
    factor(schoolnr)109  -4.4071 0.2769 -15.917        31.5      < 0.001  ***
    factor(schoolnr)110   3.0876 0.4218   7.319        31.6      < 0.001  ***
    factor(schoolnr)111   1.2341 0.3645   3.386        33.6      0.00182   **
    factor(schoolnr)112  -0.2500 0.4327  -0.578        32.0      0.56748     
    factor(schoolnr)115  -0.1319 0.4179  -0.316        30.4      0.75445     
    factor(schoolnr)116  -0.9663 0.4511  -2.142        37.6      0.03872    *
    factor(schoolnr)118  -4.1961 0.3504 -11.975        31.3      < 0.001  ***
    factor(schoolnr)119  -0.3764 0.4254  -0.885        33.3      0.38261     
    factor(schoolnr)121  -3.1618 0.4610  -6.858        35.0      < 0.001  ***
    factor(schoolnr)123   2.7602 0.2742  10.065        79.0      < 0.001  ***
    factor(schoolnr)124   3.6916 0.3958   9.328        32.1      < 0.001  ***
    factor(schoolnr)125   1.7979 0.3552   5.061        33.4      < 0.001  ***
    factor(schoolnr)130   5.6101 0.4385  12.793        37.4      < 0.001  ***
    factor(schoolnr)132   6.2813 0.4103  15.310        33.7      < 0.001  ***
    factor(schoolnr)136   3.8728 0.4443   8.717        35.6      < 0.001  ***
    factor(schoolnr)137   4.4038 0.4166  10.572        31.2      < 0.001  ***
    factor(schoolnr)141   1.1447 0.4237   2.702        33.3      0.01076    *
    factor(schoolnr)142   5.1327 0.4232  12.128        32.3      < 0.001  ***
    factor(schoolnr)147   7.4570 0.4345  17.162        31.5      < 0.001  ***
    factor(schoolnr)148   2.0988 0.4457   4.709        34.9      < 0.001  ***
    factor(schoolnr)149   2.5312 0.4209   6.014        30.9      < 0.001  ***
    factor(schoolnr)150   1.5676 0.4150   3.777        31.5      < 0.001  ***
    factor(schoolnr)151   0.8163 0.4195   1.946        31.6      0.06058    .
    factor(schoolnr)152   6.2098 0.4255  14.593        32.5      < 0.001  ***
    factor(schoolnr)155   4.0699 0.4161   9.782        31.0      < 0.001  ***
    factor(schoolnr)156   1.0325 0.4308   2.397        31.4      0.02266    *
    factor(schoolnr)159   2.8261 0.4024   7.023        31.8      < 0.001  ***
    factor(schoolnr)160   3.5236 0.2557  13.779        38.4      < 0.001  ***
    factor(schoolnr)161   5.4721 0.3829  14.293        32.8      < 0.001  ***
    factor(schoolnr)164   4.6960 0.4234  11.091        30.5      < 0.001  ***
    factor(schoolnr)167   3.8574 0.4182   9.224        30.6      < 0.001  ***
    factor(schoolnr)170   1.9269 0.4180   4.610        31.1      < 0.001  ***
    factor(schoolnr)175   4.1858 0.4077  10.267        35.7      < 0.001  ***
    factor(schoolnr)176   6.5950 0.4201  15.700        31.0      < 0.001  ***
    factor(schoolnr)177   1.4245 0.4199   3.392        30.5      0.00194   **
    factor(schoolnr)179  -7.1657 0.2125 -33.715        41.9      < 0.001  ***
    factor(schoolnr)182   1.7860 0.2113   8.451        34.4      < 0.001  ***
    factor(schoolnr)183   0.8589 0.3729   2.303        30.5      0.02826    *
    factor(schoolnr)184   0.9973 0.3839   2.598        33.5      0.01383    *
    factor(schoolnr)188  -0.5847 0.3032  -1.928        33.1      0.06242    .
    factor(schoolnr)189   3.2232 0.3779   8.530        32.1      < 0.001  ***
    factor(schoolnr)192  -1.8091 0.4496  -4.023        37.0      < 0.001  ***
    factor(schoolnr)193   6.2468 0.4308  14.501        32.7      < 0.001  ***
    factor(schoolnr)195   0.6267 0.4210   1.488        31.7      0.14650     
    factor(schoolnr)196   2.7931 0.4324   6.459        35.0      < 0.001  ***
    factor(schoolnr)197   1.3164 0.4603   2.860        34.6      0.00713   **
    factor(schoolnr)198  -2.5450 0.3200  -7.953        34.3      < 0.001  ***
    factor(schoolnr)199  -4.1093 0.2785 -14.757        32.7      < 0.001  ***
    factor(schoolnr)204   0.9846 0.4150   2.373        31.2      0.02399    *
    factor(schoolnr)206   3.7918 0.4353   8.710        32.0      < 0.001  ***
    factor(schoolnr)209   2.4428 0.4206   5.807        32.5      < 0.001  ***
    factor(schoolnr)210  -0.4076 0.4372  -0.932        32.1      0.35823     
    factor(schoolnr)212   1.3766 0.4185   3.289        31.1      0.00250   **
    factor(schoolnr)214  -3.5027 0.2946 -11.888        32.9      < 0.001  ***
    factor(schoolnr)215   2.4207 0.4178   5.794        30.4      < 0.001  ***
    factor(schoolnr)216   3.2505 0.4410   7.372        32.5      < 0.001  ***
    factor(schoolnr)217   3.1796 0.4449   7.147        32.2      < 0.001  ***
    factor(schoolnr)218   5.1435 0.3697  13.911        62.4      < 0.001  ***
    factor(schoolnr)219   5.0357 0.4230  11.905        30.3      < 0.001  ***
    factor(schoolnr)222   1.2702 0.4156   3.056        31.8      0.00451   **
    factor(schoolnr)224   0.1618 0.4265   0.379        31.1      0.70703     
    factor(schoolnr)226   0.4386 0.4260   1.030        32.3      0.31078     
    factor(schoolnr)227   3.1155 0.4252   7.328        30.6      < 0.001  ***
    factor(schoolnr)228   7.8209 0.4246  18.420        31.2      < 0.001  ***
    factor(schoolnr)231   4.9959 0.4204  11.882        31.2      < 0.001  ***
    factor(schoolnr)233  -3.6288 0.4254  -8.531        33.5      < 0.001  ***
    factor(schoolnr)234   4.4580 0.3188  13.983        32.4      < 0.001  ***
    factor(schoolnr)235   4.1846 0.4228   9.897        31.6      < 0.001  ***
    factor(schoolnr)237   1.6961 0.4150   4.087        31.1      < 0.001  ***
    factor(schoolnr)240  -0.2137 0.4258  -0.502        31.8      0.61922     
    factor(schoolnr)241   1.8219 0.4171   4.368        30.5      < 0.001  ***
    factor(schoolnr)242   5.1196 0.4241  12.072        31.1      < 0.001  ***
    factor(schoolnr)243   4.7182 0.4237  11.135        32.9      < 0.001  ***
    factor(schoolnr)244  -0.8147 0.4161  -1.958        31.0      0.05931    .
    factor(schoolnr)246   3.9481 0.4392   8.989        32.1      < 0.001  ***
    factor(schoolnr)249   0.1811 0.4219   0.429        31.6      0.67064     
    factor(schoolnr)250  -0.6933 0.4225  -1.641        33.8      0.11005     
    factor(schoolnr)252  -0.3637 0.4199  -0.866        30.4      0.39321     
    factor(schoolnr)256  -8.2543 0.4278 -19.295        31.9      < 0.001  ***
    factor(schoolnr)258  -8.2412 0.4220 -19.529        33.5      < 0.001  ***
   ```
   :::
   :::

