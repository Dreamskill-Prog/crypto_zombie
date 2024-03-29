---
title: Withdraws
actions: ['checkAnswer', 'hints']
requireLogin: true
material:
  editor:
    language: sol
    startingCode:
      "zombiehelper.sol": |
        pragma solidity ^0.4.19;

        import "./zombiefeeding.sol";

        contract ZombieHelper is ZombieFeeding {

          uint levelUpFee = 0.001 ether;

          modifier aboveLevel(uint _level, uint _zombieId) {
            require(zombies[_zombieId].level >= _level);
            _;
          }

          // 1. สร้างฟังก์ชั่นสำหรับการถอนตรงนี้

          // 2. สร้างฟังก์ชั่น setLevelUpFee ตรงนี้

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
---

ในหัวข้อก่อนหน้า เราได้เรียนรู้ถึงการส่ง Ether ไปยัง contract แล้วจะเกิดอะไรขึ้นหลังจากเราส่งมันไปล่ะ?

หลังจากที่ได้ส่ง Ether ไปยัง contract มันจะถูกเก็บไว้ภายใน Ethereum account ของ contract จะก็จะอยู่แบบนั้นไปเรื่อย ๆ — หากเราไม่ได้เพิ่มฟังก์ชั่นในการถอน Ether ออกมาจาก contract อื่น

สามารถเขียนฟังก์ชั่นในการถอน Ether ออกมาจาก contract ได้ตามนี้:

```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```

โน้ตไว้ว่าเรากำลังใช้ `owner` และ `onlyOwner` จาก contract ชื่อ `Ownable` โดยสมมติว่าเราได้ทำการอิมพอร์ตมันออกมา

เราสามารถโอน Ether ไปยัง address โดยการใช้ฟังก์ชั่น `transfer` และ `this.balance` จะทำการรีเทิร์นยอดคงเหลือทั้งหมดที่ได้เก็บไว้ภายใน contract ออกมา ดังนั้นหากผู้ใช้ 100 คนได้ชำระ 1 Ether มายัง contract ของเรา `this.balance` ก็จะมีค่าเท่ากับ 100 Ether

อาจใช้ `transfer` ในการโอนเงินไปยัง Ethereum address ใด ๆ ก็ได้อีกด้วย ่เช่น เราอาจสามารถมีฟังก์ชั่นที่โอน Ether กลับไปยัง `msg.sender` หากมีเหตุการณ์ของการจ่ายเงินเกินเข้ามา:

```
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```

หรือแม้แต่ภายใน contract ที่มีผู้ซื้อและผู้ขาย ก็อาจจะมีการบันทึก address ของผู้ขายเอาไว้ภายในหน่วยความจำ ทำให้สามารถสรุปยอดที่ได้รับไปยังผู้ขายได้ เมื่อมีคนเข้ามาซื้อสินค้าของเขา: `seller.transfer(msg.value)`.

ทั้งหมดนี้ก็เป็นตัวอย่างบางส่วนที่บอกเราว่าโปรแกรมมิ่งบน Ethereum นั้นเจ๋งมาก ๆ อย่างไร — เราจะสามารถมี decentralized marketplace แบบนี้ซึ่งจะไม่ถูกควบคุมโดยผู้ใดผู้หนึ่ง

## ถึงเวลาทดสอบ

1. สร้างฟังก์ชั่น `withdraw` ภายใน contract ที่หน้าตาเหมือนกับตัวอย่างของฟังก์ชั่น `GetPaid` ด้านบน

2. ราคาของ ether มีค่าสูงเกิน 10x ในปีที่ผ่านมา ในขณะที่ 0.001 ether มีค่าประมาณ $1 ณ ตอนที่ได้เขียนบทเรียนนี้ หากมันมีค่า 10x อีกรอบจะทำให้ 0.001 ETH ของเรามีค่าเท่ากับ $10 และเกมก็จะมีมูลค่าสูงขึ้นตามไป

  ดังนั้นจึงเป็นเรื่องดีที่เราจะต้องสร้างฟังก์ชั่นที่ยอมให้เรา ผู้ซึ่งเป็นเจ้าของของ contract สามารถแก้ไข `levelUpFee` ได้

  a. สร้างฟังก์ชั่นที่ชื่อว่า `setLevelUpFee` ที่จะรับ 1  argument `uint _fee` เป็นแบบ `external` และใช้ modifier `onlyOwner`

  b. ฟังก์ชั่นนี้ตั้งให้ `levelUpFee` มีค่าเท่ากับ `_fee`
