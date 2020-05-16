# CSE 311A Programming Assignment 2

## Question 1

Part 1: Program run output, generated plots, and other data [here](#Question-1-Implementation).

Part 2: Looking at the plots, I noticed the following trends:

- Initially, players of type AC are at a disadvantage and die due to their significant losses in games against type AD, but they are replaced with types T4T and G (which perform well in cooperating with each other and AC, and can manage losses against AD).

- Then, as the game progresses, players of the type AD are driven to extinction. This is because all of the other player types get a large advantage of constant cooperation (+3) when playing against each other. Ignoring the first turns with TFT and G, AD gets a small payoff (+1) with these players. Despite the large payoff AD gets from games with AC (+5), this is not enough to recover this relative loss because AC's population is small from deaths in the early generations.

- The game stagnates after the extinction of AD players, because the only actions left are cooperation (AC, G, and T4T will always cooperate with each other). At this point, there is only a random replacement of "top" performers, which is just a random selection of all players.

## Question 2

Part 1: Program run output, generated plots, and other data [here](#Question-2-Implementation).

Part 2: Looking at the plots, I noticed the following trends for each distribution:

First Distribution: One Grudger player, one Always Cooperate player, and n-2 Tit-for-Tat players

- In this example, there is a stagnation of player populations and payoffs from the first generation onwards, and a small probabilistic chance of extinction for any of the player types. As the analysis in question 1 reveals, in a game of T4T, G, and AC players, the result is games where every player always cooperates. As a result, the only chance of extinction comes from the probability that the last surviving players of a type end up in the bottom p% (which is meaningless since they all have the same payoffs).

Second Distribution: One Grudger player, one Always Cooperate player, and n-2 Always Defect players

- Due to their small numbers, the G and AC players go extinct after the first generation. Unlike in question 1, there aren't enough cooperating-type players to balance out the penalty from AD players doing extremely well against AC players and somewhat well against G players. After this, AD rules supreme forever.

Third Distribution: Mostly AD, with some G, and very little AC, and a tiny amount of T4T

- Initially, AC dies and is replaced with AD. This occurs because AC is a comparatively worse performer due to its losses against AD type players. Unlike in other circumstances, the large number of AD players makes these losses too significant even with fellow cooperating-style players T4T and G.

- Once AC dies, AD begins its slow demise. T4T and G, with their cooperating nature in games with each other, grow their payoffs faster than AD can handle the post-first game stalemate with these players. However, given AD's high starting number of players and the replacement rate, this takes some time.

## Question 3

Part 1: Program run output, generated plots, and other data [here](#Question-3-Implementation).

Part 2: Looking at the plots, I noticed the following trends

- Up to a $p$ value of ~15, the same trends noticed in question 1 progress at relatively faster rates as p increases. This is because the amount of replacement occurring after each generation causes change to occur more rapidly and there is a faster replacement of the poor performers (AC and AD).

- After a $p$ value of ~15, a new trend emerges where both AD and AC become extinct. Only T4T and G survive. Since both AC and AD end up in the bottom performers, they die. However, since are still some AD and AC at the same time in the last generation where both exist, AC is a comparatively worse performer due to its losses against the soon-to-die AD players. Unlike in question 1, AC players cannot outlive AD players to become equivalent performers to T4T and G players.

## Program Code


```python
import random
from itertools import islice
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
%matplotlib inline

# True: Cooperate, False: Defect
class Player:
    payoff = 0
    def reset(self):
        self.choice = self.og_choice
    def fullreset(self):
        self.choice = self.og_choice
        self.payoff = 0
    def update(self, opponent_choice):
        pass
    
# Tit-for-Tat (T4T) players
# Start by cooperating and play the opponent’s last action in subsequent rounds
class T4T(Player):
    og_choice = True
    choice = True # Defaults to cooperating, should change to opponent's last action
    def update(self, opponent_choice):
        self.choice = opponent_choice
        
# Grudger (G) players
# Cooperate until the opponent defects, after which it only defects
class G(Player):
    og_choice = True
    choice = True # Defaults to cooperating, should change to defection as response
    def update(self, opponent_choice):
        if not opponent_choice:
            self.choice = False
        
# Always Cooperate (AC) players
# Always cooperate
class AC(Player):
    og_choice = True
    choice = True

# Always Defect (AD) players
# Always defect
class AD(Player):
    og_choice = False
    choice = False
        
```


```python
def simulate_generation(players, m):
    # Players needs to be an input list of already made player objects
    already_visited = 0
    for one in players:
        already_visited += 1
        for two in islice(players, already_visited, None):
            for _ in range(m):
                if one.choice and two.choice:
                    one.payoff += 3
                    two.payoff += 3
                elif not one.choice and not two.choice:
                    one.payoff += 1
                    two.payoff += 1
                elif one.choice:
                    two.payoff += 5
                elif two.choice:
                    one.payoff += 5
                else:
                    print("Never be here!")

                one_choice = one.choice
                two_choice = two.choice
                one.update(two_choice)
                two.update(one_choice)

            # Resets player state between games
            one.reset()
            two.reset()

def birth(players, m, p, k, printing=True):
    reset_global_stats()
    p = p / 100
    generation = players
    for i in range(k):
        simulate_generation(generation, m)
        if printing:
            print("Gen ", (i+1),": ", sep="", end="")
        print_stats(generation, i+1, printing)
        
        random.shuffle(generation) # Allows easy tie-breaking
        generation.sort(key=lambda x: x.payoff)
        p_num_players = int(len(generation) // (1/p))
        del generation[:p_num_players]
        new_players = []
        for new in generation[-p_num_players:]:
            new_players.append(type(new)())
        generation = generation + new_players
        for player in generation:
            player.fullreset()
        
```


```python
stackplot_type_percentage = [[], [], [], []]
stackplot_type_total = [[], [], [], []]
multiline_type_avg = [[], [], [], []]
unknown_type_payoff_total = [] # Unused since captured by stackplot_type_total

def reset_global_stats():
    # Access to globals
    global stackplot_type_percentage
    global stackplot_type_total
    global multiline_type_avg
    global unknown_type_payoff_total
    stackplot_type_percentage = [[], [], [], []]
    stackplot_type_total = [[], [], [], []]
    multiline_type_avg = [[], [], [], []]
    unknown_type_payoff_total = []

def print_stats(players, k, printing):
    # Access to globals
    global stackplot_type_percentage
    global stackplot_type_total
    global multiline_type_avg
    global unknown_type_payoff_total
    
    # Always in T4T, G, AC, AD order
    n = len(players)
    player_types = ['T4T', 'G', 'AC', 'AD']
    player_types_num = np.array([0, 0, 0, 0])
    player_types_payoff = np.array([0, 0, 0, 0])
    for player in players:
        if type(player) is T4T:
            player_types_num[0] += 1
            player_types_payoff[0] += player.payoff
        elif type(player) is G:
            player_types_num[1] += 1
            player_types_payoff[1] += player.payoff
        elif type(player) is AC:
            player_types_num[2] += 1
            player_types_payoff[2] += player.payoff
        elif type(player) is AD:
            player_types_num[3] += 1
            player_types_payoff[3] += player.payoff
        else:
            print("Nope! Not here")
    
    player_types_num = (player_types_num * 100) // n
    
    if printing:
        for i in range(len(player_types)):
            print(player_types[i], ": ", player_types_num[i], "%  ", sep="", end="")
        print()


        print("      " + " "*len(str(k)), end="")
        for i in range(len(player_types)):
            print(player_types[i], ": ", player_types_payoff[i], "  ", sep="", end="")
        print("Total: ", sum(player_types_payoff), end="")
        print()
        print("      " + " "*len(str(k)), end="")
    
    for i in range(4):
        stackplot_type_percentage[i].append(player_types_num[i])
        stackplot_type_total[i].append(player_types_payoff[i])
    unknown_type_payoff_total.append(sum(player_types_payoff))
    
    
    with np.errstate(invalid='ignore', divide='ignore'):
        # player_types_payoff = np.nan_to_num(player_types_payoff / player_types_num)
        # Corrects graphing issue
        player_types_payoff = (player_types_payoff / player_types_num)

    for i in range(4):
        multiline_type_avg[i].append(player_types_payoff[i])
        
    if printing:
        for i in range(len(player_types)):
            print(player_types[i], ": ", player_types_payoff[i], "  ", sep="", end="")
        print()

```


```python
def gen_n_player_evenly(n):
    players = []
    for _ in range(n // 4):
        players.append(T4T())
        players.append(G())
        players.append(AC())
        players.append(AD())
    return players
    
```


```python
def plot_globals():    
    # Plot percentage of population by type.
    labels = ['T4T', 'G', 'AC', 'AD']
    
    _, ax = plt.subplots()
    x_axis_vals = np.arange(1, len(stackplot_type_percentage[0])+1)
    x_axis_ticks = np.arange(1, len(stackplot_type_percentage[0])+1, step=(len(stackplot_type_percentage[0])+1) // 10)
    
    ax.stackplot(x_axis_vals, stackplot_type_percentage, labels=labels, step='mid')
    
    ax.legend(loc='upper right')
    ax.set_xlabel('Generation')
    ax.set_ylabel('Percentage')
    ax.set_title('Population by Type')
    ax.set_xticks(x_axis_ticks)
    plt.show()
    
#     plt.clf()
    
    _, ax = plt.subplots()
    ax.stackplot(np.arange(1, len(stackplot_type_percentage[0])+1), stackplot_type_total, labels=labels, step='mid')
    ax.legend(loc='upper right')
    ax.set_xlabel('Generation')
    ax.set_ylabel('Payoff')
    ax.set_title('Payoff by Type and Total Payoff')
    ax.set_xticks(x_axis_ticks)
    plt.plot()
    plt.show()
    
#     plt.clf()
    
    _, ax = plt.subplots()
    ax.set_title("Average Payoff by Type")
    ax.set_xlabel("Generation")
    ax.set_ylabel("Average Payoff")
    ax.set_xticks(x_axis_ticks)
    line_type = ['-', '--', ':', '-.']
    for i in range(len(multiline_type_avg)):
        plt.plot(x_axis_vals, multiline_type_avg[i], line_type[i], label=labels[i], marker='o')
    ax.legend(loc="upper right")
    plt.show()


```

## Question 1 Implementation
Back To Top [JUMP](#Question-1)


```python
# Running the program with specified parameters

n = 100 # Starting num of players
m = 5 # Rounds between players
p = 5 # Pruning between generations
k = 20 # Generations
players = gen_n_player_evenly(n)
birth(players, m, p, k)
```

    Gen 1: T4T: 25%  G: 25%  AC: 25%  AD: 25%  
           T4T: 30250  G: 30250  AC: 27750  AD: 29875  Total:  118125
           T4T: 1210.0  G: 1210.0  AC: 1110.0  AD: 1195.0  
    Gen 2: T4T: 29%  G: 26%  AC: 20%  AD: 25%  
           T4T: 35090  G: 31460  AC: 22200  AD: 27875  Total:  116625
           T4T: 1210.0  G: 1210.0  AC: 1110.0  AD: 1115.0  
    Gen 3: T4T: 32%  G: 28%  AC: 15%  AD: 25%  
           T4T: 38720  G: 33880  AC: 16650  AD: 25875  Total:  115125
           T4T: 1210.0  G: 1210.0  AC: 1110.0  AD: 1035.0  
    Gen 4: T4T: 35%  G: 30%  AC: 15%  AD: 20%  
           T4T: 44275  G: 37950  AC: 17775  AD: 21100  Total:  121100
           T4T: 1265.0  G: 1265.0  AC: 1185.0  AD: 1055.0  
    Gen 5: T4T: 37%  G: 33%  AC: 15%  AD: 15%  
           T4T: 48840  G: 43560  AC: 18900  AD: 16125  Total:  127425
           T4T: 1320.0  G: 1320.0  AC: 1260.0  AD: 1075.0  
    Gen 6: T4T: 39%  G: 36%  AC: 15%  AD: 10%  
           T4T: 53625  G: 49500  AC: 20025  AD: 10950  Total:  134100
           T4T: 1375.0  G: 1375.0  AC: 1335.0  AD: 1095.0  
    Gen 7: T4T: 42%  G: 38%  AC: 15%  AD: 5%  
           T4T: 60060  G: 54340  AC: 21150  AD: 5575  Total:  141125
           T4T: 1430.0  G: 1430.0  AC: 1410.0  AD: 1115.0  
    Gen 8: T4T: 46%  G: 39%  AC: 15%  AD: 0%  
           T4T: 68310  G: 57915  AC: 22275  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 9: T4T: 47%  G: 37%  AC: 16%  AD: 0%  
           T4T: 69795  G: 54945  AC: 23760  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 10: T4T: 45%  G: 39%  AC: 16%  AD: 0%  
            T4T: 66825  G: 57915  AC: 23760  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 11: T4T: 43%  G: 41%  AC: 16%  AD: 0%  
            T4T: 63855  G: 60885  AC: 23760  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 12: T4T: 41%  G: 42%  AC: 17%  AD: 0%  
            T4T: 60885  G: 62370  AC: 25245  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 13: T4T: 39%  G: 43%  AC: 18%  AD: 0%  
            T4T: 57915  G: 63855  AC: 26730  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 14: T4T: 40%  G: 41%  AC: 19%  AD: 0%  
            T4T: 59400  G: 60885  AC: 28215  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 15: T4T: 41%  G: 40%  AC: 19%  AD: 0%  
            T4T: 60885  G: 59400  AC: 28215  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 16: T4T: 44%  G: 38%  AC: 18%  AD: 0%  
            T4T: 65340  G: 56430  AC: 26730  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 17: T4T: 45%  G: 37%  AC: 18%  AD: 0%  
            T4T: 66825  G: 54945  AC: 26730  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 18: T4T: 47%  G: 35%  AC: 18%  AD: 0%  
            T4T: 69795  G: 51975  AC: 26730  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 19: T4T: 47%  G: 38%  AC: 15%  AD: 0%  
            T4T: 69795  G: 56430  AC: 22275  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 20: T4T: 46%  G: 37%  AC: 17%  AD: 0%  
            T4T: 68310  G: 54945  AC: 25245  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  



```python
# Generating plots

plot_globals()
```


![png](img/output_12_0.png)



![png](img/output_12_1.png)



![png](img/output_12_2.png)


## Question 2 Implementation
Back To Top [JUMP](#Question-2)

### First Distribution: One Grudger player, one Always Cooperate player, and n-2 Tit-for-Tat players


```python
# Running the program with specified parameters
n = 100
m = 5 # Rounds between players
p = 5 # Pruning between generations
k = 20 # Generations

# Manually creating distributions
players = [G(), AC()]
for _ in range(n-len(players)):
    players.append(T4T())

birth(players, m, p, k)
plot_globals()
```

    Gen 1: T4T: 98%  G: 1%  AC: 1%  AD: 0%  
           T4T: 145530  G: 1485  AC: 1485  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 2: T4T: 98%  G: 1%  AC: 1%  AD: 0%  
           T4T: 145530  G: 1485  AC: 1485  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: 1485.0  AD: nan  
    Gen 3: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 4: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 5: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 6: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 7: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 8: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 9: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
           T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
           T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 10: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 11: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 12: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 13: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 14: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 15: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 16: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 17: T4T: 99%  G: 1%  AC: 0%  AD: 0%  
            T4T: 147015  G: 1485  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 18: T4T: 98%  G: 2%  AC: 0%  AD: 0%  
            T4T: 145530  G: 2970  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 19: T4T: 97%  G: 3%  AC: 0%  AD: 0%  
            T4T: 144045  G: 4455  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 20: T4T: 97%  G: 3%  AC: 0%  AD: 0%  
            T4T: 144045  G: 4455  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  



![png](img/output_15_1.png)



![png](img/output_15_2.png)



![png](img/output_15_3.png)


### Second Distribution: One Grudger player, one Always Cooperate player, and n-2 Always Defect players


```python
# Running the program with specified parameters
n = 100
m = 5 # Rounds between players
p = 5 # Pruning between generations
k = 20 # Generations

# Manually creating distributions
players = [G(), AC()]
for _ in range(n-len(players)):
    players.append(AD())

birth(players, m, p, k)
plot_globals()
```

    Gen 1: T4T: 0%  G: 1%  AC: 1%  AD: 98%  
           T4T: 0  G: 407  AC: 15  AD: 50862  Total:  51284
           T4T: nan  G: 407.0  AC: 15.0  AD: 519.0  
    Gen 2: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 3: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 4: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 5: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 6: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 7: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 8: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 9: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
           T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
           T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 10: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 11: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 12: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 13: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 14: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 15: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 16: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 17: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 18: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 19: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  
    Gen 20: T4T: 0%  G: 0%  AC: 0%  AD: 100%  
            T4T: 0  G: 0  AC: 0  AD: 49500  Total:  49500
            T4T: nan  G: nan  AC: nan  AD: 495.0  



![png](img/output_17_1.png)



![png](img/output_17_2.png)



![png](img/output_17_3.png)


### Third Distribution: Mostly AD, with some G, and very little AC and tiny amount of T4T


```python
# Running the program with specified parameters
n = 100
m = 5 # Rounds between players
p = 5 # Pruning between generations
k = 20 # Generations

# Manually creating distributions
players = []
for _ in range(8):
    players.append(AC())
for _ in range(2):
    players.append(T4T())
for _ in range(70):
    players.append(AD())
for _ in range(20):
    players.append(G())

birth(players, m, p, k)
plot_globals()
```

    Gen 1: T4T: 2%  G: 20%  AC: 8%  AD: 70%  
           T4T: 1430  G: 14300  AC: 3480  AD: 52010  Total:  71220
           T4T: 715.0  G: 715.0  AC: 435.0  AD: 743.0  
    Gen 2: T4T: 2%  G: 20%  AC: 3%  AD: 75%  
           T4T: 1320  G: 13200  AC: 1080  AD: 48225  Total:  63825
           T4T: 660.0  G: 660.0  AC: 360.0  AD: 643.0  
    Gen 3: T4T: 2%  G: 25%  AC: 0%  AD: 73%  
           T4T: 1364  G: 17050  AC: 0  AD: 44019  Total:  62433
           T4T: 682.0  G: 682.0  AC: nan  AD: 603.0  
    Gen 4: T4T: 2%  G: 30%  AC: 0%  AD: 68%  
           T4T: 1474  G: 22110  AC: 0  AD: 42364  Total:  65948
           T4T: 737.0  G: 737.0  AC: nan  AD: 623.0  
    Gen 5: T4T: 2%  G: 35%  AC: 0%  AD: 63%  
           T4T: 1584  G: 27720  AC: 0  AD: 40509  Total:  69813
           T4T: 792.0  G: 792.0  AC: nan  AD: 643.0  
    Gen 6: T4T: 3%  G: 39%  AC: 0%  AD: 58%  
           T4T: 2541  G: 33033  AC: 0  AD: 38454  Total:  74028
           T4T: 847.0  G: 847.0  AC: nan  AD: 663.0  
    Gen 7: T4T: 4%  G: 43%  AC: 0%  AD: 53%  
           T4T: 3608  G: 38786  AC: 0  AD: 36199  Total:  78593
           T4T: 902.0  G: 902.0  AC: nan  AD: 683.0  
    Gen 8: T4T: 4%  G: 48%  AC: 0%  AD: 48%  
           T4T: 3828  G: 45936  AC: 0  AD: 33744  Total:  83508
           T4T: 957.0  G: 957.0  AC: nan  AD: 703.0  
    Gen 9: T4T: 5%  G: 52%  AC: 0%  AD: 43%  
           T4T: 5060  G: 52624  AC: 0  AD: 31089  Total:  88773
           T4T: 1012.0  G: 1012.0  AC: nan  AD: 723.0  
    Gen 10: T4T: 6%  G: 56%  AC: 0%  AD: 38%  
            T4T: 6402  G: 59752  AC: 0  AD: 28234  Total:  94388
            T4T: 1067.0  G: 1067.0  AC: nan  AD: 743.0  
    Gen 11: T4T: 6%  G: 61%  AC: 0%  AD: 33%  
            T4T: 6732  G: 68442  AC: 0  AD: 25179  Total:  100353
            T4T: 1122.0  G: 1122.0  AC: nan  AD: 763.0  
    Gen 12: T4T: 8%  G: 64%  AC: 0%  AD: 28%  
            T4T: 9416  G: 75328  AC: 0  AD: 21924  Total:  106668
            T4T: 1177.0  G: 1177.0  AC: nan  AD: 783.0  
    Gen 13: T4T: 10%  G: 67%  AC: 0%  AD: 23%  
            T4T: 12320  G: 82544  AC: 0  AD: 18469  Total:  113333
            T4T: 1232.0  G: 1232.0  AC: nan  AD: 803.0  
    Gen 14: T4T: 11%  G: 71%  AC: 0%  AD: 18%  
            T4T: 14157  G: 91377  AC: 0  AD: 14814  Total:  120348
            T4T: 1287.0  G: 1287.0  AC: nan  AD: 823.0  
    Gen 15: T4T: 11%  G: 76%  AC: 0%  AD: 13%  
            T4T: 14762  G: 101992  AC: 0  AD: 10959  Total:  127713
            T4T: 1342.0  G: 1342.0  AC: nan  AD: 843.0  
    Gen 16: T4T: 11%  G: 81%  AC: 0%  AD: 8%  
            T4T: 15367  G: 113157  AC: 0  AD: 6904  Total:  135428
            T4T: 1397.0  G: 1397.0  AC: nan  AD: 863.0  
    Gen 17: T4T: 11%  G: 86%  AC: 0%  AD: 3%  
            T4T: 15972  G: 124872  AC: 0  AD: 2649  Total:  143493
            T4T: 1452.0  G: 1452.0  AC: nan  AD: 883.0  
    Gen 18: T4T: 11%  G: 89%  AC: 0%  AD: 0%  
            T4T: 16335  G: 132165  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 19: T4T: 11%  G: 89%  AC: 0%  AD: 0%  
            T4T: 16335  G: 132165  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  
    Gen 20: T4T: 12%  G: 88%  AC: 0%  AD: 0%  
            T4T: 17820  G: 130680  AC: 0  AD: 0  Total:  148500
            T4T: 1485.0  G: 1485.0  AC: nan  AD: nan  



![png](img/output_19_1.png)



![png](img/output_19_2.png)



![png](img/output_19_3.png)


## Question 3 Implementation
Back To Top [JUMP](#Question-3)

Note: Third Python cell has high step value and generating individual p-value outputs. First and second Python cell caculates values with step=1, but does not generate plots for each p-value.


```python
# Systematically varying value of p
# Running the program with specified parameters from Q1

n = 100 # Starting num of players
m = 5 # Rounds between players
p = 1 # Pruning between generations, will vary
k = 20 # Generations

labels = ['T4T', 'G', 'AC', 'AD']
p_vals = np.arange(1, 25, step=1)
last_gen_type_percentage = [[], [], [], []]
last_gen_type_total = [[], [], [], []]
last_gen_type_avg = [[], [], [], []]

for p in p_vals:
    players = gen_n_player_evenly(n)
    birth(players, m, p, k, printing=False) # Printing disabled
    for i in range(len(labels)):
        last_gen_type_percentage[i].append(stackplot_type_percentage[i][-1].copy())
        last_gen_type_total[i].append(stackplot_type_total[i][-1].copy())
        last_gen_type_avg[i].append(multiline_type_avg[i][-1].copy())
    print("Computed P-Val", p, end=". ")
```

    Computed P-Val 1. Computed P-Val 2. Computed P-Val 3. Computed P-Val 4. Computed P-Val 5. Computed P-Val 6. Computed P-Val 7. Computed P-Val 8. Computed P-Val 9. Computed P-Val 10. Computed P-Val 11. Computed P-Val 12. Computed P-Val 13. Computed P-Val 14. Computed P-Val 15. Computed P-Val 16. Computed P-Val 17. Computed P-Val 18. Computed P-Val 19. Computed P-Val 20. Computed P-Val 21. Computed P-Val 22. Computed P-Val 23. Computed P-Val 24. 


```python
_, ax = plt.subplots()
ax.stackplot(p_vals, last_gen_type_percentage, labels=labels, step='mid')

ax.legend(loc='upper right')
ax.set_xlabel("P-Val")
ax.set_ylabel('Percentage')
ax.set_title('Population by Type')
ax.set_xticks(p_vals)
plt.show()

_, ax = plt.subplots()
ax.stackplot(p_vals, last_gen_type_total, labels=labels, step='mid')
ax.legend(loc='upper right')
ax.set_xlabel("P-Val")
ax.set_ylabel('Payoff')
ax.set_title('Payoff by Type and Total Payoff')
ax.set_xticks(p_vals)
plt.plot()
plt.show()

_, ax = plt.subplots()
ax.set_title("Average Payoff by Type")
ax.set_xlabel("P-Val")
ax.set_ylabel("Average Payoff")
ax.set_xticks(p_vals)
line_type = ['-', '--', ':', '-.']
for i in range(len(multiline_type_avg)):
    plt.plot(p_vals, last_gen_type_avg[i], line_type[i], label=labels[i], marker='o')
ax.legend(loc="upper right")
plt.show()
```


![png](img/output_23_0.png)



![png](img/output_23_1.png)



![png](img/output_23_2.png)



```python
# Systematically varying value of p and plot output for each p-val

p_vals = np.arange(1, 25, step=5)

for p in p_vals:
    print("Computing P-Val", p, "...")
    players = gen_n_player_evenly(n)
    birth(players, m, p, k, printing=False) # Printing disabled
    plot_globals()
```

    Computing P-Val 1 ...



![png](img/output_24_1.png)



![png](img/output_24_2.png)



![png](img/output_24_3.png)


    Computing P-Val 6 ...



![png](img/output_24_5.png)



![png](img/output_24_6.png)



![png](img/output_24_7.png)


    Computing P-Val 11 ...



![png](img/output_24_9.png)



![png](img/output_24_10.png)



![png](img/output_24_11.png)


    Computing P-Val 16 ...



![png](img/output_24_13.png)



![png](img/output_24_14.png)



![png](img/output_24_15.png)


    Computing P-Val 21 ...



![png](img/output_24_17.png)



![png](img/output_24_18.png)



![png](img/output_24_19.png)

