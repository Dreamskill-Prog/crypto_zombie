---
title: Refactoring Common Logic
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
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

          // 1. สร้าง modifier ตรงนี้

          function setKittyContractAddress(address _address) external onlyOwner {
            kittyContract = KittyInterface(_address);
          }

          function _triggerCooldown(Zombie storage _zombie) internal {
            _zombie.readyTime = uint32(now + cooldownTime);
          }

          function _isReady(Zombie storage _zombie) internal view returns (bool) {
              return (_zombie.readyTime <= now);
          }

          // 2. เพิ่ม modifier เข้าไปยังส่วน definition ของฟังก์ชั่น:
          function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
            // 3. ลบบรรทัดนี้ออก
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
      "zombieattack.sol": |
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

        modifier ownerOf(uint _zombieId) {
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

        function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal ownerOf(_zombieId) {
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
---

ใครก็ตามที่เรียกฟังก์ชั่น`attack` — เราต้องการแน่ใจว่าผู้ใช้คนนั้นเป็นเจ้าของซอมบี้ตัวที่เข้าโจมตีด้วย เพราะไม่ยังงั้นเราคงต้องมาโฟกัสเรื่องความปลอดภัยแน่นอนหากผู้ใช้สามารถโจมตีซอมบี้ของผู้ใช้อื่นได้

งั้นลองมาดูกันว่าเราจะสามารถเพิ่มการ check ว่าผู้ที่เรียกฟังก์ชั่นนี้นั้นเป็นเจ้าของ `_zombieId` ที่ตัวเองได้เพิ่มเข้าไปหรือไม่?

ให้ใช้เวลาคิดดูสักครู่ ว่าเราจะสามารถคิดคำตอบได้ด้วยตัวเองหรือเปล่านะ

แป๊บนึงสิ... ลองกลับไปดูโค้ดในบทก่อนหน้าก็ได้ เผื่อจะได้ไอเดียอะไรขึ้นมา ...

เอาล่ะ คำตอบอยู่ข้างล่างนี้แล้ว แต่ขออย่าเพิ่งลงไปดูจนกว่าคุณจะได้ลองคิดทบทวนดูก่อนนะ

## คำตอบมาแล้วจ้า

เราได้ทำการสร้างตัว check นี้ขึ้นมาหลายครั้งแล้วล่ะ ใน `changeName()`, `changeDna()`, และ `feedAndMultiply()` เราได้ใช้ check ดังนี้:

```
require(msg.sender == zombieToOwner[_zombieId]);
```

ซึ่งนี้ก็จะใช้ตรรกะเดียวกันที่เราต้องใช้ในฟังก์ชั่น `attack` เนื่องจากเราต้องใช้ตรรกะนี้หลายครั้ง จึงขอให้กลับไปดูในส่วนของ `modifier` ของฟังก์ชั่นนี้เพื่อทำการจัดระเบียบและตัดส่วนที่ซ้ำกันออกไปเพื่อป้องกันความซ้ำซ้อนในตัวเอง

## มาทดสอบกันเลย

เราจะกลับไปยัง `zombiefeeding.sol` เพราะเนื่องจากตรงนี้เป็นส่วนแรกที่เราได้ใช้ไอเดียนี้ เราจะมาทำการปรับแต่งโครงสร้างของมันให้เป็น `modifier`ของตัวเอง

1. สร้าง `modifier` ที่มีชื่อว่า `ownerOf` ที่จะรับ 1 argument ซึ่งก็คือ`_zombieId` (ชนิด `uint`).

  ในส่วนของ body ควรมีการ `require` ว่า `msg.sender` นั้นเท่ากับ `zombieToOwner[_zombieId]` จากนั้นกลับไปทำต่อในส่วนของฟังก์ชั่น โดยสามารถกลับไปดูที่ `zombiehelper.sol` หากจำ syntax ของ modifier ไม่ได้

2. เปลี่ยนส่วน definition ของฟังก์ชั่นที่ชื่อ `feedAndMultiply` เพื่อให้มันใช้ modifier `ownerOf` 

3. ตอนนี้ก็ถึงเวลาที่จะได้ใช้ `modifier` แล้ว โดยสามารถลบบรรทัดที่เป็น `require(msg.sender == zombieToOwner[_zombieId]);` ออกไปได้
