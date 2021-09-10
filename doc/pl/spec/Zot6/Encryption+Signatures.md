### Szyfrowanie

Witryny udostępniają w swoim dokumencie wykrywania witryn tablicę zawierającą 0 lub więcej algorytmów szyfrowania, które są możliwe do zaakceptowania, w kolejności preferencji. Witryny wysyłające zaszyfrowane dokumenty MUSZĄ korzystać z tej listy, aby określić najbardziej odpowiedni algorytm dla obu stron. Jeśli nie można wynegocjować odpowiedniego algorytmu, strona MOŻE powrócić do zwykłego tekstu (dane niezaszyfrowane), ale jeśli kanał komunikacji nie jest zabezpieczony SSL, strona wysyłająca NIE MOŻE używać zwykłego tekstu, a strona odbierająca MOŻE zignorować lub odrzucić komunikację, jeśli zawiera ona informacje prywatne lub informacje poufne.

Jeśli strona odbierająca nie obsługuje podanego algorytmu, MUSI zwrócić błąd 400.

Zaszyfrowane informacje są hermetyzowane w tablicy lub obiekcie JSON z następującymi komponentami:

```
'encrypted' => true
'key'       => Klucz szyfrowania, base64urlencoded
'iv'        => Wektor inicjujący szyfrowanie, base64urlencoded
'alg'       => Zastosowany algorytm szyfrowania
'data'      => Zaszyfrowany ładunek, base64urlencoded
```

Flaga logiczna 'encrypted' wskazuje, że jest to struktura kryptograficzna i wymaga odszyfrowania w celu wyodrębnienia informacji. Element 'alg' jest obowiązkowy. Inne elementy mogą zostać zmienione w razie potrzeby w celu obsługi różnych mechanizmów lub algorytmów. Na przykład niektóre mechanizmy mogą wymagać pola 'hmac'. Przedstawione elementy obsługują szeroką gamę standardowych algorytmów szyfrowania.

Element 'key' i 'iv' są pseudolosowymi sekwencjami bajtów, zaszyfrowanymi kluczem publicznym RSA odbiorcy przed kodowaniem base64urlen. Kluczem odbiorcy używanym w większości przypadków (domyślnie) będzie klucz publiczny witryny zdalnej. W pewnych okolicznościach (jeśli określono) kluczem publicznym RSA będzie klucz kanału docelowego lub odbiorcy.

Zarówno 'key', jaki i 'iv' MOGĄ być uzupełnione do 255 znaków. Niezbędna ilość dopełnienia zależy od algorytmu szyfrowania. Przychodząca strona MUSI usunąć dodatkowe wypełnienie dla obu parametrów, aby ograniczyć maksymalną długość obsługiwaną przez wybrany algorytm. Na przykład algorytm aes256cbc (niezalecany) używa klucza o długości 32 bajtów i iv 16 bajtów.

Nazwy algorytmów to nazwy algorytmów pisane małymi literami używane przez bibliotekę openssl z usuniętą interpunkcją. Na przykład algorytm openssl 'aes-256-ctr' jest oznaczony jako 'aes256ctr'.

Mogą być używane rzadkie algorytmy, które nie są obsługiwane przez openssl, ale w tym dokumencie nie definiuje sie dokładnych nazw tych algorytmów.


### Podpisy

Pochodzenie tożsamości jest dostarczane za pomocą podpisów HTTP (obecnie odpowiednia specyfikacja to draft-cavage-http-signatures-10). W przypadku korzystania z transportu szyfrowanego podpis HTTP MOŻE być zaszyfrowany przy użyciu tego samego wynegocjowanego algorytmu, który jest używany w wiadomości do ochrony koperty. Zobacz [Podpisy HTTP](spec/HTTPSignatures/Home). Jeśli witryna używa negocjowanego szyfrowania, jak opisano w poprzedniej sekcji, MUSI być w stanie odszyfrować sygnatury HTTP.

W kilku miejscach komunikacji, w których weryfikacja jest związana z osobą trzecią, która nie jest nadawcą odpowiedniego pakietu HTTP, podpisane dane lub obiekty są określone/wymagane. Można użyć dwóch metod podpisu, w zależności od tego, czy podpisane dane są pojedynczą wartością, czy obiektem JSON. Metoda używana dla pojedynczych wartości jest nazywana tutaj SimpleSignatures. Wykorzystywana metoda podpisywania obiektów jest tradycyjnie nazywana "magicznymi podpisami protokołu Salmon", wykorzystującą serializację JSON.


#### Prosty podpisy

Wartość `data` jest podpisywana odpowiednią metodą RSA i algorytmem skrótu, na przykład `sha256`, który wskazuje sygnaturę pary kluczy RSA przy użyciu wyznaczonego klucza prywatnego RSA i algorytmu mieszającego sha256. Wynik jest zakodowany w base64url, poprzedzony nazwą algorytmu i kropką (0x2e) i wysłany jako dodatkowy element danych, jeśli określono.


````
"foo_sig": "sha256.EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe..."
````

Aby zweryfikować, zawartość elementu jest ona rozdzielana w pierwszym okresie w celu wyodrębnienia algorytmu i podpisu. Podpis jest dekodowany w base64url i weryfikowany przy użyciu odpowiedniej metody i algorytmu skrótu przy użyciu wyznaczonego klucza publicznego RSA (w przypadku podpisów RSA). Właściwy klucz do użycia jest zdefiniowany w innym miejscu protokołu Zot w zależności od kontekstu podpisu.

Implementacje MUSZĄ obsługiwać podpisy RSA-SHA256. MOGĄ obsługiwać dodatkowe metody podpisu. 

#### Podpisy "magicznej koperty" Salmon z serializacją JSON

````
{
  "signed": true,
  "data": "PD94bWwgdmVyc2lvbj0nMS4wJyBlbmNvZGl...",
  "data_type": "application/x-zot+json",
  "encoding": "base64url",
  "alg": "RSA-SHA256",
  "sigs": [
    {
    "value": "EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe...",
    "key_id": "4k8ikoyC2Xh+8BiIeQ+ob7Hcd2J7/Vj3uM61dy9iRMI"
    }
  ]
}
````

Logiczny element `signed` nie jest zdefiniowany w specyfikacji magicznej koperty. Jest to flaga logiczna, która wskazuje, że bieżący element jest podpisanym obiektem i wymaga weryfikacji i rozpakowania w celu pobrania rzeczywistej zawartości elementu.

Podpisane dane są pobierane przez rozpakowanie komponentu magicznego podpisu `data`. Rozpakowanie odbywa się poprzez usunięcie wszystkich białych znaków (0x0d 0x0a 0x20 i 0x09) i zastosowanie dekodowania base64 ścieżki URL.

`Key_id` to identyfikator osoby podpisującej zakodowany w formacie base64urlen, który po zastosowaniu w procesie „Discovery” Zot spowoduje zlokalizowanie klucza publicznego. Jest to zazwyczaj adres URL bazy serwera lub adres URL „domu” kanału. Identyfikatory Webfinger (acct:user@domain) MOGĄ być również używane, jeśli wynikowy dokument Webfinger zawiera wykrywalny klucz publiczny (salmon-public-key lub Webid key).

The key_id is the base64urlencoded identifier of the signer, which when applied to the Zot 'Discovery' process will result in locating the public key. This is typically the server base url or the channel "home" url. Webfinger identifiers (acct:user@domain) MAY also be used if the resultant webfinger document contains a discoverable public key (salmon-public-key or Webid key). 

Verification is performed by using the magic envelope verification method. First remove all whitespace characters from the 'data' value. Append the following fields together separated with a period (0x2e). 

data . data_type . encoding . algorithm

This is the "signed data". Verification will take place using the RSA verify function using the signed data, the base64urldecoded sigs.value,  algorithm specified in 'alg' and the public key found by performing Zot discovery on the base64urldecoded sigs.key_id. 

When performing Zot discovery for keys, it is important to verify that the principal returned in the discovery response matches the principal in the key_id and that the discovery response is likewise signed and validated.  

Some historical variants of magic signatures emit base64 url encoding with or without padding.

In this specification, encoding MUST be the string "base64url", indicating the url safe base64 encoding as described in RFC4648, and without any trailing padding using equals (=) characters

The unpacked data (once verified) may contain a single value or a compound object. If it contains a single value, this becomes the value of the enclosing element. Otherwise it is merged back as a compound object. 

Example: source document

```
{ 
    "guid": {
      "signed": true,
      "data": "PD94bWwgdmVyc2lvbj0nMS4wJyBlbmNvZGl...",
      "data_type": "application/x-zot+json",
      "encoding": "base64url",
      "alg": "RSA-SHA256",
      "sigs": [
        {
        "value": "EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe...",
        "key_id": "4k8ikoyC2Xh+8BiIeQ+ob7Hcd2J7/Vj3uM61dy9iRMI"
        }
      ]
    },
    "address": "foo@bar"
}
```

Decoding the data parameter (and assuming successful signature verification) results in

````
"abc12345"
````

Merging with the original document produces

````
{
    "guid": "abc12345",
    "address": "foo@bar"
}
````

Example using a signed object containing multiple elements:


Decoding the data parameter (and assuming successful signature verification) results in

````
{
    "guid": "abc12345",
    "name": "Barbara Jenkins"
}
````

Merging this with the original document produces

````
{
    "guid": {
        "guid": "abc12345",
        "name": "Barbara Jenkins"
    },
    "address": "foo@bar"
}
````