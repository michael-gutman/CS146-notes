CS146 Tutorial 4 Jan 24 2018
AST

```
data Exp = Lit Int
         | Bin Op Exp Exp

data Op = Add | Mult

1 + 2 * (3 + 4)
->
Bin Add (Lit 1) (Bin Mult (Lit 2) (Bin Add (Lit 3) (Lit 4)))

opTrans :: Op -> (Int -> Int -> Int)
opTrans Add = (+)
opTrans Mult = (*)

interp :: Exp -> Int
interp (Lit i) = i
interp (Bin o a b) = (opTrans o) (interp a) (interp b)
```
***
```
data Exp = Lit Int
         | Bin Op Exp Exp
         | Fun String Exp
         | With (String Exp) Exp
         | App Exp Exp
         | Var String

(With (x (Fun y (Bin Add (Var y) (Lit 1))))
      (App (Var x) (Lit 5)))

subst :: String -> Exp -> Exp -> Exp
subst v e@(Var x) i
  if v == x i else e
subst _ v@(Lit _) _ = v
subst v e@(App p b) a = (App p' b')
  where p' = subst v p a,
        b' = subst v b a
subst v (Fun p b) a
  if v == p (Fun p b) else
    subst v b a

interp :: Exp -> Int
interp (Lit i) = i
interp (Bin o a b) = (opTrans o) (interp a) (interp b)
interp (App f a) = subst p b a
  where (Fun p b) = interp f
    e = interp a
interp (With (a b) c) =
  interp (App (Fun a c) b)
interp v@(Var i) = v
```
