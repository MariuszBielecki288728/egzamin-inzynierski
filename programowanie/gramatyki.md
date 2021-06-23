# Programowanie
operatory na językach https://www.cs.toronto.edu/~amir/teaching/csc236f15/materials/lec09.pdf
## Luty 2021, wariant B

### Zadanie 1
$S \to SS | abSba | X$  
$X \to \epsilon | cXd | cXc$

a) Czy $abbaabbac$ należy do $L(G_1)$?
Nie, bo jedyne produkcje, z których może powstać symbol terminalny $c$, to $X \to cXd$ lub $X \to cXc$, a w rozważanym ciągu mamy tylko jedno $c$ i nie mamy $d$.

b) czy $G_1$ jest jednoznaczna?
Nie, bo mamy 2 drzewa produkcji dla jednego wyrażenia $abba$.  
$S \to abSbaS \to abXbaX \to abba$  
$S \to S SabSba \to XabXba \to abba$  

c) $(cd)^*$ - jak do tego dojść? Rozważamy wyrażenie regularne i zastanawiamy się które ze słów nim opisanych da się wyprodukować za pomocą gramatyki. W tym przypadku wszystkie wyrażenia z $a$ opisane za pomocą tego wyrażenia odpadają, bo produkcje w $G_1$ zawsze generują $a$ razem z $b$, którego to wyrażenie na pewno nie opisuje.

d) <https://web.stanford.edu/class/archive/cs/cs103/cs103.1132/lectures/17/Small17.pdf>
Zostają nam tylko produkcje produkujące $a$, $b$ lub $c$.  
$S \to SS | abSba | X$  
$X \to \epsilon | cXc$  
może Earley parser? a może jakaś ifologia?

```python
def try_easy(string):
    # ab has to have matching number of ba
    ab_counter = 0
    drain_only = False  # in drain mode only ba is permitted unless ab_counter is 0
    while True:
        if len(string) == 0:
            return ab_counter == 0  # check result
        if len(string) == 1 and string[0] != "c" or ab_counter < 0:
            return False  # lonely letter that is not c or ba appears more time than ba (also when ba is first)
        elif string[0] == "c":
            if drain_only is True and ab_counter != 0:
                return False  # when in drain mode, there shoub be only ba occurence, or else the recorsive pattern abRba is done, so check ab_counter
            if ab_counter != 0:
                drain_only = True
            c_counter = 0
            while string and string[0] == "c":
                c_counter += 1
                string = string[1:]
            if c_counter % 2 != 0:
                return False

            continue

        elif string[:2] == "ab" and drain_only is False:
            ab_counter += 1
        elif string[:2] == "ab" and drain_only is True:
            if ab_counter != 0:
                return False
            else:
                drain_only = False
                ab_counter += 1

        elif string[:2] == "ba":
            if drain_only is False:
                drain_only = True
            ab_counter -= 1

        string = string[2:]
```
### Zadanie 2
Napisz w Haskellu funkcje moda, która dla listy liczb znajduję tę, która występuje najwięcej razy.
```haskell=
import Data.Maybe
import Data.List

find_most_ocurrent :: Eq a => [a] -> Int -> Int -> a -> a -> a
find_most_ocurrent [] best_acc acc best_item current_item
    | acc > best_acc = current_item
    | acc <= best_acc = best_item

find_most_ocurrent (x : xs) best_acc acc best_item current_item
    | x == current_item = find_most_ocurrent xs best_acc (acc + 1) best_item current_item
    | x /= current_item && acc > best_acc = find_most_ocurrent xs acc 1 current_item x
    | x /= current_item && acc <= best_acc = find_most_ocurrent xs best_acc 1 best_item x


moda :: Ord a => [a] -> Maybe a
moda [] = Nothing
moda l@(h:t) = Just (find_most_ocurrent (sort l) 0 0 h h)
```
### Zadanie 3
Napisz w prologu predykat my_is, który działa tak, jakby był zdefiniowany w następujący sposób:
```prolog
my_is(V, E) :- V is E
```
Nie można korzystać z is i innych predykatów arytmetycznych, ale można z poniższych:
```prolog
add(X, Y, Z) :- number(X), number(X), Z is X + Y.
mult(X, Y, Z) :- number(X), number(X), Z is X * Y.
div(X, Y, Z) :- number(X), number(X), Z is X / Y.
minus(X, Y, Z) :- number(X), number(X), Z is X - Y.
```
Sposób z CDG, kłopot jest taki, że najpierw trzeba stokenizować wyrażenie do listy atomów.
```prolog
% https://www.cs.auckland.ac.nz/courses/compsci220s1t/archive/compsci220ft/lectures/GGlectures/220ch4_gramm.pdf
% 4.5 An Unambiguous Grammar for Expressions

%https://www3.cs.stonybrook.edu/~warren/xsbbook/node24.html
% DCG with an evaluator

:- table expr/3, term/3.

expr(Val) --> expr(Eval), [+], term(Tval), {add(Eval, Tval, Val)}.
expr(Val) --> expr(Eval), [-], term(Tval), {minus(Eval, Tval, Val)}.
expr(Val) --> term(Val).
term(Val) --> term(Tval), [*], factor(Fval), {mult(Tval, Fval, Val)}.
term(Val) --> term(Tval), [/], factor(Fval), {div(Tval, Fval, Val)}.
term(Val) --> factor(Val).
factor(Val) --> ['('], expr(Val), [')'].
factor(Int) --> [S], {atom(S), atom_number(S, Int)}.

my_is(V, E) :- with_output_to(chars(Echars), write(E)), expr(V, Echars, []).
```
O wiele fajniejsza opcja to wykorzystanie unifikacji termów już zaimplementowanych w prologu, czyli standardowych operatorów. Prolog ma już poprawnie zdefiniowaną gramatykę dla tych operatorów, więc nie musimy się martwić o priorytet i łączność.
```prolog
my_is2(E, E) :- atomic(E).
my_is2(V, LE + RE) :- my_is2(LV, LE), my_is2(RV, RE), add(LV, RV, V).
my_is2(V, LE - RE) :- my_is2(LV, LE), my_is2(RV, RE), minus(LV, RV, V).
my_is2(V, LE * RE) :- my_is2(LV, LE), my_is2(RV, RE), mult(LV, RV, V).
my_is2(V, LE / RE) :- my_is2(LV, LE), my_is2(RV, RE), div(LV, RV, V).
```
## Czerwiec 2020, wariant B

### Zadanie 1
Gramatyka $G_1$ z symbolem startowym $S$ nad alfabetem $\{a, b, c\}$:  
$S \to X \mid cSc$  
$X \to aaX \mid bbX \mid XX \mid \epsilon$  
Gramatyka $G_2$ z symbolem startowym $S$ nad alfabetem $\{a, b, c\}$:  
$S \to X \mid ccScc$  
$X \to aaaX \mid bbbX \mid \epsilon$

a) **Czy $ccaabbcc$ należy do $L(G_1)$?**  
Tak, bo: $S \to cSc \to ccScc \to ccXcc \to ccaaXcc \to ccaabbXcc \to ccaabbcc$

b) **Czy $G_2$ jest jednoznaczna?**
Tak, ponieważ każde słowo w języku $L(G_2)$ można wyprowadzić tylko na jeden sposób. (definicja jednoznaczności, no ale miało być krótko :C)

Prefiksy ustalają, która produkcja zostanie użyta, więc podciąg zaczyna się na cc, to nie ma możliwości wybrania innej produkcji. analogicznie z aaa... i bbb... i słowem pustym.

c) **Czy można usunąć jakąś produkcję z $G_1$, nie zmieniając generowanego przez nią języka?**  
gramatyka generuje podciągi aa, aaaa, bb, bbbb, aabb, bbaa... (gdzie każdy podciąg następujących po sobie tych samych literek musi mieć parzystą długość) otoczone ciągami c o równej długości.
taki sam język uzyskamy usuwając produkcję $X \to XX$, bo możemy ją zastąpić kombinacjami produkcji $X \to aaX$ oraz $X \to bbX$

d) **Napisz gramatykę $G_3$ taką, że $L(G_1)\cap L(G_2) = L(G_3)$**  
Korzystając z opisu słownego z poprzedniego podpunktu, dotychczas (w $G_1$) ciągi literek c (otaczające pozostałe literki) mogły mieć dowolną długość, pod warunkiem,że była ta sama po obu stronach słowa. Po przecięciu tych języków, $G_2$ nakłada nam dodatkową restrykcję - ciągi następujacych po sobie "c", muszą mieć parzystą długość. Z ciągami następujących po sobie "a" lub "b" jest troszkę ciekawiej. Ciągi te wyprodukowane przez $G_1$ muszą mieć długość podzielną przez 2 (bo dwójkami są generowane), a wyprodukowane przez 
$G_2$ -- muszą mieć długość podzielną przez 3. A więc po przecięciu tych zbiorów, zostają tylko ciągi podzielne zarówno przez 2, jak i przez 3, czyli podzielne przez 6.  

$S \to X \mid ccScc$  
$X \to aaaaaaX \mid bbbbbbX \mid \epsilon$

e)
```python=
def check(string: str) -> bool:
    cc_occurences = 0
    while len(string) > 1 and string[:2] == "cc":
        cc_occurences += 1
        string = string[2:]
    if not string:
        # jeśli są same c, to musi być parzysta liczba dwójek
        return cc_occurences % 2 == 0

    while string:
        if string[:6] == 6 * "a":
            while string[:6] == 6 * "a":
                string = string[6:]
        elif string[:6] == 6 * "b":
            while string[:6] == 6 * "b":
                string = string[6:]
        else:
            break
    while len(string) > 1 and string[:2] == "cc":
        cc_occurences -= 1
        string = string[2:]
    return cc_occurences == 0 and len(string) == 0

```

### Zadanie 2
Napisz w prologu predykat no_repetition(L), który (dla stałej listy L) kończy się sukcesem wtedy i tylko wtedy, gdy żaden element na liście L się nie powtarza. Predykat powinien być napisany bez użycia rekurencji, można jednak użyć predykatów append, member i negacji.

```prologhttps://hackmd.io/bwoRs40WTm-VQ3rFH6FERA#Zadanie-21
no_repetition(L) :- 
    \+ (append(_, [X | More], L), member(X, More)).
```

# Luty 2020

Zadanie 3
Napisz program obliczający wartość wyrażenia, złożonego z nawiasów, stałych 0, 1, oraz operatorów + i *. + to logiczny $or$, a * to logiczny $and$. W prologu trzeba napisać predykat evaluate(+Expression, -Value).

Poniżej implementacja używająca CDG, ale polecam użyć sposobu z zadania o my_is i pozwolić prologowi zająć się odpowiednimi priorytetami i łącznościami.

```prolog
:- table expr/3, term/3.

expr(1) --> expr(1), [+], term(0).
expr(1) --> expr(1), [+], term(1).
expr(1) --> expr(0), [+], term(1).
expr(0) --> expr(0), [+], term(0).
expr(Val) --> term(Val).
term(Val) --> term(LVal), [*], factor(RVal), {Val is LVal * RVal}.
term(Val) --> factor(Val).
factor(Val) --> ['('], expr(Val), [')'].
factor(0) --> ['0'].
factor(1) --> ['1'].

evaluate(Expression, Value) :- with_output_to(chars(Echars), write(Expression)), expr(Value, Echars, []).

%  ?- evaluate(1+1+1*0, Value).
%  Value = 1.
```