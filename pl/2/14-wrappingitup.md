---
title: Zakończmy zabawę
actions:
  - 'sprawdźOdpowiedź'
  - 'podpowiedź'
material:
  saveZombie: prawda
  zombieBattle:
    zombie:
      lesson: 1
    humanBattle: fałsz
    hideSliders: prawda
    answer: 1
---
Tak jest, udało Ci się ukończyć lekcje 2!

You can check out the demo to the right to see it in action. Go ahead, I know you can't wait until the bottom of this page 

## Implementacja Javascript

Gdy jesteśmy gotowi, aby umieścić kontrakt w Ethereum będziemy musieli tylko skompilować i wdrożyć `ZombieFeeding` — ten kontrakt jest naszym ostatecznym kontraktem, który dziedziczy po `ZombieFactory` i ma dostęp do wszystkich publicznych metod w obu kontraktach.

Spójrzmy na przykład interakcji z naszymi wdrożonymi kontraktami za pomocą Javascript i web3.js:

    var abi = /* abi wygenerowane przez kompilator */
    var ZombieFeedingContract = web3.eth.contract(abi)
    var contractAddress = /* adres naszego kontraktu w Ethereum po wdrożeniu */
    var ZombieFeeding = ZombieFeedingContract.at(contractAddress)
    
    // Załóżmy, że posiadamy ID Zombiaka oraz ID Kotka, którego chcemy zaatakować
    let zombieId = 1;
    let kittyId = 1;
    
    // Aby dostać się do obrazka KryptoKiciusia, musimy zapytać o jego web API. Ta
    // informacja nie jest przechowywana w blockchain'ie, tylko na ich serwerze.
    // jeśli wszystko zostało przechowane w blockchain'ie, nie powinniśmy się martwić
    // o to, że serwer padnie, zmianę API, czy firma
    // zablokuje nam możliwość załadowania ich assetów jeśli nie polubią naszej gry Zombi ;)
    let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
    $.get(apiUrl, function(data) {
      let imgUrl = data.image_url
      // zrób coś, aby wyświetlić obrazek
    })
    
    // Kiedy użytkownik kliknie na Kotka:
    $(".kittyImage").click(function(e) {
      // Wywołaj z kontraktu metodę `feedOnKitty` 
      ZombieFeeding.feedOnKitty(zombieId, kittyId)
    })
    
    // Nasłuchuj nowego eventu NewZombie z naszego kontraktu, więc możemy to wyświetlić:
    ZombieFactory.NewZombie(function(error, result) {
      if (error) return
      // Ta funkcja wyświetli Zombi, jak w lekcji 1:
      generateZombie(result.zombieId, result.name, result.dna)
    })
    

# Give it a try!

Select the kitty you want to feed on. Your zombie's DNA and the kitty's DNA will combine, and you'll get a new zombie in your army!

Notice those cute cat legs on your new zombie? That's our final `99` digits of DNA at work 

You can start over and try again if you want. When you get a kitty zombie you're happy with (you only get to keep one), go ahead and proceed to the next chapter to complete lesson 2!