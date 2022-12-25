tags: [[codewars]] 

![[Pasted image 20221207101557.png]]

Mine:
```
function grow($a) {
  return array_reduce($a, fn($carry, $item) => $carry *= $item, 1);
}
```

The best:
```
function grow($a) {
  return array_product($a);
}
```

array_reduce - принимает массив, callback и стартовое значение (не обязательное), в callback передается два аргумента $carry - переменная которая сохраняет состояение, $item текущий элемент массива.

array_product - возвращает произведение всех элементов.