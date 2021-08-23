### OpenWebAuth

OpenWebAuth zapewnia lekką formę uwierzytelniania międzydomenowego między witrynami w otwartej sieci. Podmioty główne zaangażowane w uwierzytelnianie mogą, ale nie muszą mieć żadnego wcześniej istniejącego związku.

OpenWebAuth wykorzystuje webfinger (RFC7033) i podpisy HTTP (draft-cavage-http-signatures-09) z prostą usługą generowania tokenów, aby zapewnić bezproblemowe i wolne od interakcji uwierzytelnianie między różnymi witrynami.

Na przykład w witrynie podunk.edu członek udostępnił wideo prywatnie pod adresem bob@example.com. Aby bob@example.com mógł zweryfikować swoją tożsamość i obejrzeć chroniony film, musi potwierdzić swoją tożsamość w podunk.edu.

Na wysokim poziomie, aby aktor mógł odwiedzić inną witrynę jako uwierzytelniony widz, najpierw jest przekierowywany do usługi, która może tworzyć podpisy cyfrowe w jego imieniu i która otrzymuje docelowy adres URL. Ta usługa musi mieć dostęp do ich prywatnego klucza podpisywania.

Klucz publiczny jest przechowywany w usłudze kompatybilnej z protokłem WebFinger lub udostępniany bezpośrednio jako właściwość tego protokołu. Zasób WebFinger jest dostarczany jako keyID nagłówka "Authorization" podpisu HTTP.

Jest bardzo mały konsensus na dostarczanie kluczy publicznych w WebFinger, z wyjątkiem magicznego klucza publicznego. W przypadkach, w których preferuje się format klucza wyższego poziomu, MOŻNA użyć nazwę właściwości "https://w3id.org/security/v1#publicKeyPem", która choć nieoficjalna, jest zgodna z JSON-LD i ActivityPub. Serwery MOGĄ korzystać z innych zasobów WebFinger, jeśli w dokumencie WebFinger JRD nie ma nadającego się do użytku klucza publicznego. Te mechanizmy wykrywania są poza zakresem tego dokumentu.

Żądanie WebFinger z *baseurl* docelowego adresu URL zwraca wpis dla punktu końcowego usługi OpenWebAuth. Na przykład:

````
rel: https://purl.org/openwebauth/v1
type: application/json
href: https://example.com/openwebauth
````

Redirector podpisuje żądanie HTTPS GET do punktu końcowego OpenWebAuth za pomocą podpisów HTTP, które zwracają dokument json:

````
{
  'success': true,
  'encrypted_token': 'bgU50kUhtlMV5gKo1ce'
}
````

"Token" to jednorazowy token dostępu - ogólnie, unikalna haszowana wartość o długości od 16 do 56 znaków (jest to zgodne z szyfrowaniem RSA OAEP przy użyciu 1024-bitowego klucza RSA). Wynikowy token najprawdopodobniej zostanie użyty w adresie URL, więc znaki MUSZĄ należeć do zakresu [a-zA-Z0-9]. Aby wygenerować `encrypted_token`, ten token jest najpierw szyfrowany kluczem publicznym aktora przy użyciu szyfrowania RSA, a wynik jest zaszyfrowany jako base64_url.

Jeśli podpis nie może zostać zweryfikowany, usługa OpenWebAuth zwraca


````
{
    'success': false,
    'message': 'Reason'
}
````

'message' nie jest wymagany. 

Jeśli zwrócony zostanie zaszyfrowany token, będzie on odszyfrowywany:

- odszyfrowanie 'encrypted_token' przy użyciu szyfru base64_url;
- odszyfrowanie tego wyniku przy użyciu klucza prywatnego RSA należącego do oryginalnego aktora, czego wynikiem jest 'token' w zwykłym tekście.

Token dodawanay jest do docelowego adresu URL jako parametr zapytania 'owt' (OpenWebToken).

```
303 https://example.com/some/url?owt=abc123
```

Sesja przeglądarki użytkownika jest teraz przekierowywana do docelowego adresu URL (z dostarczonym tokenem) i jest uwierzytelniana, co umożliwia dostęp do różnych zasobów sieciowych w zależności od uprawnień przyznanych jego tożsamości webfinger.


> ##### Uwaga: 
> Wszystkie interakcje MUSZĄ odbywać się przez transport `https` z ważnymi certyfikatami. Sygnatury HTTP nie zapewniają żadnej ochrony przed atakami typu "replay". Usługa tokenów OpenWebAuth MUSI odrzucić tokeny po pierwszym użyciu i POWINNA odrzucić nieużywane tokeny w ciągu kilku minut od wygenerowania.
