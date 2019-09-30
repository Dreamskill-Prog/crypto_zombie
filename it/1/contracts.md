---
title: "Contratti"
actions:
  - 'controllaRisposta'
  - 'suggerimenti'
material:
  editor:
    language: sol
    startingCode: |
      pragma solidity //1. Inserisci la versione solidity qui

      //2. Crea il contratto qui
    answer: >
      pragma solidity ^0.4.25;

      contract ZombieFactory {
      }
---

Iniziamo dalle basi assolute:

Il codice Solidity è contenuto nei **contratti**. Un `contratto` è un fondamentale elemento costitutivo delle applicazioni Ethereum — tutte le variabili e le funzioni appartengono al contratto, e sarà il punto di partenza di tutti i vostri progetti.

Un contratto vuoto nominato `HelloWorld` potrebbe essere simile a questo:

    contract HelloWorld {
    
    }
    

## Version Pragma

Tutto il codice sorgente solidity dovrebbe iniziare con "version pragma" — una dichiarazione della versione del compilatore di Solidity che dovrebbe utilizzare questo codice. Questo serve a evitare problemi con le versioni future del compilatore che potrebbero introdurre modifiche che potrebbero infrangere il tuo codice.

It looks like this: `pragma solidity ^0.4.25;` (for the latest solidity version at the time of this writing, 0.4.25).

Mettendola insieme, ecco un contratto iniziale da zero — la prima cosa che scriverai ogni volta che inizi un nuovo progetto:

    pragma solidity ^0.4.25;
    
    contract HelloWorld {
    
    }
    

# Facciamo un test

Per iniziare a creare il nostro esercito di Zombie, creiamo un contratto base chiamato `ZombieFactory`.

1. In the box to the right, make it so our contract uses solidity version `0.4.25`.

2. Crea un contratto vuoto chiamato `ZombieFactory`.

Al termine, clicca su "controlla risposta" sotto. Se rimani bloccato, puoi fare clic su "suggerimento".