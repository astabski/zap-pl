## Treść

Niektóre aplikacje i implementacje Zot są w stanie obsługiwać złożone struktury treści, w tym aplikacje osadzone, treści identyfikujące tożsamość i uwierzytelnione łącza. Nie wymaga się, aby wszystkie implementacje obsługiwały tego typu treści
lub mogły być w stanie obsługiwać te typy treści. Ponadto formaty serializacji danych obsługiwane przez implementację mogą wpływać na możliwość pełnego reprezentowania bogatej zawartości identyfikującej tożsamość.

### ActivityStreams

Implementacja, która obsługuje tylko ActivityStreams2, otrzymuje treść "message" jako typ "Artykuł", zawierającą ogólne renderowanie HTML treści źródłowej w elementach "content" lub "contentMap". To ogólne renderowanie jest odpowiednie dla każdego
obserwatora, a niektóre łącza HTML mogą być niedostępne ze względu na wymagania dotyczące uprawnień i uwierzytelniania.

Źródło renderowanego kodu HTML jest dostępne w elemencie "source". Implementacje, które chcą obsługiwać zawartość identyfikującą tożsamość i uwierzytelnione łącza, powinny używać tej zawartości źródłowej (zwłaszcza jeśli jest to typ "text/x-zot-bbcode") do dynamicznego generowania renderowania HTML, które jest specyficzne dla bieżącego obserwatora.

Dokładny zestaw funkcji obsługiwany przez implementację i lokalne filtrowanie zabezpieczeń MOŻE skutkować wyrenderowaną zawartością, która jest pusta. Implementacje MOGĄ zdecydować, aby nie wyświetlać wyrenderowanej treści, która jest pusta, lub MOGĄ wskazywać obserwatorowi, że treść nie może zostać pomyślnie wyrenderowana. Implementacje POWINNY zapewniać opcję menu "wyświetl źródło" i zapewniać możliwość dostępu do oryginalnej treści, jeśli renderowanie skutkuje pustą treścią i jest typem MIME, który jest określony jako bezpieczny do wyświetlania w formacie źródłowym.


### Zot

The same considerations apply to content which uses the 'zot' serialisation. In this case, the content source is the "body" element and is of type "mimetype". The observer-neutral HTML rendering will be provided in the "html" element.