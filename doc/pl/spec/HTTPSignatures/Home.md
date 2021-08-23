### Szyfrowane podpisy HTTP

Metoda dostarczania podpisów klucza publicznego i uwierzytelniania dla żądań HTTP jest opisana w dokumencie [Signing HTTP Messages draft-cavage-http-signatures-09](https://datatracker.ietf.org/doc/html/draft-cavage-http-signatures-09).

Podstawowym ograniczeniem (wadą) podpisów HTTP podatność na wyciek metadanych nadawcy komunikacji poprzez "keyId".

Zaszyfrowane podpisy HTTP korygują to, szyfrując nagłówek podpisu.

Szyfrowanie wykorzystuje klucz publiczny witryny odbierającej. Zawartość nagłówka `Signature` lub `Authorization` po utworzeniu podpisu HTTP jest przekazywana przez funkcję szyfrującą f(header,key,algorithm) z kluczem publicznym strony zdalnej oraz wspólnie uzgodnionym algorytmem szyfrującym, który zwraca zaszyfrowaną strukturę zawierającą:

- **key**: "losowy" łańcuch zaszyfrowany kluczem publicznym RSA zdalnej witryny i zakodowany w base64url
- **iv**: dosdatkowy "losowy" łańcuch zaszyfrowany kluczem publicznym RSA zdalnej witryny i zakodowany w base64url (opcjonalnie)
- **alg**: użyty algorytm szyfrowania
- **data**: zaszyfrowane dane, zakodowane w base64url
- **hmac**: kod HMAC zakodowany w base64url (opcjonalnie) 

Nagłówek jest generowany przez zastosowanie każdej nazwy pola, po której następuje '=', a następnie wartość pola w cudzysłowiu i pola oddzielone przecinkami (tak samo jak opisano to w draft-cavage-http-signatures)

Szyfrowanie odbywa się poprzez zaszyfrowanie ciągu nagłówka wybranym algorytmem z wykorzystanie łańcuchów `key` i `iv`. Łańcuchy `key` i `iv` **mogą** mieć większą długość niż pozwala na to określony algorytm. Łańcuchy te są skracane do żądanej długości klucza i długości wektora inicjalizacji przed szyfrowaniem, ale przesyłane są w całości. Zazwyczaj losowy ciąg ma długość 256 oktetów, a 'key' i `iv` są na ogół ograniczone do 16 lub 32 oktetów (w zależności od zastosowanego algorytmu szyfrowania). Nie ma ograniczeń dotyczących znaków "losowego" ciągu oktetu.

Wynikowy nagłówek:

````
Signature: iv="d-uqkRoeXCoL1T5DU74ywizSM2RgsI9ZXWREKVg3_Qjd-mUWTJVGLq2hOQi3XKaa9Q7R6uB6UzlRmLfxBMZhVIxHjdNgfSRQ_oXafiSv8bZzMVKLZCjw6PfxBcljFs5gaQ7vEGuOVZ5nUaNEU7QX7WFr7BQKlev_6GFruv7HOsehGCokpyHHkKwrQ_4WJxUZp7o1ZhS1masPqMrEtUxDGfKwHfiHILuMdWDBvv2Xk4iHzlCi9fRVUEvzzFvv1rXsanjbaypZMIfSNj31kvsGfs6IyHpIaKbFqRs_iCxfujKDYh-2Dsg02bJTF1qx9BHJqLKNpfc0iReVe_xV2Qom3-SrJe1K8mRzYQJuOyyDuQk04GBlw7ken698JcwuS0G0OMfvGh5okq_0wM_O09iYumnJlEZT2a5nJ8ifc-kZfu8zdIPyAjJvS3a3KGEsytLxuUekFPVIpEoV2rmgOWz0TzDg-mIgwFffcx3kDa_WWhPGCFwVOzGU9um0KKStThKNXrbjYEAHVxD0gYXPgwmL8KayCo2A2s4bE2W8FfSURGu4Noqr9VsZ69Bcygzitv3aWCeIAk0y7kjJ0yQDfuIOjK1GP4HECq5NJIf8L3LJKw8QIBKm_0nx4gV9rLSAKCe3S63-D1tp9hafeiKQvGSwR0ybhxTJrhkcxd2nieVAyoA",key="Ca14lvjZua-ED8kXbedNLmrk6mRMHZm9NugcphyBMKEBo8MXLLnTsZchkAP-auWa0iJFKRwtdYUW_IGO-WX_qKZ8VNOslViveTYY-ybLTjQUj--YCFuURLYUWYTEmDcOImPWc8cQYGjTL_PN5X7vo7t3cm6rdV2W4tio2Rrmg3-cjhXBBRElr3GQKQ7i9ljBPs2YffoRsJ7f8DycKeyTv1T9xwr5lDklWOcOMTD4_39cZN2BI-b3AcGhBG4oYabUavW3BLGX7-SnezUcbTP3RyCVGI0ylVS8FmHSBZmW0oWfrVmz0oc0UcZYQMk8rb2WL_2ZdnzV_yZsjbBTFHG1ytIYyMeJsUU-pv4b4TodZmuDKT5UGtXPhm8Lsh-JpFo8xj5Yl15T9H6yLVHMR7Wzx_r2SvlJUsyqzBpaZE8DMd0zzrNZwgHQZ08wVHieKKO-TIqdypZHkxGGM68u2NPPW8-mXHgd_w9fUNM5fZRKPL9GxoVqoe9hx2f6CXPD95GAwjer9hbJcOmvxA1veXpIQzlkd-kEc8EuECaC50aUZJZbUIghYFo9NAA-UgNb26TyuY1OwE55MstPA6OO1sFki2u1G1T5JGWWgIOAziCcZbDYl1NPFWD2I1sV__rYeZ6XaaW4GXIVqD3wyBpmBRIoFx43gVDTISyUjhjUjjVHbZE",alg="aes256ctr",data="CLBNNE-tR1lRm0QL5gS86HyfwMs_16xKSSHTBP7MUEmRhGR00s0cdOfLC-PCZKlpG3ZRvc_lxnd53GGycNiTskisAb1mTbTrUBvk7hpDGNciUEB_7-hehjRiztmfi_oR-H0sCsVK9qDJdYepr4BYIgznVcB0uEN-POm97H4cTTVD8xCxLeEX0ArgDzgv_-Bq-nMcyht2LdGFl4Ej3bhEOhzvd-Xs1m6Z3E55dw0Bx7QDtkorvoetgMJrhgPKjYkIUWGoyVqa8MssvYIT8w9mpPDm4_QuVSNiPLIrKwQ3vob_hxcvENY-l0vXihdnpMzg81Sdk0E4FS4uQ9HtYSWsjOaFDSWRlxc-C5RhIvnHST4uEy3tjI--OHYQo2mFG2fWM3h8bYPq6r41W79qxsfmdSydmV1G5rFIqaz7gOa2JGOtW19WPJ8FTNFLVDehrFD6FJUy185gYyXosonp2EF3qlC8k_fzmazrzUrx0YmQ941870LJAwtEC7P-XiHV3dj-tZRYPgiSp7m8cMm7Z8WGgN8lLb61t5di5XS8zAv3FU1EAvvyL7PQhDi1U-s2cQXk3hXTNhOIymUYRhSV8NZrk80EsOrbPevSNQyYKXWCeUbnyhUznZQ3Lwq-UWAufcwrVY5uIJKeNu2lZ42xzSHWW3hn0ymcXzBOz7_wip9pSPY1nsTwApqTaIjURMEHhPvgaKRzNmuKbWP-d5Ihjeqw6JGXoAw0beWPJ4rqOlpQtn63deyBR5ylcRe4Ok2n03fZBnzJAobfZuHkiW93Yvc_byF-rpMJ3C8BSFYhGNDzYeRea3d9BEsqz_sr2HNpJyLhPssiZlZdjGRfqQ5UvCIJgT_NY57FoRCx4RHRpSxkjyF5XaKXW0_uNK7Oxk30qOCbIsLkQJqB2JIVrFFDBPITZIQVq2OamcBVk09OPuIMvsNBUTt2sxcZ7LVAA61ubv0jU39TcYO_OCs2eL7WaH7zDs9wHmxlwvzrPclduY5Gx2pwkrI_nb42j4Nc5imUkvzkIAhbYOB-XBClNVjFdEqYH35lziqEl9I6_w"
````

Odszyfrowanie nagłówka odwraca ten proces. 

- dekodowane są łańcuchy base64url `key` i `iv` oraz  pole `data` jak i pole `hmac`, jeśli występuje; 
- użyty zostaje klucz prywatney witryny do odszyfrowania `key` i `iv`;
- zastosowany zostaje algorytm deszyfrujący 'alg' do 'data' z wykorzystaniem 'key' i 'iv', obcinając 'key' i 'iv', jeśli jest to konieczne dla wybranego algorytmu;
- wynikiem końcowym jest podpis HTTP określony w draft-cavage-http-signatures, a więc procedura jest zgodna z tym dokumentem.
 
Opis wykrywania kluczy publicznych witryny i negocjowanie algorytmu wykracza poza zakres tego dokumentu.
