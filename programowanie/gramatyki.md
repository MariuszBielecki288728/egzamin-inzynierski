# Gramatyki
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

