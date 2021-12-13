```python
import numpy as np
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 22: Crab Combat

Reference: https://adventofcode.com/2020/day/22

## Part 1

**What is the winning player's score?**


```python
cards1 = list()
cards2 = list()
mode = "Player 1"
with open('cards_input.txt', 'r') as fid:
    # Throw away first line
    fid.readline()
    for line in fid:
        if '' == line.strip():
            # Empty line
            continue
        if line.startswith("Player 2"):
            mode = "Player 2"
            continue
        if "Player 1" == mode:
            cards1.append(int(line.strip()))
        elif "Player 2" == mode:
            cards2.append(int(line.strip()))
```


```python
h1 = cards1.copy() # Player 1 hand
h2 = cards2.copy() # Player 2 hand
while(bool(h1) and bool(h2)):
    # Play until either player 1 or player 2 are out of cards
    p1, p2 = h1.pop(0), h2.pop(0)
    if p1 > p2:
        # Player 1 wins
        h1.extend([p1, p2])
    else:
        # Player 2 wins
        h2.extend([p2, p1])
```


```python
def get_score(hand):
    # Supply winning hand array-like to get the
    # score returned
    arr = np.array(hand, dtype=np.uint32)
    ptvals = np.arange(len(arr), dtype=np.uint32)[::-1] + 1
    return (arr * ptvals).sum()
```


```python
if h1:
    print("Winner: PLAYER 1")
    winner = 1
    win_hand = np.array(h1.copy(), dtype=np.uint32)
else:
    print("Winner: PLAYER 2")
    winner = 2
    win_hand = np.array(h2.copy(), dtype=np.uint32)
    
win_hand
```

    Winner: PLAYER 2
    




    array([23,  8, 49, 40, 30, 17, 43, 34, 31, 13, 44, 32, 36, 20, 25, 11, 26,
           18, 21, 16, 48, 19, 24,  4, 38, 37, 33, 14, 29, 10,  7,  6, 47, 46,
           45, 39,  9,  2, 50, 12, 42,  1, 41,  5, 28,  3, 35, 27, 22, 15],
          dtype=uint32)




```python
#Markdown("Player {} wins with a score of **{}**".format(winner, get_score(win_hand)))
```

## Part 2

**What is the winning player's score?**


```python
test_cards1 = [9, 2, 6, 3, 1]
test_cards2 = [5, 8, 4, 7, 10]
```


```python
# Construct recursive function:
def play_combat(cards1, cards2):
    #print("START OF ROUND: ", cards1, cards2)
    h1 = cards1.copy() # Player 1 hand
    h2 = cards2.copy() # Player 2 hand
    handlog = set() # To keep track of hands per subgame
    # Play regular combat
    while(bool(h1) and bool(h2)):
        #print("START OF ROUND: ", h1, h2)
        # Convert the hands to tuples so they are hashable and
        # can be added to the handhash set. If they are already
        # in the set the round is already over and player one wins
        handhash = (tuple(h1), tuple(h2))
        if handhash in handlog:
            #print("PREVIOUSLY SEEN HANDS: ", h1, h2)
            winner, win_hand = (1, h1)
            break
        handlog.add(handhash)
        p1, p2 = h1.pop(0), h2.pop(0)
        # Play until either player 1 or player 2 are out of cards
        # For the cards that were dealt this round, check whether
        # the players have at least that many cards remaining in their
        # hands. If either do not, proceed to play regular combat.
        # If both do, recurse
        if len(h1) >= p1 and len(h2) >= p2:
            # Play recursive combat
            #print("RECURSING")
            winner, win_hand = play_combat(h1[:p1], h2[:p2])
            if 1 == winner:
                # Player 1 wins recursive combat
                h1.extend([p1, p2])
            else:
                # Player 2 wins recursive combat
                h2.extend([p2, p1])
            #print("AFTER RECURSING: ", h1, h2)
        elif p1 > p2:
            # Player 1 wins the round
            h1.extend([p1, p2])
            #print(1, h1, h2)
        else:
            # Player 2 wins the round
            h2.extend([p2, p1])
            #print(2, h1, h2)
    if h1:
        # Player 1 wins the hand
        #print(1, p1, p2)
        winner, win_hand = (1, h1)
    else:
        # Player 2 wins the hand
        #win_hand = np.array(h2.copy(), dtype=np.uint32)
        #print(2, p2, p1)
        winner, win_hand = (2, h2)
    #print("END SUBGAME: ", winner, win_hand)
    #print(h1, h2)
    return winner, win_hand
```


```python
#play_combat(test_cards1, test_cards2)
winner2, win_hand2 = play_combat(cards1, cards2)
```


```python
#Markdown("Player {} wins with a score of **{}**".format(winner2, get_score(win_hand2)))
```


```python

```
