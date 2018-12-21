---
title: "Kontrakty"
actions:
  - 'sprawdźOdpowiedź'
  - 'podpowiedź'
material:
  editor:
    language: sol
    startingCode: |
      pragma solidity //1. Wpisz tutaj wersję Solidity

      //2. Tutaj utwórz kontrakt
    answer: >
      pragma solidity ^0.4.25;

      contract ZombieFactory {
      }
---
Zacznijmy od całkowitych podstaw:

Kod Solidity zawarty jest w **kontraktach**. `Kontrakt` jest fundamentalnym blokiem budującym aplikacje oparte na Ethereum — wszystkie zmienne i funkcje należą do kontraktu i to będzie punkt wyjścia dla wszystkich Twoich projektów.

Pusty kontrakt `HelloWorld` wygląda następująco:

    contract HelloWorld {
    
    }
    

## Wersja Pragma

Kod źródłowy Solidity zaczyna się poprzez "wersję pragma" — deklarację wersji kompilatora Solidity, którego nasz kod będzie używał. Stosuje się to aby zapobiec problemom z przyszłymi wersjami, które moga potencjalnie zawierać zmiany kolidujące z Twoim kodem.

It looks like this: `pragma solidity ^0.4.25;` (for the latest solidity version at the time of this writing, 0.4.25).

Sklejając to w całość, poniżej mamy elementarną formę kontraktu — pierwszą rzecz, którą napiszesz, zaczynając nowy projekt:

    pragma solidity ^0.4.25;
    
    contract HelloWorld {
    
    }
    

# Wypróbujmy zatem

Aby zacząć tworzyć naszą armię Zombiaków, utwórzmy najpierw startowy kontrakt o nazwie `ZombieFactory`.

1. In the box to the right, make it so our contract uses solidity version `0.4.25`.

2. Utwórz pusty kontrakt o nazwie `ZombieFactory`.

Kiedy skończysz, kliknij "sprawdź odpowiedź" poniżej. Jeżeli utkniesz, możesz uzyskać podpowiedź klikając w przycisk "podpowiedź".