CS146 tutorial 3 January 17 2018

```
data [t] = []
         | t:[t]
```

```
prepend :: Integer -> [[Integer]] -> [[Integer]]
prepend i [] = []
prepend i (x:xs) = (i:x):y
              where y = prepend i xs

prepend 1 [[2],[3]]
-> [[1, 2], [1, 3]]
```
```
insert :: Integer -> [Integer] -> [[Integer]]
insert i [] = [[i]]
insert i y@(x:xs) = (i:y):z
              where
                ys = insert i xs
                z = prepend x ys

insert 1 [2, 3, 4]
-> [[1, 2, 3, 4], [2, 1, 3, 4], [2, 3, 1, 4], [2, 3, 4, 1]]
```
```
add :: Integer -> [[Integer]] -> [[Integer]]
add _ [] = []
add i (x:xs) =
        let a = insert i x
            b = add i xs
        in a++b

add 1 [[2, 3], [4, 5]]
-> [[1, 2, 3], [2, 1, 3], [2, 3, 1], [1, 4, 5], [4, 1, 5], [4, 5, 1]]
```
```
perm :: [Integer] -> [[Integer]]
perm [] = [[]]
perm (x:xs) = add x (perm xs)
```
```
comb :: [Integer] -> Integer -> [[Integer]]
comb y i
    | i == 0 = []
    | y == [] = []
    | otherwise = a++c
              where a = comb (tail y) i
                    b = comb (tail y) (i - 1)
                    c = prepend (head y) b

comb [2, 3, 4] 2
-> [2, 3], [2, 4], [3, 4]
```
```
[("Alice", (1414, "cafebake")),
 ("Bob", (2818, "deadheat")),
 ("Eve", (3141, "bandfood"))]
... (t1, t2)

data Maybe t2 = Just t2
              | Nothing

lookup :: t_1 -> [t1, t2] -> Maybe t2

lookupX :: String -> [(String, (Integer, String))] -> (Integer, String)
lookupX a b =
        case (lookup a b) of
          Just v -> v
          Nothing -> error "bad"

lookup a b =
    (fromMaybe "Empty" (lookup a b))
```

```
/u/cs146/pub/marmoset_submit cs146 Q1 Q1.rkt
w3m marmoset.student.cs.uwaterloo.ca
```
```
[1..100]
[2,4..11]
[x | x <- [1..100], mod x 13 == 0]
[(x, y, z) | x <- [1..100], y <- [x..100], z <- [y..100], x^2 + y^2 == z^2]

zipWith + [1, 2] [3, 4] -> [4, 6]

fibonnaci = 0:1:ZipWith + helper (tail helper)
```
