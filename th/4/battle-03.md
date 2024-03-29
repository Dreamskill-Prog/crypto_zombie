---
title: Zombie Fightin'
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombieattack.sol": |
        import "./zombiehelper.sol";

        contract ZombieBattle is ZombieHelper {
          uint randNonce = 0;
          // สร้าง attackVictoryProbability ตรงนี้เลย

          function randMod(uint _modulus) internal returns(uint) {
            randNonce++;
            return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
          }

          // สร้างฟังก์ชั่นใหม่ตรงนี้
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

          function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
            zombies[_zombieId].name = _newName;
          }

          function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
            require(msg.sender == zombieToOwner[_zombieId]);
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

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          function _triggerCooldown(Zombie storage _zombie) internal {
            _zombie.readyTime = uint32(now + cooldownTime);
          }

          function _isReady(Zombie storage _zombie) internal view returns (bool) {
              return (_zombie.readyTime <= now);
          }

          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
            require(msg.sender == zombieToOwner[_zombieId]);
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

        contract ZombieFactory is Ownable {

            event NewZombie(uint zombieId, string name, uint dna);

            uint dnaDigits = 16;
            uint dnaModulus = 10 ** dnaDigits;
            uint cooldownTime = 1 days;

            struct Zombie {
              string name;
              uint dna;
              uint32 level;
              uint32 readyTime;
            }

            Zombie[] public zombies;

            mapping (uint => address) public zombieToOwner;
            mapping (address => uint) ownerZombieCount;

            function _createZombie(string _name, uint _dna) internal {
                uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime))) - 1;
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
    answer: >
      import "./zombiehelper.sol";

      contract ZombieBattle is ZombieHelper {
        uint randNonce = 0;
        uint attackVictoryProbability = 70;

        function randMod(uint _modulus) internal returns(uint) {
          randNonce++;
          return uint(keccak256(now, msg.sender, randNonce)) % _modulus;
        }

        function attack(uint _zombieId, uint _targetId) external {
        }
      }
---

ณ ตอนนี้เราก็มีแหล่งในการสุ่มภายใน contract แล้ว ซึ่งจะนำมาใช้ในการคำนวณผลลัพธ์จากการต่อสู้ของซอมบี้นั่นเอง

การต่อสู้ของซอมบี้จะทำงานดังนี้:

- ให้เลือกซอมบี้ออกมา 1 ตัว และเลือกซอมบี้ฝั่งตรงข้ามออกมาให้เป็นฝ่ายตรงข้ามที่จะโจมตี
- หากคุณเลือกที่จะเป็นซอมบี้ตัวที่เข้าไปโจมตี จะมีโอกาส 70% ในการชนะ ส่วนซอมบี้ตัวที่ถูกโจมตีจะได้รับโอกาส 30% ที่จะชนะ
- ซอมบี้ทั้งหมด (ทั้งตัวที่เข้าโจมตีและถูกโจมตี) จะมีตัวแปร `winCount` และ `lossCount` ที่ผลจากการต่อสู้จะส่งผลต่อการเพิ่ม (increment) จำนวน
- ถ้าซอมบี้ที่เข้าโจมตีชนะ มันก็จะได้รับเลเวลเพิ่มขึ้นและสร้างซอมบี้ตัวใหม่ขึ้นมา
- ถ้าหากว่ามันแพ้ จะไม่มีอะไรเกิดขึ้น (ยกเว้นตัวแปร `lossCount` incrementing)
- ไม่ว่าจะแพ้หรือชนะ ฟังก์ชั่น cooldown ของซอมบี้ตัวที่เข้าโจมตีจะเริ่มทำงานเหมือนกัน

เรียกได้ว่าต้องใช้ logic หลายอย่างในการเขียนโค้ดขึ้นมา ดังนั้นเราจะค่อย ๆ สร้างมันขึ้นมาในบทเรียนที่กำลังจะมาถึง

## มาทดสอบกันได้แล้ว

1. เพิ่ม variable ชนิด `uint` ให้กับ contract ของเราโดยใช้ชื่อว่า `attackVictoryProbability`และตั้งให้มีค่าเท่ากับ `70`

2. สร้างฟังก์ชั่นที่มีชื่อว่า `attack`ซึ่งจะรับ parameter 2 ค่า : `_zombieId` (ชนิด `uint`) และ `_targetId` (`uint`) ซึ่งเป็นฟังก์ชั่น `external` 

ปล่อยส่วน body ของฟังก์ชั่นเอาไว้ก่อนในตอนนี้
