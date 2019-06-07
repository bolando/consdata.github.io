---
layout:    post
title:     "Angular - detekcja zmian strategią onPush"
date:      2019-05-29 07:00:00 +0100
published: false
author:    kgabara
tags:
    - angular
    - frontend
---
Każda aplikacja rozwijana odpowiednio długi czas może rozrosnąć się do ogromnych rozmiarów, a konkretniej do sporej liczby komponentów, jeżeli mówimy o aplikacji frontendowej pisanej z wykorzystaniem Angulara. Duża liczba komponentów może spowodować, że w pewnym momencie nasza aplikacja przestanie spełniać oczekiwania odnośnie wydajności. W takim momencie powinniśmy pomyśleć nad możliwościami naprawy tego problemu. Jednym z rozwiązań mogłaby być zmiana domyślnej strategii detekcji zmian.


## Detekcja zmian

Załóżmy, że mamy przed sobą kod aplikacji odpowiedzialnej za zarządzanie hodowlą zwierzątek. Przykładowym komponentem odpowiedzialnym za wyświetlanie informacji o krówkach byłby cow-run.component, czyli wybieg krówek, który przekazuje obiekt pojedyńczej krówki do cow.component. Z drugiej strony mamy pig-run.component, który spełnia te same założenia co komponent krówek. Przykładowe drzewo komponentów mogłoby wyglądać tak:

//tutaj zdjęcie komponentów

Angular dla każdego komponentu tworzy odpowiadający jemu (komponentowi) ChangeDetector. Uzupełnimy więc drzewo komponentów, żeby bardziej zobrazować tę sytuację.

// drugie zdjecie z changedetectorami

Widząc jak to wygląda, możemy przejść dalej: czyli jak to działa.

### Jak to działa?
Domyślnie ChangeDetector nasłuchuje na każdą zmianę stanu aplikacji - zmianę inputów, zmianę modelu prezentowanego na templatce, wywołania asynchroniczne, zdarzenia DOM, interwały. Każda taka zmiana powoduje porównanie obecnie prezentowanych w drzewie DOM wartości do tych, które przechowuje komponent - w momencie wykrycia różnic komponent oznaczany jest jako "brudny". Następnie dokonywana jest projekcja modelu na drzewo DOM, czyli faktyczne zaktualizowanie widoku.


Detekcja zmian w każdym świeżo utworzonym komponencie ustawiona jest na wartość ChangeDetectionStrategy.Default, co przekłada się na detekcję zmian strategią CheckAlways. Strategia ta sprawia, że podczas KAŻDEGO zmianu stanu aplikacji sprawdzane jest całe drzewo komponentów.

// zdjecie obrazujace sprawdzanie komponentow
## Strategia onPush

### W czym może pomóc?
 Jeżeli każdy komponent używałby domyślnej strategii detekcji zmian, mechanizm ten choć bardzo wydajny, mógłby wprowadzać opóźnienie w działaniu całej aplikacji, gdyż przy każdym zdarzeniu asynchronicznym lub zmianie inputów skanowane byłoby całe drzewo komponentów 


### Jak używać?
Zmiana inputow przez przekazanie referencji do nowego/innego obiektu, używanie asyncPipe przy operowaniu na obiektach typu Observable. AsyncPipe w swoim kodzie źródłowym podczas subskrypcji do przekazywanego obiektu Observable wykorzystuje referencje do mechanizmu detekcji zmian danego komponentu, aby powiadomić o "zabrudzeniu", czyli przekazać informację o prawdopodobnej potrzebie odświeżenia widoku danego komponentu. 

```typescript
        _updateLatestValue(async, value) {
            if (async === this._obj) {
                this._latestValue = value;
                this._ref.markForCheck();
            }
        }
```
        
Fragment kodu opisujący działanie asyncPipe, czyli dlaczego to działa :)


### Na co należy uważać?
Przekazywanie zmian w inpucie typem prostym - nie triggeruje detekcji zmian

## Przejęcie kontroli nad mechanizmem detekcji zmian
Po zmianie domyślnej strategii detekcji zmian na onPush mamy również możliwosć 
## Kilka słów na zakończenie 
