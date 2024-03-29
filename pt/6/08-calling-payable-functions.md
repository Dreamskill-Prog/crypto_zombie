---
title: Chamando funções pagáveis
actions: ['verificarResposta', 'dicas']
requireLogin: true
material:
  editor:
    language: html
    startingCode:
      "index.html": |
        <!DOCTYPE html>
        <html lang="en">
          <head>
            <meta charset="UTF-8">
            <title>CryptoZombies front-end</title>
            <script language="javascript" type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
            <script language="javascript" type="text/javascript" src="web3.min.js"></script>
            <script language="javascript" type="text/javascript" src="cryptozombies_abi.js"></script>
          </head>
          <body>
            <div id="txStatus"></div>
            <div id="zombies"></div>

            <script>
              var cryptoZombies;
              var userAccount;

              function startApp() {
                var cryptoZombiesAddress = "YOUR_CONTRACT_ADDRESS";
                cryptoZombies = new web3js.eth.Contract(cryptoZombiesABI, cryptoZombiesAddress);

                var accountInterval = setInterval(function() {
                  // Verifique se a conta foi alterada
                  if (web3.eth.accounts[0] !== userAccount) {
                    userAccount = web3.eth.accounts[0];
                    // Chamar a função para atualizar a interface do usuário com a nova conta
                    getZombiesByOwner(userAccount)
                    .then(displayZombies);
                  }
                }, 100);
              }

              function displayZombies(ids) {
                $("#zombies").empty();
                for (id of ids) {
                  // Busca os detalhes de zumbis do nosso contrato. Retorna um objeto `zumbi`
                  getZombieDetails (id)
                  .then(function(zumbi) {
                    // Usando os "template literals" do ES6 para injetar variáveis ​​no HTML.
                    // Anexa cada um ao nosso div #zombies
                    $("#zombies").append(`<div class="zombie">
                      <ul>
                        <li>Nome: ${zombie.name}</li>
                        <li>DNA: ${zombie.dna}</li>
                        <li>Nível: ${zombie.level}</li>
                        <li>Vitórias: ${zombie.winCount}</li>
                        <li>Perdas: ${zombie.lossCount}</li>
                        <li>Ready Time: ${zombie.readyTime}</li>
                      </ul>
                    </div>`);
                  });
                }
              }

              function createPseudoRandomZombie(name) {
                // Isso vai demorar um pouco, então atualize a interface do usuário para que o usuário saiba
                // a transação foi enviada
                $("#txStatus").text("Criando novo zumbi no blockchain. Isso pode demorar um pouco ...");
                // Envie o tx para nosso contrato:
                return cryptoZombies.methods.createPseudoRandomZombie(name)
                .send({from: userAccount})
                .on("receipt", function (receipt) {
                  $ ("#txStatus").text("Criado com sucesso" + name + "!");
                  // A transação foi aceita no blockchain, vamos redesenhar a interface do usuário
                  getZombiesByOwner(userAccount).then(displayZombies);
                })
                .on ("error", function (error) {
                  // Faça algo para alertar o usuário sobre a falha da transação
                  $("#txStatus").text(erro);
                });
              }

              function feedOnKitty(zombieId, kittyId) {
                $("#txStatus").text("Eating a kitty. This may take a while...");
                return cryptoZombies.methods.feedOnKitty(zombieId, kittyId)
                .send({ from: userAccount })
                .on("receipt", function(receipt) {
                  $("#txStatus").text("Ate a kitty and spawned a new Zombie!");
                  getZombiesByOwner(userAccount).then(displayZombies);
                })
                .on("error", function(error) {
                  $("#txStatus").text(error);
                });
              }

              // Comece aqui

              function getZombieDetails(id) {
                return cryptoZombies.methods.zombies(id).call()
              }

              function zombieToOwner(id) {
                return cryptoZombies.methods.zombieToOwner(id).call()
              }

              function getZombiesByOwner(owner) {
                return cryptoZombies.methods.getZombiesByOwner(owner).call()
              }

              window.addEventListener('load', function() {

                // Verificando se o Web3 foi injetado pelo navegador (Mist/MetaMask)
                if (typeof web3 !== 'undefined') {
                  // Use o provedor de Mist/MetaMask
                  web3js = new Web3(web3.currentProvider);
                } else {
                  // Caso o usuário não tem web3. Provavelmente
                  // mostre a ele uma mensagem dizendo-lhe para instalar o Metamask
                  // afim de usar nosso aplicativo.
                }

                // Agora você pode iniciar seu aplicativo e acessar o web3js livremente:
                startApp()

              })
            </script>
          </body>
        </html>
      "zombieownership.sol": |
        pragma solidity ^0.4.19;

        import "./zombieattack.sol";
        import "./erc721.sol";
        import "./safemath.sol";

        contract ZombieOwnership is ZombieAttack, ERC721 {

          using SafeMath for uint256;

          mapping (uint => address) zombieApprovals;

          function balanceOf(address _owner) public view returns (uint256 _balance) {
            return ownerZombieCount[_owner];
          }

          function ownerOf(uint256 _tokenId) public view returns (address _owner) {
            return zombieToOwner[_tokenId];
          }

          function _transfer(address _from, address _to, uint256 _tokenId) private {
            ownerZombieCount[_to] = ownerZombieCount[_to].add(1);
            ownerZombieCount[msg.sender] = ownerZombieCount[msg.sender].sub(1);
            zombieToOwner[_tokenId] = _to;
            Transfer(_from, _to, _tokenId);
          }

          function transfer(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
            _transfer(msg.sender, _to, _tokenId);
          }

          function approve(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
            zombieApprovals[_tokenId] = _to;
            Approval(msg.sender, _to, _tokenId);
          }

          function takeOwnership(uint256 _tokenId) public {
            require(zombieApprovals[_tokenId] == msg.sender);
            address owner = ownerOf(_tokenId);
            _transfer(owner, msg.sender, _tokenId);
          }
        }
      "zombieattack.sol": |
        pragma solidity ^0.4.19;

        import "./zombiehelper.sol";

        contract ZombieAttack is ZombieHelper {
          uint randNonce = 0;
          uint attackVictoryProbability = 70;

          function randMod(uint _modulus) internal returns(uint) {
            randNonce++;
            return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
          }

          function attack(uint _zombieId, uint _targetId) external onlyOwnerOf(_zombieId) {
            Zombie storage myZombie = zombies[_zombieId];
            Zombie storage enemyZombie = zombies[_targetId];
            uint rand = randMod(100);
            if (rand <= attackVictoryProbability) {
              myZombie.winCount++;
              myZombie.level++;
              enemyZombie.lossCount++;
              feedAndMultiply(_zombieId, enemyZombie.dna, "zombie");
            } else {
              myZombie.lossCount++;
              enemyZombie.winCount++;
              _triggerCooldown(myZombie);
            }
          }
        }
      "zombiehelper.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefeeding.sol";

        contract ZombieHelper is ZombieFeeding {

          uint levelUpFee = 0.001 ether;

          modifier aboveLevel(uint _level, uint _zombieId) {
            require(zombies[_zombieId].level >= _level);
            _;
          }

          function withdraw() external onlyOwner {
            owner.transfer(this.balance);
          }

          function setLevelUpFee(uint _fee) external onlyOwner {
            levelUpFee = _fee;
          }

          function levelUp(uint _zombieId) external payable {
            require(msg.value == levelUpFee);
            zombies[_zombieId].level++;
          }

          function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) onlyOwnerOf(_zombieId) {
            zombies[_zombieId].name = _newName;
          }

          function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) onlyOwnerOf(_zombieId) {
            zombies[_zombieId].dna = _newDna;
          }

          function getZombiesByOwner(address _owner) external view returns(uint[]) {
            uint[] memory result = new uint[](ownerZombieCount[_owner]);
            uint counter = 0;
            for (uint i = 0; i < zombies.length; i++) {
              if (zombieToOwner[i] == _owner) {
                result[counter] = i;
                counter++;
              }
            }
            return result;
          }

        }
      "zombiefeeding.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefactory.sol";

        contract KittyInterface {
          function getKitty(uint256 _id) external view returns (
            bool isGestating,
            bool isReady,
            uint256 cooldownIndex,
            uint256 nextActionAt,
            uint256 siringWithId,
            uint256 birthTime,
            uint256 matronId,
            uint256 sireId,
            uint256 generation,
            uint256 genes
          );
        }

        contract ZombieFeeding is ZombieFactory {

          KittyInterface kittyContract;

          modifier onlyOwnerOf(uint _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            _;
          }

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          function _triggerCooldown(Zombie storage _zombie) internal {
            _zombie.readyTime = uint32(now + cooldownTime);
          }

          function _isReady(Zombie storage _zombie) internal view returns (bool) {
              return (_zombie.readyTime <= now);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal onlyOwnerOf(_zombieId) {
            Zombie storage myZombie = zombies[_zombieId];
            require(_isReady(myZombie));
            _targetDna = _targetDna % dnaModulus;
            uint newDna = (myZombie.dna + _targetDna) / 2;
            if (keccak256(_species) == keccak256("kitty")) {
              newDna = newDna - newDna % 100 + 99;
            }
            _createZombie("NoName", newDna);
            _triggerCooldown(myZombie);
          }

          function feedOnKitty(uint _zombieId, uint _kittyId) public {
            uint kittyDna;
            (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
            feedAndMultiply(_zombieId, kittyDna, "kitty");
          }
        }
      "zombiefactory.sol": |
        pragma solidity ^0.4.19;

        import "./ownable.sol";
        import "./safemath.sol";

        contract ZombieFactory is Ownable {

          using SafeMath for uint256;

          event NewZombie(uint zombieId, string name, uint dna);

          uint dnaDigits = 16;
          uint dnaModulus = 10 ** dnaDigits;
          uint cooldownTime = 1 days;

          struct Zombie {
            string name;
            uint dna;
            uint32 level;
            uint32 readyTime;
            uint16 winCount;
            uint16 lossCount;
          }

          Zombie[] public zombies;

          mapping (uint => address) public zombieToOwner;
          mapping (address => uint) ownerZombieCount;

          function _createZombie(string _name, uint _dna) internal {
            uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime), 0, 0)) - 1;
            zombieToOwner[id] = msg.sender;
            ownerZombieCount[msg.sender]++;
            NewZombie(id, _name, _dna);
          }

          function _generatePseudoRandomDna(string _str) private view returns (uint) {
            uint rand = uint(keccak256(_str));
            return rand % dnaModulus;
          }

          function createPseudoRandomZombie(string _name) public {
            require(ownerZombieCount[msg.sender] == 0);
            uint randDna = _generatePseudoRandomDna(_name);
            randDna = randDna - randDna % 100;
            _createZombie(_name, randDna);
          }

        }
      "ownable.sol": |
        /**
         * @title Ownable
         * @dev The Ownable contract has an owner address, and provides basic authorization control
         * functions, this simplifies the implementation of "user permissions".
         */
        contract Ownable {
          address public owner;

          event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

          /**
           * @dev The Ownable constructor sets the original `owner` of the contract to the sender
           * account.
           */
          function Ownable() public {
            owner = msg.sender;
          }


          /**
           * @dev Throws if called by any account other than the owner.
           */
          modifier onlyOwner() {
            require(msg.sender == owner);
            _;
          }


          /**
           * @dev Allows the current owner to transfer control of the contract to a newOwner.
           * @param newOwner The address to transfer ownership to.
           */
          function transferOwnership(address newOwner) public onlyOwner {
            require(newOwner != address(0));
            OwnershipTransferred(owner, newOwner);
            owner = newOwner;
          }

        }
      "safemath.sol": |
        pragma solidity ^0.4.18;

        /**
         * @title SafeMath
         * @dev Math operations with safety checks that throw on error
         */
        library SafeMath {

          /**
          * @dev Multiplies two numbers, throws on overflow.
          */
          function mul(uint256 a, uint256 b) internal pure returns (uint256) {
            if (a == 0) {
              return 0;
            }
            uint256 c = a * b;
            assert(c / a == b);
            return c;
          }

          /**
          * @dev Integer division of two numbers, truncating the quotient.
          */
          function div(uint256 a, uint256 b) internal pure returns (uint256) {
            // assert(b > 0); // Solidity automatically throws when dividing by 0
            uint256 c = a / b;
            // assert(a == b * c + a % b); // There is no case in which this doesn't hold
            return c;
          }

          /**
          * @dev Subtracts two numbers, throws on overflow (i.e. if subtrahend is greater than minuend).
          */
          function sub(uint256 a, uint256 b) internal pure returns (uint256) {
            assert(b <= a);
            return a - b;
          }

          /**
          * @dev Adds two numbers, throws on overflow.
          */
          function add(uint256 a, uint256 b) internal pure returns (uint256) {
            uint256 c = a + b;
            assert(c >= a);
            return c;
          }
        }
      "erc721.sol": |
        contract ERC721 {
          event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
          event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

          function balanceOf(address _owner) public view returns (uint256 _balance);
          function ownerOf(uint256 _tokenId) public view returns (address _owner);
          function transfer(address _to, uint256 _tokenId) public;
          function approve(address _to, uint256 _tokenId) public;
          function takeOwnership(uint256 _tokenId) public;
        }
    answer: |
      <!DOCTYPE html>
      <html lang="en">
        <head>
          <meta charset="UTF-8">
          <title>CryptoZombies front-end</title>
          <script language="javascript" type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
          <script language="javascript" type="text/javascript" src="web3.min.js"></script>
          <script language="javascript" type="text/javascript" src="cryptozombies_abi.js"></script>
        </head>
        <body>
          <div id="txStatus"></div>
          <div id="zombies"></div>

          <script>
            var cryptoZombies;
            var userAccount;

            function startApp() {
              var cryptoZombiesAddress = "YOUR_CONTRACT_ADDRESS";
              cryptoZombies = new web3js.eth.Contract(cryptoZombiesABI, cryptoZombiesAddress);

              var accountInterval = setInterval(function() {
                // Verifique se a conta foi alterada
                if (web3.eth.accounts[0] !== userAccount) {
                  userAccount = web3.eth.accounts[0];
                  // Chamar a função para atualizar a interface do usuário com a nova conta
                  getZombiesByOwner(userAccount)
                  .then(displayZombies);
                }
              }, 100);
            }

            function displayZombies(ids) {
              $("#zombies").empty();
              for (id of ids) {
                // Busca os detalhes de zumbis do nosso contrato. Retorna um objeto `zumbi`
                getZombieDetails (id)
                .then(function(zumbi) {
                  // Usando os "template literals" do ES6 para injetar variáveis ​​no HTML.
                  // Anexa cada um ao nosso div #zombies
                  $("#zombies").append(`<div class="zombie">
                    <ul>
                      <li>Nome: ${zombie.name}</li>
                      <li>DNA: ${zombie.dna}</li>
                      <li>Nível: ${zombie.level}</li>
                      <li>Vitórias: ${zombie.winCount}</li>
                      <li>Perdas: ${zombie.lossCount}</li>
                      <li>Ready Time: ${zombie.readyTime}</li>
                    </ul>
                  </div>`);
                });
              }
            }

            function createPseudoRandomZombie(name) {
              // Isso vai demorar um pouco, então atualize a interface do usuário para que o usuário saiba
              // a transação foi enviada
              $("#txStatus").text("Criando novo zumbi no blockchain. Isso pode demorar um pouco ...");
              // Envie o tx para nosso contrato:
              return cryptoZombies.methods.createPseudoRandomZombie(name)
              .send({from: userAccount})
              .on("receipt", function (receipt) {
                $ ("#txStatus").text("Criado com sucesso" + name + "!");
                // A transação foi aceita no blockchain, vamos redesenhar a interface do usuário
                getZombiesByOwner(userAccount).then(displayZombies);
              })
              .on ("error", function (error) {
                // Faça algo para alertar o usuário sobre a falha da transação
                $("#txStatus").text(erro);
              });
            }

            function feedOnKitty(zombieId, kittyId) {
              // Isso vai demorar um pouco, então atualize a interface do usuário para que o usuário saiba
              // a transação foi enviada
              $("#txStatus").text("Comendo um gatinho. Isso pode demorar um pouco...");
              // Envie o tx para nosso contrato:
              return cryptoZombies.methods.feedOnKitty(zombieId, kittyId)
              .send({ from: userAccount })
              .on("receipt", function(receipt) {
                $("#txStatus").text("Comeu um gatinho e gerou um novo Zumbi!");
                // A transação foi aceita no blockchain, vamos redesenhar a interface do usuário
                getZombiesByOwner(userAccount).then(displayZombies);
              })
              .on("error", function(error) {
                // Faça algo para alertar o usuário sobre a falha da transação
                $("#txStatus").text(error);
              });
            }

            function levelUp(zombieId) {
              $("#txStatus").text(""Upando seu zumbi..."");
              return cryptoZombies.methods.levelUp(zombieId)
              .send({ from: userAccount, value: web3js.utils.toWei("0.001", "ether") })
              .on("receipt", function(receipt) {
                $("#txStatus").text("Poder esmagador! Zumbi upado com sucesso");
              })
              .on("error", function(error) {
                $("#txStatus").text(error);
              });
            }

            function getZombieDetails(id) {
              return cryptoZombies.methods.zombies(id).call()
            }

            function zombieToOwner(id) {
              return cryptoZombies.methods.zombieToOwner(id).call()
            }

            function getZombiesByOwner(owner) {
              return cryptoZombies.methods.getZombiesByOwner(owner).call()
            }

            window.addEventListener('load', function() {

              // Verificando se o Web3 foi injetado pelo navegador (Mist/MetaMask)
              if (typeof web3 !== 'undefined') {
                // Use o provedor de Mist/MetaMask
                web3js = new Web3(web3.currentProvider);
              } else {
                // Caso o usuário não tem web3. Provavelmente
                // mostre a ele uma mensagem dizendo-lhe para instalar o Metamask
                // afim de usar nosso aplicativo.
              }

              // Agora você pode iniciar seu aplicativo e acessar o web3js livremente:
              startApp()

            })
          </script>
        </body>
      </html>
---

A lógica para `attack`, `changeName` e `changeDna` será extremamente similar, então eles são triviais de implementar e não vamos perder tempo codificando-os nesta lição.

> Na verdade, já existe muita lógica repetitiva em cada uma dessas chamadas de função, portanto, provavelmente faria sentido refatorar e colocar o código comum em sua própria função. (E use um sistema de templates para as mensagens `txStatus` — já estamos vendo o quanto as coisas seriam mais limpas com um framework como o Vue.js!)

Vamos ver outro tipo de função que requer tratamento especial nas funções Web3.js — `payable`.

## Upar!

Lembre-se em `ZombieHelper`, nós adicionamos uma função pagável onde o usuário pode subir de nível:

```
function levelUp(uint _zombieId) external payable {
  require (msg.value == levelUpFee);
  zombies[_zombieId].level++;
}
```

A maneira de enviar Ether junto com uma função é simples, com uma ressalva: precisamos especificar quanto enviaremos em "wei", não em Ether.

## O que é um Wei?

Um `wei` é a menor subunidade de Ether — existem 10 ^ 18 `wei` em um `ether`.

É um monte de zeros para contar — mas felizmente o Web3.js tem um utilitário de conversão que faz isso por nós.

```
// Isto irá converter 1 ETH para Wei
web3js.utils.toWei("1", "ether");
```

Em nosso DApp, definimos `levelUpFee = 0.001 ether`, então quando chamamos nossa função `levelUp`, podemos fazer o usuário enviar o `0.001` Ether junto com ele usando o seguinte código:

```
cryptoZombies.methods.levelUp(zombieId)
.send({from: userAccount, value: web3js.utils.toWei("0.001", "ether")})
```

## Vamos testar

Vamos adicionar uma função `levelUp` abaixo `feedOnKitty`. O código será muito semelhante ao `feedOnKitty`, mas:

1. A função terá 1 parâmetro, `zombieId`

2. Pré-transação, ele deve exibir o texto `txStatus` `"Upando seu zumbi..."`

3. Quando ele chama `levelUp` no contrato, ele deve enviar `"0.001"` ETH convertido `toWei`, como no exemplo acima

4. Após o sucesso, ele deve exibir o texto `"Poder esmagador! Zumbi upado com sucesso"`

5. Nós **não** precisamos redesenhar a interface do usuário consultando nosso smart contract com `getZombiesByOwner` — porque neste caso sabemos que a única coisa que mudou foi o nível de um zumbi.
