# TITRE
Pour contextualiser un peu, je voulais travailler sur des données une interface R shiny, dans le cadre d'un dispositif IOT. 
Le dispositif envoie des données GPS, mais pas à chaque trame de communication (pour économiser de la batterie). Le problème est que lorsqu'on récupère les données
sur Thingsboard, on récupère une "key", une par une. Dans mon cas j'ai 20 valeurs pour "batterie" et "dectection" mais seulement 12 pour "longitutude" et "lattitude".
Du coup quand je veux faire un `cbind()`, ça marche pô (tailles différentes). Ci-dessous les deux data-frames pour mieux se les représenter. 

### Data frame "batterie" & "detection"
```
        key                  ts    value     key.1 value.1
20 batterie 2021-05-18 09:25:00 12.83871 detection       0
19 batterie 2021-05-18 10:28:11 12.83871 detection       0
18 batterie 2021-05-18 10:32:02 12.83871 detection       1
17 batterie 2021-05-18 10:36:02 12.83871 detection       1
16 batterie 2021-05-18 10:56:12 12.83871 detection       3
15 batterie 2021-05-18 10:56:53 12.83871 detection       1
14 batterie 2021-05-18 11:00:53 12.83871 detection       1
13 batterie 2021-05-18 11:02:24 12.83871 detection       3
12 batterie 2021-05-18 11:04:57 12.83871 detection       3
11 batterie 2021-05-18 11:05:38 12.83871 detection       1
10 batterie 2021-05-18 11:07:57 12.83871 detection       3
9  batterie 2021-05-18 11:09:43 12.83871 detection       3
8  batterie 2021-05-18 11:10:34 12.83871 detection       1
7  batterie 2021-05-18 11:14:34 12.83871 detection       1
6  batterie 2021-05-18 11:17:35 12.83871 detection       3
5  batterie 2021-05-18 11:18:56 12.83871 detection       1
4  batterie 2021-05-18 11:21:04 12.83871 detection       3
3  batterie 2021-05-18 11:22:39 12.83871 detection       0
2  batterie 2021-05-18 11:23:57 12.83871 detection       3
1  batterie 2021-05-18 11:23:57 12.83871 detection       3
```
### Data frame "latitude" & "longitude"
```
       key                  ts    value     key.1 value.1
12 latitude 2021-05-18 09:25:00 43.61932 longitude 3.85712
11 latitude 2021-05-18 10:28:11 43.61932 longitude 3.85712
10 latitude 2021-05-18 10:56:12 43.61932 longitude 3.85712
9  latitude 2021-05-18 11:02:24 43.61932 longitude 3.85712
8  latitude 2021-05-18 11:04:57 43.61932 longitude 3.85712
7  latitude 2021-05-18 11:07:57 43.61932 longitude 3.85712
6  latitude 2021-05-18 11:09:43 43.61932 longitude 3.85712
5  latitude 2021-05-18 11:17:35 43.61932 longitude 3.85712
4  latitude 2021-05-18 11:21:04 43.61997 longitude 3.85674
3  latitude 2021-05-18 11:22:39 43.61932 longitude 3.85712
2  latitude 2021-05-18 11:23:57 43.61932 longitude 3.85712
1  latitude 2021-05-18 11:23:57 43.61932 longitude 3.85712
```
## Fonctionnement
Le programme va boucler sur la df la plus grande `x`, et compare la valeur de `ts` à la ligne `i`. Si elles sont égales il écrit dans une nouvelle df les les deux
df concaténées. Sinon, il remplace les champs manquant par `NA` (not available).
### LA fonction magique
```R
cbindDFsizediff <- function (x, y) {
  j = 1
  c = data.frame()
  for (i in 1:length(x[,'ts'])) {
    if (x[i, 'ts'] == y[j, 'ts']) {
      c = rbind(c, cbind(x[i, ], y[j, ]))
      j = j + 1
    } else {
      key <- c('latitude')
      ts <- c(x[i, 'ts'])
      value <- c(NA)
      temp = data.frame(key, ts, value)
      key.1 <- c('longitude')
      value.1 <- c(NA)
      temp2 = data.frame(key.1, value.1)
      c = rbind(c, cbind(x[i, ], cbind(temp, temp2)))
    }
  }
}
```
Les explications sont un peu abstraites mais le code est assez situationnel, je vous ai donné juste de quoi vous donner un idée pour vous lancer.
### Sortie
```
       key                  ts    value     key.1 value.1      key                  ts    value     key.1 value.1
20 batterie 2021-05-18 09:25:00 12.83871 detection       0 latitude 2021-05-18 09:25:00 43.61932 longitude 3.85712
19 batterie 2021-05-18 10:28:11 12.83871 detection       0 latitude 2021-05-18 10:28:11 43.61932 longitude 3.85712
18 batterie 2021-05-18 10:32:02 12.83871 detection       1 latitude 2021-05-18 10:32:02       NA longitude      NA
17 batterie 2021-05-18 10:36:02 12.83871 detection       1 latitude 2021-05-18 10:36:02       NA longitude      NA
16 batterie 2021-05-18 10:56:12 12.83871 detection       3 latitude 2021-05-18 10:56:12 43.61932 longitude 3.85712
15 batterie 2021-05-18 10:56:53 12.83871 detection       1 latitude 2021-05-18 10:56:53       NA longitude      NA
14 batterie 2021-05-18 11:00:53 12.83871 detection       1 latitude 2021-05-18 11:00:53       NA longitude      NA
13 batterie 2021-05-18 11:02:24 12.83871 detection       3 latitude 2021-05-18 11:02:24 43.61932 longitude 3.85712
12 batterie 2021-05-18 11:04:57 12.83871 detection       3 latitude 2021-05-18 11:04:57 43.61932 longitude 3.85712
11 batterie 2021-05-18 11:05:38 12.83871 detection       1 latitude 2021-05-18 11:05:38       NA longitude      NA
10 batterie 2021-05-18 11:07:57 12.83871 detection       3 latitude 2021-05-18 11:07:57 43.61932 longitude 3.85712
9  batterie 2021-05-18 11:09:43 12.83871 detection       3 latitude 2021-05-18 11:09:43 43.61932 longitude 3.85712
8  batterie 2021-05-18 11:10:34 12.83871 detection       1 latitude 2021-05-18 11:10:34       NA longitude      NA
7  batterie 2021-05-18 11:14:34 12.83871 detection       1 latitude 2021-05-18 11:14:34       NA longitude      NA
6  batterie 2021-05-18 11:17:35 12.83871 detection       3 latitude 2021-05-18 11:17:35 43.61932 longitude 3.85712
5  batterie 2021-05-18 11:18:56 12.83871 detection       1 latitude 2021-05-18 11:18:56       NA longitude      NA
4  batterie 2021-05-18 11:21:04 12.83871 detection       3 latitude 2021-05-18 11:21:04 43.61997 longitude 3.85674
3  batterie 2021-05-18 11:22:39 12.83871 detection       0 latitude 2021-05-18 11:22:39 43.61932 longitude 3.85712
2  batterie 2021-05-18 11:23:57 12.83871 detection       3 latitude 2021-05-18 11:23:57 43.61932 longitude 3.85712
1  batterie 2021-05-18 11:23:57 12.83871 detection       3 latitude 2021-05-18 11:23:57 43.61932 longitude 3.85712
```
