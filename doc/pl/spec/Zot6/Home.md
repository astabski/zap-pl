## Zot/6 aka Zot 2018

Dokument opisuje wersję 6 protokołu Zot. Jest to żywy dokument, ostatnio zmodyfikowany 2018-09-14.

Zot to **WebMTA**, który zapewnia zdecentralizowaną tożsamość i protokół komunikacyjny przy użyciu HTTPS/JSON.

Wcześniejsze zmiany protokołu zot dotyczyły tworzenia tożsamości nomadycznych i uwierzytelniania międzydomenowego, aby umożliwić zdecentralizowaną sieć z funkcjami rywalizującymi z dużymi scentralizowanymi dostawcami.

Zot/6 opiera się na tych koncepcjach i usprawnia wiele interakcji, wykorzystując wnioski wyciągnięte przez dziesięciolecia budowania zdecentralizowanych systemów.

Implementacja referencyjna (aplikacja społecznościowa) jest dostępna pod adresem https://codeberg.org/zot/zap ; implementacja referencyjna nie jest jeszcze w 100% zgodna ze specyfikacją lub kompletna, ale może być używana do podstawowych testów zgodności i celów programistycznych.

### Różnice w stosunku do wcześniejszych wersji Zot

1. Usprawniona komunikacja przy użyciu transferu bezpośredniego (push). Wcześniejsze wersje wykorzystywały model dostawy 'powiadom/odbierz'.
2. Komponent uwierzytelniania (Magic-Auth) został wyodrębniony do osobnej i niezależnej specyfikacji ['OpenWebAuth'](spec/OpenWebAuth/Home.md).
3. Uwzględnienie ActivityStreams (JSON-LD) jako obsługiwanej (podstawowej) serializacji.
4. Porzucenie wymagań dotyczących implementacji w celu obsługi serializacji wtórnych.
5. Przeniesienie wykrywania usługi do nagłówka HTTP Accept w oparciu o punkty końcowe usługi, gdzie można wybrać różne reprezentacje przez zmianę nagłówka Accept żądania HTTPS.
6. Oddzielenie "portable-id" od używanego algorytmu podpisu.

### Różnice pomiędzy Zot a innymi usługami WebMTA

Zot różni się architektonicznie od innych protokołów "komunikacji społecznościowej" opartych na HTTPS, takich jak Ostatus, ActivityPub czy Diaspora. Podstawowe różnice to:

1. Obsługa tożsamości nomadycznej, gdzie tożsamość nie jest trwale powiązana z serwerem DNS.
2. Agenty MUA zbudowane na bazie Zot są w stanie używać i wyrażać szczegółowe uprawnienia międzydomenowe.
3. Negocjacja szyfrowania dla dodatkowej ochrony wiadomości oprócz HTTPS.
4. Zot nie definiuje bezwzględnego formatu ładunku treści. Implementacje MUSZĄ obsługiwać ActivityStreams. Dodatkowe typy komunikatów i formaty serializacji MOGĄ zapewniać ulepszenia specyficzne dla dostawcy.
5. Federacja między innymi protokołami WebMTA nie jest utrudniona przez niepotrzebne ograniczenia komunikacji stron trzecich. Wiadomości z niekompatybilnych systemów mogą być przekazywane do innych witryn, które nie obsługują protokołu 3-ciej strony.
6. Dostarczane są szczegółowe raporty dotyczące dostarczania na potrzeby audytu i rozwiązywania problemów; co ma kluczowe znaczenie w rozproszonej usłudze komunikacyjnej.

## Podstawowe pojęcia

### Tożsamość 

W Zot/6, podstawowym atrybutem systemu jest zdecentralizowana tożsamość. Jest to inne podejście niż (i często niezgodne) z typowymi projektami komunikacji zdecentralizowanej lub P2P, w których kluczowym elementem jest przesyłana wiadomość, a tożsamość jest jedynie przypisaniem do wiadomości.

Tożsamość składa się z dwóch podstawowych elementów:

1. "oświadczenia" dotyczącego tożsamości, które jest łańcuchem tekstowym.
2. klucza opartego na RSA, za pomocą którego można zweryfikować tę tożsamość.

Zostaną one szczegółowo opisane później. "Ważna tożsamość" to tożsamość, która została zweryfikowana przy użyciu technik kryptograficznych z kluczem publicznym. Do czasu zweryfikowania tożsamość jest "zarejestrowana" (niezweryfikowana). "Nieprawidłowa tożsamość" to tożsamość, której weryfikacja nie powiodła się lub została unieważniona.

W Zot/6 mogą istnieć deklarowane (niezweryfikowane) tożsamości i MOGĄ mieć uprawnienia do wykonywania operacji. Pozwala to na zaistnienie komunikacji krzyżowej między Zot/6 a innymi systemami komunikacyjnymi z różniącymi się koncepcyjnie tożsamościami i różnymi metodami walidacji.

Implementacja MOŻE zdecydować o zablokowaniu niezweryfikowanych tożsamości. Może to znacznie ograniczyć możliwość interakcji z obcymi systemami, protokołami i sieciami.

Nieprawidłowe tożsamości NIE MOGĄ mieć żadnych uprawnień. Z definicji są fałszerstwami lub zostały unieważnione.

### Kanały

W innych systemach kanały są zazwyczaj określane jako "użytkownicy", ale w Zot/6 pojęcie to jest abstrakcyjne i może przybierać inne znaczenia. Kanał jest reprezentowany przez tożsamość.

### Lokalizacja i niezależność lokalizacji

Lokalizacja to tożsamość (kanał) i podlega wszystkim zasadom związanym z tożsamością. Oświadczenie tożsamości to zazwyczaj w pełni kwalifikowany "podstawowy" adres URL usługi sieciowej świadczącej usługi Zot/6, pisany małymi literami, z usuniętymi końcowymi ukośnikami. Międzynarodowe nazwy domen są konwertowane na "punycode" ([RFC 3492](https://datatracker.ietf.org/doc/html/rfc3492)).

"Tożsamość niezależna od lokalizacji" (*ang. location independent identity*) to ważna tożsamość kanału połączona z prawidłową lokalizacją, z dodatkową właściwością, że para kanał-lokalizacja została zweryfikowana przy użyciu klucza tożsamości kanału. Mówiąc potocznie, użytkownik podpisuje swoją stronę internetową własnym kluczem.

Model niezależny od lokalizacji umożliwia mobilność kanału do różnych a nawet wielu lokalizacji, które wszystkie mogą być ważne jednocześnie. To jest określane gdzie indziej jako "tożsamość nomadyczna".

Mówiąc prościej, koncepcja tożsamości niezależnej od lokalizacjii, oznacza w podstawowym znaczeniu, że kanał może pojawić się w dowolnym miejscu w dowolnym czasie, o ile tożsamość niezależna od lokalizacji jest ważna.


### Połączone tożsamości

Można powiązać dwie lub więcej tożsamości (kanałów). Mówiąc prościej, pozwala nam to zdefiniować trwały token, który odnosi się do tożsamości, której ciąg oświadczenia lub klucz publiczny (lub oba na raz) uległy zmianie lub są scalane z innego systemu. W celu powiązania tożsamości, oprócz wymogu, że obie tożsamości MUSZĄ zostać potwierdzone, każda tożsamość MUSI podpisać drugą połączoną tożsamość i wszystkie te podpisy MUSZĄ być ważne.

W przypadku zapewniania kontrolowanego dostępu do zasobów witryny, wszystkie powiązane tożsamości POWINNY zostać sfałdowane lub zredukowane do wspólnego identyfikatora, aby próba uzyskania dostępu do chronionego zasobu przez dowolną połączoną tożsamość zakończyła się sukcesem dla zasobu, który został udostępniony dla wszystkch jego połączonych tożsamości.


### Uwagi dotyczące nomadyczności

Niezależne od lokalizacji, właściwości Zot/6 nakładają dodatkowe wymagania na systemy komunikacyjne. W przypadku wiadomości wychodzących należy podjąć próbę dostarczenia wiadomości do każdej połączonej i nomadycznej lokalizacji powiązanej z tożsamością dostarczającą wiadomość. Usługa MOŻE usunąć prawidłową lokalizację połączoną lub nomadyczną z listy celów dostawy, jeśli lokalizacja jest oznaczona jako "martwa" lub "nieosiągalna". Lokalizacja MOŻE być oznaczona jako "martwa" lub "nieosiągalna" z wielu powodów; ogólnie przez brak komunikacji przez ponad 30 dni lub za pomocą ręcznych sposobów, jeśli administrator witryny ma powody, by sądzić, że lokalizacja została zamknięta na stałe. Witryna POWINNA zachować zapis lokalizacji i automatycznie usunąć oznaczenie "martwy" lub "nieosiągalny", jeśli w przyszłości z tej lokalizacji zostanie zainicjowana prawidłowa komunikacja.


## Transport i podstawowy protokół

Komunikacja z Zot/6 odbywa się głównie poprzez żądania https JSON. Żądania są podpisywane przez "nadawcę" przy użyciu protokołu HTTP-Signatures (draft-cavage-http-signatures-xx). Jeśli ładunek jest obecny, podpisane nagłówki MUSZĄ zawierać nagłówek Digest używający SHA-256 lub SHA-512, jak określono w RFC5843.

W przypadku żądań HTTP, które nie zawierają ładunku, wszystkie nagłówki MOGĄ być podpisane, ale co najmniej jeden nagłówek MUSI być podpisany.

Podpisane żądania MOGĄ zostać odrzucone, jeśli:

- Weryfikacja skrótu nie powiodła się'
- Żądanie zawiera ładunek JSON, a treść nie jest podpisana;
- Nie powiodło się pobieranie klucza publicznego.

Klucze publiczne są pobierane przy użyciu Zot/6 Discovery we właściwości 'keyId' danych podpisu i pobierając klucz publiczny z tego dokumentu. Zot Discovery zostało opisane w kolejnym rozdziale.

Odbiorcy weryfikujący te żądania POWINNI sprawdzić, czy osoba podpisująca ma władztwo nad przekazanymi danymi, z racji bycia autorem lub pośrednikiem (nadawcą).
