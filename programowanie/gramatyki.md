# Gramatyki
operatory na językach https://www.cs.toronto.edu/~amir/teaching/csc236f15/materials/lec09.pdf
## luty 2021

$S \to SS | abSba | X$
$X \to \epsilon | cXd | cXc$

a) Czy $abbaabbac$ należy do $L(G_1)$?
Nie, bo jedyne produkcje, z których może powstać symbol terminalny $c$, to $X \to cXd$ lub $X \to cXc$, a w rozważanym ciągu mamy tylko jedno $c$ i nie mamy $d$.

b) czy $G_1$ jest jednoznaczna?
Nie, bo mamy 2 drzewa produkcji dla jednego wyrażenia $abba$.
$S \to abSbaS \to abXbaX \to abba$
$S \to S SabSba \to XabXba \to abba$

c) $(cd)^*$

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
