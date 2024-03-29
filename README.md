### **1. Przeszukiwanie zbiorów**

Do przygotowania niniejszego dokumentu wykorzystano następujące biblioteki:

```{r, message = F, results='hide'}
library(knitr)
library(rmarkdown)
library(stringr)
library(viridis)
```

**Zbiory poddane przeszukiwaniu:**

* **Hotel Reservations Dataset** <https://www.kaggle.com/datasets/ahsan81/hotel-reservations-classification-dataset>, autor: Ahsan Raza, licencja: Creative Commons 4.0
* **Success/Fail Dataset from Crunchbase** <https://www.kaggle.com/datasets/yanmaksi/big-startup-secsees-fail-dataset-from-crunchbase>, autor: Yan Maksi, licencja: Community Data License Agreement
* **Wine Quality Data Set** <https://archive.ics.uci.edu/ml/datasets/wine+quality>, źródło: P. Cortez, A. Cerdeira, F. Almeida, T. Matos and J. Reis.*Modeling wine preferences by data mining from physicochemical properties*. In: Decision Support Systems, Elsevier, 47(4):547-553, 2009, licencja: Creative Commons 4.0
* **Heart Failure Prediction** <https://www.kaggle.com/datasets/fedesoriano/heart-failure-prediction>, źródło: dane otwartoźródłowe z miejsc różnych wyszczególnione w powyższym linku, w sekcji *Source*, licencja: Open Database
* **Climate Change: Earth Surface Temperature Data** <https://www.kaggle.com/datasets/berkeleyearth/climate-change-earth-surface-temperature-data>, źródło: Berkeley Earth’s data, licencja: Creative Commons Non-Commercial
* **Netflix Movies and TV Shows** <https://www.kaggle.com/datasets/shivamb/netflix-shows>, autor: Shivam Bansal, licencja: Domena Publiczna

Aby utworzyć tabelkę wektory, określające zawartość kolumn, umieszczono w `data.frame`. Za pomocą funkcji `kable`, będącej częścią pakietu `knitr`, ów `data.frame` można przedstawić w formie prostej, acz eleganckiej tabelki *(nie tabeli)*.
```{r}
nazwa_zbioru <- c("Climate Change: Earth Surface Temperature Data", "Netflix Movies and TV Shows", "Success/Fail Dataset from Crunchbase", "Wine Quality Data Set")
przyczyna_odrzucenia <- c("Szereg czasowy", "Mało danych kategorycznych, niespójny format danych", "Mało danych dot. faktycznych porażek, znaczna dominacja sukcesów", "Brak zmiennych kategorycznych")
df_1 <- data.frame(nazwa_zbioru, przyczyna_odrzucenia)
```
**Zbiory odrzucone:**
```{r}
kable(df_1, col.names = gsub("[_]", " ", names(df_1))) #funkcja gsub pozwala na podmianę "_" na spację w automatycznie podstawianych nazwach kolumn
```

**Zbiory rokujące:**

Dwa dobrze rokujące zbiory danych to Hotel Reservations Dataset (dalej: **HRD**) i Heart Failure Prediction (**HFP**). Obydwa modelują zmienne o dwóch klasach. W HRD są to odpowiednio: *Canceled* i *Not_Canceled*.  W przypadku HFP jest ona wyrażona w sposób binarny (zdrowy-chory).

```{r, echo=FALSE}
#zdecydowałem się w tym  przypadku, ustawić parametr echo=FALSE, bo tabelka powstała identycznie do poprzedniej.
nazwa_zbioru <- c("Hotel Reservations Dataset", "Heart Failure Prediction")
zalety <- c("Dużo obserwacji",  "Dużo zmiennych kategorycznych")
df_2 <- data.frame(nazwa_zbioru, zalety)
kable(df_2, col.names = gsub("[_]", " ", names(df_2)))
```

Zdecydowaną zaletą zbioru HRD jest jego obszerność, ponad 36 tys. obserwacji, w porównaniu do niecałego tysiąca w zbiorze HFP. Nie brakuje w nim również zmiennych kategorycznych, lub takich, które można za takie uznać (szczegóły w sekcji 2). Zdecydowano się więc na analizę tego zbioru.

### **2. Opis wybranego zbioru**

Po pobraniu, załadowano zbiór HRD do dalszego sprawdzenia, korzystając z funkcji `read.csv`:

```{r}
hotel_data <- read.csv("https://github.com/MichalMSlusarski/Analiza-danych-zaliczenie-2/raw/main/hotel_reservations.csv", header = T, sep = ',')
```

Na początku za pomocą funkcji `apply` przyjmującej za argument funkcję `anyNA` sprawdzono, czy w kolumnach znajdują się dane brakujące. Margin = 2 (drugi argument) oznacza działanie na kolumnach. Funkcję tę zadano w argumencie funkcji `paged_table`, której zadaniem jest estetyczne przedstawienie tabeli w formacie html. Dodatkowe atrybuty funkcji `paged_table` ustalane są w sekcji: `{r, ...}`, gdzie `rows.print` ustala liczbę rzędów na jednej stronie.
```{r, rows.print=4}
na_check <- apply(hotel_data, 2, anyNA)
paged_table(as.data.frame(na_check))
```

Dla każdej z kolumn funkcja zwraca wartość `FALSE`. Oznacza to, że w zbiorze nie ma wartości brakujących (co nie oznacza braku wartości błędnych).

Po upewnieniu się, że zbiór nie jest wybrakowany, wywołano funkcję `head` celem zaprezentowania zawartości i struktury zbioru.

**Fragment Hotel Reservations Dataset:**
```{r}
paged_table(head(hotel_data))
```
<br>
Następnie, posługując się funkcją `summary`, zwrócono tabelę z opisem kolumn zbioru. Zwrócony opis przypisano jako `data.frame` do zmiennej `df_3`, którą następnie przekazano do `paged_table`. Jednakże, samo rzutowanie `summary` na `data.frame` skutkuje nieczytelnym wynikiem. Użycie `unclass` zwraca kopię wartości tabeli z usuniętym atrybutem `class`.
```{r}
df_3 <- data.frame(unclass(summary(hotel_data)), check.names = FALSE)
```

**Opis kolumn w zbiorze:**
```{r}
paged_table(df_3)
```

W opisywanym zbiorze znajduje się **36275** obserwacji **19** zmiennych. 

**Zmienne opisane w tabeli to:**

* Booking_ID: numer identyfikacyjny rezerwacji
* no_of_adults: Liczba os. dorosłych
* no_of_children: Liczba dzieci
* no_of_weekend_nights: Liczba nocy weekendowych (Sob-Nd) wchodzących w zakres rezerwacji
* no_of_week_nights: Liczba nocy w tygodniu (Pon-Pt) wchodzących w zakres rezerwacji
* **type_of_meal_plan: Rodzaj zamówionego posiłku**
* **required_car_parking_space: Czy rezerwowano miejsce parkingowe [0 - nie, 1 - tak]**
* **room_type_reserved: Typ pokoju, wg klasyfikacji INN Hotels**
* lead_time: Liczba dni od czasu rezerwacji do czasu planowanego rozpoczęcia wypoczynku
* arrival_year: Rok planowanego rozpoczęcia wypoczynku
* arrival_month: Miesiąc planowanego rozpoczęcia wypoczynku
* arrival_date: Dzień (w miesiącu) planowanego rozpoczęcia wypoczynku
* **market_segment_type: Segment rynku**
* **repeated_guest: Czy klient jest powracający [0 - nie, 1 - tak]**
* no_of_previous_cancellations: Liczba poprzednich odwołanych rezerwacji
* no_of_previous_bookings_not_canceled: Liczba poprzednich rezerwacji nie odwołanych
* avg_price_per_room: Średnia cena za dobę
* no_of_special_requests: Liczba zamówień specjalnych do rezerwacji
* **booking_status: Zmienna modelowana - rezerwacja została odwołana czy nie.**

Dominują zmienne numeryczne dyskretne typu `int`, tylko jedna zmienna - `avg_price_per_room` jest zmiennoprzecinkowa. Pozostałe zmienne są typu `char`. Zmienne dyskretne w zbiorze charakteryzuje niski zakres przyjmowanych wartości *(0-4, 0-10, 0-7...)*, z wyjątkiem zmiennej `lead_time` *(0-443)*. Nie oznacza to jednak niewystępowania danych odstających, na co wskazują duże różnice między medianą, a wartością maksymalną w przypadku niektórych zmiennych. Niemniej, wizualizacja danych odstających dla tak małych przedziałów (np. 0-4), jest raczej mało wnosząca.

##### **2.1 Identyfikacja zmiennych kategorycznych**

Jak wskazują powyższe tabele oraz informacje zawarte w opisie źródłowym, zmienne kategoryczne (**pogrubione**) reprezentowane są zarówno przez wartości typu `numeric`, jak i `char`. Jest to przykład pewnej niekonsekwencji w oznaczeniach. W sumie, zbiór obejmuje 6 zmiennych kategorycznych, 4 wyrażone jako `char` i 2 jako `numeric`.

Kolejnym problemem jest brak określenia przez autora zbioru, jakie i ile poziomów przyjmują zmienne kategoryczne wyrażone za pomocą typu `char`. Żeby się tego dowiedzieć, wywołano funkcję `unique`, która zwraca wszystkie unikatowe wartości z danej kolumny. Zwrócone wyniki umieszczono w `data.frame`. Co istotne, jako że funkcja `unique` zwrócić może różną ilość unikatowych terminów, przed utworzeniem tabeli należy określić jej długość, tutaj wywołując funkcje `max` i `length`. Ilość rzędów w tabeli będzie odpowiadać wywołaniu funkcji `unique`, która zwróci najwięcej wyników. Braki w innych kolumnach uzupełnia funkcja `rep`, za argumenty przyjmująca pusty `char` do podstawienia, jako ilość powtórzeń: różnicę długości maksymalnej i długości danej kolumny, aby nie nadpisać istniejących już danych. Ponownie użyto funkcji `kable` aby uzyskać estetyczną tabelkę. Podczas wywoływania kodu, zauważyłem, że `R` nie sortuje wyników funkcji `unique`, dlatego oplotłem ją w `str_sort` z pakietu `stringr`.
```{r}
u1 <- str_sort(unique(hotel_data$type_of_meal_plan))
u2 <- str_sort(unique(hotel_data$room_type_reserved))
u3 <- str_sort(unique(hotel_data$market_segment_type))
max_u_len <- max(length(u1), length(u2), length(u3))
df_4 <- data.frame(c(u1, rep("", max_u_len - length(u1))),
                    c(u2, rep("", max_u_len - length(u2))),
                    c(u3, rep("", max_u_len - length(u3))))
```

**Poziomy przyjmowane przez zmienne kategoryczne, wyrażone typem `char`:**
```{r}
kable(df_4, col.names = c("Meal plan type", "Reserved room type", "Market segment type"), row.names = 1) #w tym przypadku wywołuję funkcję kable z dodatkowym argumentem row.names=1, aby pojawiły się indeksy rzędów
```
<br>
Wracając do problemu **zapisu zmiennych kategorycznych** - żeby uporządkować tę sytuację, wykorzystano *type casting*. Dla zmiennych typu `char` użyto funkcji `as.factor()`. 
```{r}
hotel_data$type_of_meal_plan <- as.factor(hotel_data$type_of_meal_plan)
hotel_data$room_type_reserved <- as.factor(hotel_data$room_type_reserved)
hotel_data$market_segment_type <- as.factor(hotel_data$market_segment_type)
hotel_data$booking_status <- as.factor(hotel_data$booking_status)
```

W przypadku zmiennych numerycznych, przyjmujących wartość 0-1, także można skorzystać z funkcji `as.factor` ponieważ typ `factor`, to z punktu widzenia języka opatrzona nazwą wartość `int`. 

```{r}
hotel_data$required_car_parking_space <- as.factor(hotel_data$required_car_parking_space)
hotel_data$repeated_guest <- as.factor(hotel_data$repeated_guest)
```
Nowo utworzonym *factorom* przypisano nazwy/etykiety za pomocą funkcji `levels`, jak niżej:
```{r}
levels(hotel_data$required_car_parking_space) #sprawdzam poziomy i ich kolejność, aby uniknąć pomyłki
levels(hotel_data$repeated_guest) 
levels(hotel_data$required_car_parking_space) <- c("F", "T")
levels(hotel_data$repeated_guest) <- c("F", "T")
```

**Zbiór HRD po rzutowaniu zmiennych:**
```{r}
paged_table(head(hotel_data))
```

Widać, że zgodnie z opisem źródłowym, w tabeli znajduje się teraz 6 zmiennych typu `factor`. Przyjmują one w sumie $2 + 2 + 4 + 7 + 5 = 20$ poziomów (bez wliczania zmiennej modelowanej).

<br>
Jak wspomniano na początku, część zmiennych `int` przyjmuje bardzo niewielki zakres wartości. Np. zmienna `no_special_requests` przyjmuje tylko wartości 0, 1, 2, 3, 4, 5. Co ważniejsze, w ponad 54% przypadków przyjmuje wartość 0:
```{r}
ratio <- sum(hotel_data$no_of_special_requests == 0) / 36275
ratio
```
Histogram zmiennej dobrze obrazuje tę dysproporcję:
```{r}
hist(hotel_data$no_of_special_requests, breaks = c(0:5), main = "Histogram zmiennej no_of_special_requests", xlab = "Liczba żądań specjalnych", ylab = "Częstotliwość", col = viridis(5))
```

Przedstawiona sytuacja oznacza *de facto*, że główną informacją kodowaną w tej zmiennej jest fakt *istnienia lub nieistnienia* potrzeb dodatkowych ze strony klienta. Dokładna liczba tych potrzeb, zdaje się mieć znaczenie drugorzędne. Choć nie zachodzi taka potrzeba, istnieje teoretyczne uzasadnienie dla ewentualnego rzutowania tej zmiennej na typ `factor`. Można to zrobić w następujący sposób, wykorzystując funkcję `ifelse`:
```{r} 
hotel_data$no_of_special_requests <- ifelse(hotel_data$no_of_special_requests == 0, "F", "T")
hotel_data$no_of_special_requests <- as.factor(hotel_data$no_of_special_requests)
colnames(hotel_data)[18] <- "special_requests" #należy zmienić nazwę kolumny, bo nie przedstawia już ona wartości liczbowej
```
Argumenty funkcji `ifelse` to kolejno: kolumna, warunek, działanie gdy prawda i działanie gdy fałsz. Mylące mogą być etykiety "F" i "T" użyte na jakby na odwrót. Odnoszą się one jednak, nie do warunku funkcji, tylko do zawartości kolumny: *czy klient zamówił dodatkowe usługi? - 0 oznacza fałsz, każda inna liczba - prawdę*.

##### **2.2 Zmienne wątpliwie istotne**

Kolejnymi interesującymi zmiennymi w zbiorze są `no_of_previous_cancellations` oraz `no_of_previous_bookings_not_canceled`. Zastanawiający jest dobór takich właśnie zmiennych. Myślę, że bardziej logicznym układem byłoby podanie wszystkich rezerwacji razem (np. w zmiennej `no_of_previous_bookings`) z wyszczególnieniem jednej kategorii, gdyż są one dopełniające się.
```{r}
hotel_data$no_of_previous_bookings_not_canceled <- hotel_data$no_of_previous_bookings_not_canceled + hotel_data$no_of_previous_cancellations
colnames(hotel_data)[16] <- "no_of_previous_bookings" #kolumna no_of_previous_bookings_not_canceled zostaje zmieniona na łączną sumę wszystkich
```

Podobnie specyficzny jest sposób, w jaki w zbiorze przedstawiono daty planowanego rozpoczęcia wypoczynku. Trzy zmienne `arrival_date /month /year` mogłyby zostać zapisane jako jedna data lub w ogóle **usunięte ze zbioru**. Modelowanie zmiennej dotyczącej odwołanych rezerwacji, przy użyciu ograniczonych danych czasowych może prowadzić do szukania zależności między rokiem rezerwacji, a liczbą odwołań. Byłoby to duże nadużycie, zważywszy, że w zbiorze znajdują się dane wyłącznie z dwóch lat - 2017 i 2018. Jedyną logiczną, uzasadnioną zmienną z wymienionych, wydaje się być `arrival_month`; wskazuje ona bowiem na istotne z punktu widzenia hotelarstwa sezony (ziomowy, letni, majówka).

Wyraźne istnienie sezonu potwierdzić można graficznie:
```{r}
hist(hotel_data$arrival_month, breaks = c(1:12), main = "Liczba rezerwacji w danym miesiącu", xlab = "Miesiąc", ylab = "Liczba rezerwacji", col = plasma(9))
```

Decydując się na usunięcie dat (poza miesiącem), można by to zrobić w następujący sposób:
```{r}
hotel_data <- hotel_data[,-10] #- arriavl_year
hotel_data <- hotel_data[,-11] #- arrival_date (indeks -1, po przesunięciu powyżej)
```



### **3. Podsumowanie**


Ostatecznie, po uporządkowaniu, zbiór danych prezentuje się tak:
```{r}
paged_table(head(hotel_data))
```
<br>
**W toku przeglądu:**

1. Sprawdzono czy istnieją wartości brakujące
2. Przeanalizowano strukturę zbioru, opisano zmienne
3. Zidentyfikowano zmienne kategoryczne
4. Rzutowano zmienne kategoryczne do typu `factor`
5. Wskazano zmienne, które można rozpatrywać jako kategoryczne (ewentualnie)
6. Dokonano przekształcenia zmiennych numerycznych wielopoziomowych na kategoryczne binarne
7. Wskazano zmienne, które dopełniają się, niosąc tę samą informację
8. Wskazano zmienne, których zapis jest daleki od standardowego, a wartość znikoma

Uporządkowany zbior obejmuje **18** zmiennych, z czego **7** kategorycznych (uwzględniając przetworzone). Zmienne kategoryczne przyjmują łącznie $2 + 2 + 4 + 7 + 5 + 2 = 22$ poziomy, lub $24$ jeśli wliczyć zmienną modelowaną.

Dziękuję za uwagę.
