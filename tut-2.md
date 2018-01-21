CS146 Tutorial 2
Jan 10, 2018

\> x - Save output to X  
< x - Read input from X  
x | y - Use output of X as input of Y  
x && y - Execute X and Y left to right  
diff - Compare Files

```Bash
for i in {1..2}, do __ $i __; done
```
# Haskell
```Haskell
data Lst = Empty
         | Cons Integer Lst

len:: Lst -> Integer
len Empty = 0
len (Cons _ l) = 1 + len l

map:: (Integer -> Integer) -> Lst -> Lst
map _ Empty = Empty
map f (Cons i l) = (Cons (f i) (map f l))

foldl:: (Integer -> Integer -> Integer) -> Integer -> Lst -> Integer
foldl _ a Empty = a
foldl f a (Cons i l) = foldl f a' l
        where a' = f a i

l1 = (Cons 2 (Cons 4 (Cons 4 Empty)))

len l1
map (*16) l1
foldl (\x y -> x^y) 2 l1
```

[] -> built-in Empty  
: -> built-in Cons
```Haskell
(2:(4:(8:[])))
[2, 4, 8]
```
