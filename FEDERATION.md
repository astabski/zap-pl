Federacja
==========

Implementacja ActivityPub w tym projekcie stara się być w miarę możliwości zgodna ze specyfikacją podstawową, oferując jednocześnie szereg usług i funkcji, które zwykle nie są dostarczane przez projekty ActivityPub.

Bezpośrednie wiadomości

Bezpośrednie wiadomości (anf. Direct Messages - DM) są odróżniane od innych prywatnych wiadomości za pomocą flagi `zot:directMessage` (wartość logiczna). Jest to zgodne z tym samym ułatwieniem zapewnianym przez inne projekty w innych przestrzeniach nazw i nie jest poprzedzane w działaniach, aby mogły one potencjalnie być agregowane.

Wydarzenia

Wydarzenia (ang. events) i RSVP są obsługiwane według słownika ActivityStream (AS), z wyjątkiem tego, że nie jest przesyłane `Create/Event`. `Invite/Event` to podstawowa metoda udostępniania wydarzeń. W przypadku zgodności z niektórymi starszymi aplikacjami odpowiedzi RSVP typu `Accept/Event`, `Reject/Event`, `TentativeAccept/Event` i `TentativeReject/Event` są akceptowane jako prawidłowe działania RSVP. Domyślnie wysyłamy `Accept/{Invite/Event}` (i inne odpowiedzi RSVP) według słownika AS. Zdarzenia bez strefy czasowej (np. "wydarzenia całodniowe" lub święta) są wysyłane bez 'Z' w wartości czasu wydarzenia, zgodnie z RFC3339 sekcja 4.3. Wszystkie czasy zdarzeń są podane w UTC, a zdarzenia dostosowane do strefy czasowej są transmitowane w czasie Zulu 'rrrr-mm-ddThh:mm:ssZ'. Opisy wydarzeń są przesyłane za pomocą 'summary' i akceptowane za pomocą podsumowania, tytułu i treści w kolejności występowania. Są one konwertowane wewnętrznie na zwykły tekst, jeśli zawierają HTML. Jeśli 'locatioon' zawiera współrzędne, zazwyczaj generowana jest mapa podczas renderowania.

Tożsamość nomadyczna

Tożsamość nomadyczna opisuje mechanizm, w którym dane przechowywane przez jakąś sieć społecznościową mogą być kopiowane na wiele serwerów w czasie zbliżonym do rzeczywistego, a każdy z tych serwerów może być używany w dowolnym momencie, jeśli ten "główny" serwer będzie niedostępny (tymczasowo lub na stałe). Mechanizmy tworzenia kont nomadycznych i utrzymywania mirrorów nie są obecnie obsługiwane przez ActivityPub, ale są dostępne za pośrednictwem protokołu Nomad. Jednak informacje o instancji są dostarczane do usług ActivityPub, które chcą je obsługiwać w bardziej płynny sposób. W przeciwnym razie, np.  wiadomości DM  mogą wymagać wysłania do wszystkich wystąpień aktora, aby upewnić się, że aktor nomadyczny je otrzyma. Istnieje wiele innych przykładów.

Rekord aktora dla kont nomadycznych zawiera właściwość 'copiedTo', która jest identyczna z właściwością 'movedTo' Mastodona. Usługi ActivityPub, które chcą wspierać zgodność z tym mechanizmem, powinny zapewnić, że każda instancja nomadyczna jest "składana" w pojedynczą tożsamość federacyjną, tak aby wiadomości od tego samego aktora w różnych instancjach były rozpoznawane jako należące do tego samego autora. Dodatkowo wszelkie wpisy wysyłane na konto nomadyczne powinny być wysyłane do wszystkich instancji, a prywatne dane powinny być dostępne dla wszystkich instancji aktorów. Obejmuje to dane historyczne i wpisy, ponieważ w dowolnym momencie można tworzyć nowe instancje.

Grupy

Grupy mogą być publiczne lub prywatne. Początkowy wątek rozpoczynający wpis dla grupy jest wysyłany za pomocą wiadomości DM do grupy i powinien być to jedyny odbiorcą wiadomości DM. Pomaga to zachować prywatność grup i jest metodą publikowania dostępną dla większości oprogramowania ActivityPub, w przeciwieństwie do tagów Bang (!groupname), które nie mają szerokiego wsparcia i normalnych @wzmianek, które mogą powodować problemy z prywatnością i związane z tym komplikacje. Taki wpis zostanie przekształcony na osadzony wpis autorstwa aktora grupy (i przypisany do oryginalnego aktora) i ponownie wysłany do wszystkich członków. Kontynuacje i odpowiedzi na wpisy w grupie korzystają z normalnych metod federacji. Typ aktora to 'Groups' i można go śledzić za pomocą opcji `Follow/Group` *lub* `Join/Group`, a zaprzestanie obserwowania za pomocą opcji `Undo/Follow` *lub* `Leave/Group`.

Aktualizacja: od czerwca 2021 komentarze w grupie są wysyłane normalnymi metodami, ale dodatkowa aktywność Announce jest teraz wysyłane do połączeń ActivityPub, dzięki czemu komentarze będą widoczne na domowej osi czasu w domach w witrynach mikroblogów (Mastodon itp.). Witryny konwersacyjne lub makroblogi z działającym widokiem konwersacji powinny filtrować lub ukrywać ten zbędny wpis.


Aktualizacja: od 2021-04-08 @wzmianki są teraz dozwolone do publikowania w grupach publicznych i moderowanych, ale nie można ich publikować w grupach ograniczonych lub prywatnych. Właściciel grupy może zmienić to zachowanie zgodnie z potrzebami na podstawie oczekiwań grupy w zakresie bezpieczeństwa i prywatności. Czaty (i wpisy wall-to-wall) są nadal zalecanymi metodami publikowania w grupach, ponieważ można ich używać w dowolnych grupach bez konieczności pamiętania, które są publiczne, a które prywatne; i które mogły zezwolić lub zabronić publikowania za pośrednictwem wzmianek.

Aktualizacja: od 2021-04-30. Będziemy obsługiwać przychodzące wpisy grupowe w postaci Create/Note/Collection or Add/Note/Collection, gdzie docelowa Collection|orderedCollection to grupowa skrzynka nadawcza lub "ściana". Powinno to odpowiadać zachowaniu projektu Smithereen i jest udokumentowane w [Fediverse Enhancement Proposal FEP-400e](https://git.activitypub.dev/ActivityPubDev/Fediverse-Enhancement-Proposals/src/branch/main/feps/fep-400e.md). Zapewniamy również tę strukturę automatycznie w wychodzących wpisach grupowych, jeśli rekord aktora zawiera słowo 'wall' (`sm:wall`); i dodajemy tag wzmianki w w aktywności, aby obsługiwać implementacje group/relay, które są wyzwalane tylko w przypadku wzmianek.

Komentarze

Ten projekt zapewnia kontrolę uprawnień i moderację komentarzy. Domyślnie komentarze są akceptowane tylko z istniejących połączeń. Może to zostać zmienione przez włściciela kanału. Inne strony MOGĄ wykorzystać `zot:commentPolicy` (string) jako przewodnik, jeśli nie chcą udostępniać możliwości komentowania, jeśli z góry wiadomo, że zostaną odrzucone. Jeśli komentarz nie jest dozwolony, zostanie wysłana aktywność Reject/Note. Obecnie nie ma odpowiedzi na moderowane treści, ale prawdopodobnie będą one również reprezentowane przez Reject/Note.

'commentPolicy' może być dowolną wartością z nastęþującej listy:

- 'authenticated' - odpowiada typowym uprawnieniom ActivityPub

- 'contacts' - pasuje do zatwierdzonych obserwujących

- 'any connections' - pasuje do dowolnych obserwującym bez względu na aprobatę

- 'site: foobar.com' - pasuje do dowolnego aktora lub wystąpienia klonu z "foobar.com"

- 'public' - pasuje do wszystkich, może wymagać moderacji, jeśli sieć nie jest znana

- 'self' - pasuje tylko do autora aktywności

- 'until=2001-01-01T00:00Z' - komentarze są zamykane po podanym terminie. Można to podać samodzielnie lub dołączyć do dowolnego innego ciągu commentPolicy, poprzedzając go spacją; na przykład 'contacts until=2001-01-01T00:00Z'.


Prywatne media

Dostęp do mediów prywatnych MOŻE być możliwy za pomocą protokołu OCAP lub OpenWebAuth. Bearcapy są obsługiwane, ale nie są generowane.


System uprawnień

System uprawnień Nomad ma już wieloletnią historię funkcjonowania i różni się od typowego projektu ActivityPub. Uważamy 'Follow' za anty-wzorzec, który zachęca do pseudoanonimowego stalkingu. Aktywność 'Follow'  wykonywane przez aktora w tym projekcie zazwyczaj oznacza, że aktor wysyła aktywności do odbiorcy. Może to również nadawać inne uprawnienia. 'Accept/Follow' zwykle zapewnia uprawnienia do otrzymywania treści od wskazanego aktora, w zależności od jego ustawień prywatności.


Model dostarczania

Ten projekt wykorzystuje system przekazywania, który był wykorzystywany w projektach takich jak Friendica, Diaspora i Hubzilla, a który próbuje utrzymać całe konwersacje w stanie nienaruszonym, gdzie inicjator konwersacji kontroluje dystrybucję prywatności. Nie jest to gwarantowane w przypadku ActivityPub, w którym członkowie konwersacji mogą wysyłać kopie wiadomości do innych osob, które nie znajdowały się w początkowej grupie prywatności. Zachęcamy projekty, aby nie dopuszczać dodatkowych odbiorców lub nie uwzględniać własnych obserwujących w followupach. Kontynuacje POWINNY mieć jednego odbiorcę — właściciela konwersacji lub inicjatora i są przekazywane przez właściciela pozostałym członkom konwersacji. Zwykle wymaga to użycia podpisów LD, ale może być również dostępne poprzez uwierzytelnione pobieranie aktywności przy użyciu podpisów HTTP


Treść

Treść może być bogata w multimedia i ładnie renderować się jako HTML. Do edycji treści i jej przechowywania używa się 'multicode'. Multicode konglomerat znaczników html, markdown i bbcode. Interpreter multicode pozwala przełączać się między nimi lub łączyć je do woli. Treść nie jest ograniczona co do długości i autorzy MOGĄ wykorzystać maksymalnie 24 MB pamięci. W praktyce konwersja znaczników ogranicza efektywną długość do około 200 KB, a domyślna "maksymalna długość importowanej treści" z innych witryn to 200 KB. Można to zmienić w zależności od witryny, ale jest to rzadko stosowane. Notatka może zawierać jeden lub więcej obrazów, innych mediów i łączy. Oczyszczanie HTML przede wszystkim usuwa javascript i elementy iframe oraz pozwala na przejście do większości legalnych znaczników i stylów CSS. Obrazy w tekście są dodawane również jako załączniki, z myślą o komunikacji z Mastodon (w którym usuwa się większość HTML), ale pozostają w źródle HTML. Podczas importowania treści z innych witryn, jeśli treść zawiera załącznik w postaci obrazu, treść jest skanowana w celu sprawdzenia, czy znacznik linki (a) lub (img) zawierający ten obraz jest już obecny w kodzie HTML. Jeśli nie, do tekstu jest dodawany znacznik obrazu na końcu treści przychodzącej. Ten mechanizm obsługuje wiele obrazów.

"Podsumowanie" z Mastodona ('summary') nie wymaga żadnego specjalnego traktowania, więc 'summary' może być używane zgodnie z jego przeznaczeniem jako podsumowanie treści. Honorowane jest 'sensitive' Mastodona i powoduje wywołanie dowolnych mechanizmów wybranych przez użytkownika do obsługi tego typu treści. Domyślnie obrazy są zasłonięte i sa wyśietlane za pomacą odnośnika "Kliknij, aby wyświetlić". Wrażliwy tekst ('sensitive') nie jest traktowany specjalnie, ale może być zasłonięty za pomocą wtyczki NSFW lub filtrowany w połączeniu na podstawie dopasowania ciągów, tagów, wzorców, języków lub innych kryteriów.


Edycja

Edytowane wpisy i komentarze są wysyłane w akcji Update/Note wraz ze znacznikiem czasu 'updated' oraz oryginalnym znacznikiem czasu 'published'.


Ogłaszanie

Aktywności ogłaszania ('announce')i przekazywania ('relay') wykorzystują dwa mechanizmy. Oprócz aktywności ogłaszania generowana jest nowa wiadomość z osadzoną wyrenderowaną treścią jako treści wiadomości. Wiadomość ta może (powinna) zawierać dodatkowy komentarz w celu zachowania zgodności z przepisami dozwolonego użytku prawa autorskiego. Innym powodem jest używanie przez nas uprawnień do komentowania. Komentarze do aktywności 'announce' są wysyłane do autora (który zazwyczaj akceptuje komentarze tylko ze swoich połączeń). Komentarz do osadzonych przekierowań jest wysyłany do nadawcy. Ta różnica w zachowaniu umożliwia grupom poprawne działanie w obecności uprawnień do komentowania.

Dyskusja 2021-04-17): W świecie poczty e-mail ten rodzaj konfliktu jest rozwiązywany przez użycie nagłówka 'reply-to' (np. w tym przypadku odpowiedz grupie, a nie autorowi) oraz koncepcji 'sender', która jest inna niż 'from' (autor). Niedługo będziemy modelować coś takiego po raz pierwszy w ActivityPub za pomocą 'replyTo'. Jeśli zobaczysz 'replyTo' w aktywności, oznacza to, że odpowiedzi POWINNY trafiać na ten adres, a nie do skrzynki odbiorczej autora. Najpierw wdrożymy to i przedstawimy propozycję 'sender', jeśli to się sprawdzi. Jeśli wystarczająca liczba projektów wspiera te konstrukcje, możemy wyeliminować mechanizmy wielokrotnego przekazywania, a tym samym uczynić ActivityPub bardziej wszechstronnym, jeśli chodzi o komunikację organizacyjną i grupową. Naszym głównym przypadkiem użycia słowa 'sender' jest podanie źródła ActivityPub wiadomości, która została całkowicie zaimportowana z innego systemu (np. Diaspora lub ze źródła RSS). W tym przypadku ustawimy 'attributedTo' na zdalną tożsamość, która jest autorem treści, a 'sender' na osobę, która opublikowała ją w ActivityPub.

Reakcje emoji

Zakładamy, że odpowiedź na wiadomość zawierająca dokładnie jedną emotikonę i brak innego tekstu lub znaczników jest reakcją emoji. Wskazujemy ten stan wewnętrznie przy odbiorze, ale nie robimy nic, aby zidentyfikować go konkretnie dalszym odbiorcom.

Niestandardowe emotikony Mastodona

Niestandardowe emotikony Mastodona są obsługiwane tylko w przypadku treści wpisów i nie są uważane za reakcje emotikonów. Wyświetlane nazwy i tytuły wiadomości (pole "name" ActiveStreams) są uważane za pola tekstowe, a osadzone obrazy (mechanizm niestandardowych emotikonów) nie są obsługiwane w tych lokalizacjach.

Wzmianki i wzmianki prywatne

Domyślny format wzmianki to "@Wyświetlana nazwa", ale dostępne są inne opcje, takie jak "@nazwa użytkownika" i oba "@Wyświetlana nazwa (nazwa użytkownika)<'. Wzmianki mogą również zawierać znak wykrzyknika, na przykład "@!Wyświetlana nazwa”. Oznacza to, że wiadomość bezpośrednia lub prywatna do "Wyświetlanej nazwy” i adresowanie oraz prywatność poczty zostały odpowiednio zmienione. Wszystkie treści przychodzące i wychodzące w strumieniu są przepisywane, aby wyświetlać wzmianki w wybranym przez stylu. Może zaistnieć potrzeba przyjęcia tego mechanizmu przez inne projekty, aby zapewnić spójność, ponieważ po obu stronach debaty istnieją silne odczucia dotyczące tego, która forma powinna być powszechna w fediverse.

Powiadomienia o komentarzach (Mastodon)

Nasze projekty wysyłają powiadomienia o komentarzach, jeśli ktoś odpowie na Twój wpis lub na wpis z którym wcześniej w jakiś sposób się kontaktowałeś. Mogą być również wysłane powiadomienia typu "wzmianka", jeśli wspomniano o Tobie w jakimś wpisie. Różni się to od Mastodona, który wydaje się w ogóle nie obsługiwać powiadomień o komentarzach i zapewnia tylko powiadomienia o wzmiankach. Z tego powodu użytkownicy Mastodon zazwyczaj nie otrzymują powiadomienia, chyba że autor, któremu odpowiadają, jest wymieniony w poście. Podajemy tę wzmiankę w polu 'tag' wygenerowanej aktywności, ale zwykle nie umieszczamy jej w treści wiadomości, ponieważ w rzeczywistości nie wymagamy wzmianek, które zostały utworzone wyłącznie w celu wyzwolenia powiadomienia o komentarzu.


Zakończenie rozmowy

(2021-04-17) Łatwo jest pobrać brakujące fragmenty konwersacji idąc "w górę", ale nie ma uzgodnionej metody pobierania pełnej rozmowy z punktu widzenia aktora źródłowego, a pobieranie w górę zapewnia tylko jedną gałąź konwersacji, a nie całe drzewo. Podajemy 'context' jako adres URL do kolekcji zawierającej całą konwersację (wszystkie znane gałęzie) widziane przez jej twórcę. Wymaga to specjalnego traktowania i jest bardzo podobne do 'ostatus:conversation', ponieważ jeśli kontekst jest obecny, musi zostać odtworzony w potomkach konwersacji. Nadal obsługujemy 'ostatus:conversation', ale użycie jest przestarzałe. Nie używamy 'replies' do osiągnięcia tych samych celów, ponieważ 'replies' dotyczą tylko bezpośrednich potomków w dowolnym punkcie drzewa konwersacji.


