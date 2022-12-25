tags: [[codewars]] 

![[Pasted image 20221207100426.png]]

Mine:
```
function countBy($x, $n) {
    $z = [];
  
    for ($i = 1; $i <= $n; ++$i) {
      $z[] = $i * $x;
    }

    return $z;
}
```

The Best:
```
function countBy($x, $n) {
    return range($x, $n * $x, $x);
}
```

function range - Создаёт массив, содержащий диапазон элементов

range - **range**(string|int|float `$start`, string|int|float `$end`, int|float `$step` = 1): array