#!/usr/bin/env python

from unicurses import *
from enum import Enum
from random import randint
from copy import deepcopy

# KLASY POMOCNICZE

# Ekrany w grze
class Screen(Enum):
    MENU = 0 # Menu główne
    DIFF_SELECT = 11 # Wybór poziomu trudności
    EASY = 12 # Pomocnicza. Pozmiom łatwy
    HARD = 13 # Pomocnicza. Poziom trudny.
    GAME = 1 # Właściwa gra
    PAUSE = 2 # Ekran pauzy
    WIN = 3 # Ekran zwycięstwa
    EXIT = 4 # Pomocnicza. Wyjście z gry

# Wybór w menu dialogowym (MENU, PAUSE, DIFF_SELECT)
class Choice:
    def __init__(self, label, cur_screen, description):
        self.label = label # Oznaczenie wyboru
        self.cur_screen = cur_screen # Do którego ekranu prowadzi wybór?
        self.description = description # Opis wyboru

# Wartość karty
class Value(Enum):
    ACE = 1
    TWO = 2
    THREE = 3
    FOUR = 4
    FIVE = 5
    SIX = 6
    SEVEN = 7
    EIGHT = 8
    NINE = 9
    TEN = 10
    JACK = 11 # Walet
    QUEEN = 12 # Dama
    KING = 13 # Król

# Kolor karty
class Suit(Enum):
    SPADES = 0 # Pik
    HEARTS = 1 # Kier
    CLUBS = 2 # Trefl
    DIAMONDS = 3 # Karo

# Karta

class Card:
    def __init__(self, value=0, suit=0, flip = True):
        self.value = value # Wartość
        self.suit = suit # Kolor
        self.flip = True # Czy jest odwrócona awersem do góry?

    def str_val(self): # Wyświetl wartość jako ciąg znaków
        match Value(self.value):
            case Value.ACE: return "A"
            case Value.JACK: return "J"
            case Value.QUEEN: return "Q"
            case Value.KING: return "K"
            case _: return str(self.value)

    def str_suit(self): # Wyświetl kolor jako ciąg znaków
        match Suit(self.suit):
            case Suit.SPADES: return "♠"
            case Suit.HEARTS: return "♥"
            case Suit.CLUBS: return "♣"
            case Suit.DIAMONDS: return "♦"

# ZMIENNE STAŁE

# W zależności od platformy kody klawiszowe mogą być różne - z tego powodu są zawarte w krotkach

U = (85, 117)
BACKSPACE  = (8, KEY_BACKSPACE)
ENTER = (10, KEY_ENTER)
ESC = 27

# Identyfikatory par kolorów

HIGHLIGHT = 4
BLACK_CARD = 1
RED_CARD = 3

# FUNKCJE POMOCNICZE

# tasowanie listy

def shuffle(list_in):
    old_list = list_in
    new_list = []
    for _ in range(len(old_list)):
        new_list.append(old_list.pop(randint(0,len(old_list)-1)))
    return new_list

# czy karty są różnych kolorów?

def opposite_colors(card1,card2):
    return card1.suit % 2 != card2.suit % 2

def check_color(card1,card2,same=True):
    return (card1.suit % 2 == card2.suit % 2) == same

# czy wartość jednej karty jest o 1 wyższa niż tej drugiej?

def plus1_value(hi_card,lo_card):
    return hi_card.value == lo_card.value+1

# OBIEKT GRY

class Game: 

    # Inicjalizacja programu
    def __init__(self):

        # Czy program działa?

        self.running = True

        # Inicjalizacja ekranów wyboru
        
        self.choices = []

        self.choice = 0
        self.cur_screen = Screen.MENU

        # Inicjalizacja pełnej talii kart

        self.deck_init = []

        for s in range(4):
            for v in range(13):
                self.deck_init.append(Card(v+1,s,True))

        self.state = {
            "hard": False, # poziom trudności
            "deck":shuffle(self.deck_init[:]), # stos rezerwowy
            "deck_shift":0, # przesunięcie kart w stosie rezerwowym
            "cards_on_deck":0, # karty w stosie rezerwowym
            "board":[[],[],[],[],[],[],[]], # kolumny gry
            "discard":[[],[],[],[]], # stosy końcowe
            "mp":[0,1], # pozycja wskaźnika
            "pickupp":[-1,-1], # pozycja zaznaczonej karty
            "picking":False, # czy jakaś karta jest teraz zaznaczana?
            "move":1 # numer ruchu
        }

        self.notif = ""

        self.last_moves = []

    # Konfiguracja poszczególnych ekranów (wyjaśnienia na linijce 10)
    
    def switch_screen(self, screen_in):

        self.cur_screen = screen_in
        
        self.choice = 0
        if self.cur_screen == Screen.MENU:

            self.choices = [
                Choice("Nowa Gra", Screen.DIFF_SELECT, "Rozpocznij nową grę."),
                Choice("Paszoł Won", Screen.EXIT, "Wyjdź z programu.")
            ]

        elif self.cur_screen == Screen.PAUSE:

            self.choices = [
                Choice("Kontynuuj", Screen.GAME, "Wznów grę."),
                Choice("Nowa Gra", Screen.DIFF_SELECT, "Zakończ obecną grę i rozpocznij nową.\nUWAGA: Stracisz wszystkie postępy w obecnej grze!"),
                Choice("Do Menu", Screen.MENU, "Zakończ obecną grę i wróć do menu.\nUWAGA: Stracisz wszystkie postępy w obecnej grze!")
            ]

        elif self.cur_screen == Screen.DIFF_SELECT:

            self.choices = [
                Choice("Łatwy", Screen.EASY, "Można dobierać po 1 karcie."),
                Choice("Trudny", Screen.HARD, "Dobiera się 3 karty, ale użyć można tylko wierzchniej."),
                Choice("Wróć", Screen.MENU, "Wróć do menu.")
            ]

        elif self.cur_screen == Screen.EASY:

            self.state["hard"] = False
            self.new_game()
            self.switch_screen(Screen.GAME)

        elif self.cur_screen == Screen.HARD:

            self.state["hard"] = True
            self.new_game()
            self.switch_screen(Screen.GAME)

        elif self.cur_screen == Screen.WIN:
            clear()
            addstr("WYGRYWASZ!\nPoziom trudności: "+("Trudny" if self.state["hard"] else "Łatwy")+"\nLiczba ruchów: "+str(self.state["move"])+"\nNaciśnij dowolny klawisz, aby przejść dalej.")
            getch()
            self.switch_screen(Screen.MENU)

        elif self.cur_screen == Screen.EXIT:

            self.running = False

    # Wyświetlanie karty
    def display_card(self, card, off_x = 0, off_y = 0):

        if card.flip:
            
            if card.suit % 2 == 0: attron(COLOR_PAIR(BLACK_CARD))
            else: attron(COLOR_PAIR(RED_CARD))

            mvaddstr(off_y,off_x,card.str_suit())
            addstr(card.str_val() if card.value == 10 else " "+card.str_val())

            attroff(COLOR_PAIR(BLACK_CARD))
            attroff(COLOR_PAIR(RED_CARD))

        else:
            mvaddstr(off_y,off_x,"III")

    # Podświetlanie karty
    def highlight_card(self,x,y):
        if y > 0:        

            if y <= len(self.state["board"][x]):
                
                highlighted_card = self.state["board"][x][y-1]

                if highlighted_card.flip:
                    mvchgat(y+2,x*5,3,A_STANDOUT,BLACK_CARD+(highlighted_card.suit % 2 != 0)*2,None)
                else:
                    mvchgat(y+2,x*5,3,A_BOLD,HIGHLIGHT,None)

            else:
                mvchgat(y+2,x*5,3,A_BOLD,HIGHLIGHT,None)
        
        else:

            if x < 3:
                if x < self.state["cards_on_deck"]: mvchgat(y+1,x*5,3,A_STANDOUT,BLACK_CARD+(self.state["deck"][self.state["deck_shift"]+x].suit % 2 != 0)*2,None)
                else: mvchgat(y+1,x*5,3,A_BOLD,HIGHLIGHT,None)
            else:
                if len(self.state["discard"][x-3]) > 0: mvchgat(y+1,x*5,3,A_STANDOUT,BLACK_CARD+(self.state["discard"][x-3][-1].suit % 2 != 0)*2,None)
                else: mvchgat(y+1,x*5,3,A_BOLD,HIGHLIGHT,None)

    # Ile kart jest widocznych na stosie rezerwowym?
    def cards_on_deck(self):
        return self.state["cards_on_deck"] if self.state["cards_on_deck"] < len(self.state["deck"]) else len(self.state["deck"])

    # Rozpoczęcie nowej gry
    def new_game(self):

        self.last_moves = []

        hard = self.state["hard"]

        self.state = {
            "hard": hard,
            "deck":shuffle(self.deck_init[:]),
            "deck_shift":0,
            "cards_on_deck":0,
            "board":[[],[],[],[],[],[],[]],
            "discard":[[],[],[],[]],
            "mp":[0,1],
            "pickupp":[-1,-1],
            "picking":False,
            "move":1
        }
        
        # Rozłożenie kart ze stosu rezerwowego do kolumn gry
        for r in range(len(self.state["board"])):
            for c in range(r+1):
                card = self.state["deck"].pop(randint(0,len(self.state["deck"])-1))
                card.flip = c == r
                self.state["board"][r].append(card)

        # Wszystkie karty w stosie rezerwowym muszą być awersem do góry
        for c in self.state["deck"]:
            c.flip = True

    # Wybierz kartę na określonej pozycji
    def get_card(self,x,y):
        if y>0:
            if y <= len(self.state["board"][x]): return self.state["board"][x][y-1]
        else:
            if x < 3:
                if len(self.state["deck"]) > 0 and x < self.cards_on_deck(): return self.state["deck"][x+self.state["deck_shift"]]
            else:
                if len(self.state["discard"][x-3]) > 0: return self.state["discard"][x-3][-1]
                
        return Card(0,0,False)

    def get_deck_id(self,x,y):
        if y > 0: return 2
        elif x > 2: return 1
        else: return 0

    # Uruchomienie programu
    def run(self):

        stdscr = initscr()
        noecho()
        curs_set(False)
        keypad(stdscr,True)
        LINES, COLS = getmaxyx(stdscr)

        if not has_colors(): # Czy dany terminal wspiera kolor?
            endwin()
            print("Uwaga - terminal nie wspiera koloru!")
            exit(1)

        # Inicjalizacja koloru

        start_color()

        init_pair(BLACK_CARD, COLOR_BLACK, COLOR_WHITE)
        init_pair(RED_CARD, COLOR_RED, COLOR_WHITE)

        init_pair(HIGHLIGHT, COLOR_BLUE, COLOR_WHITE)

        # Ekran początkowy

        addstr("PASJANS\nANTONI KOWALSKI\nGIGATHON, III EDYCJA\nZAŻÓŁĆ GĘŚLĄ JAŹŃ\nNaciśnij dowolny klawisz, aby przejść dalej.")
        inp = getch()

        self.switch_screen(Screen.MENU)

        # MAIN LOOP

        while self.running:

            # Oczyszczenie ekranu
            clear()
            
            if self.cur_screen == Screen.GAME:
                
                # Jeśli w stosie rezerwowym zostały jakieś karty, daj o tym znać
                if len(self.state["deck"]) > 0 and self.state["deck_shift"] < len(self.state["deck"])-self.cards_on_deck(): addstr("III")
                
                # Wyświetlanie liczby ruchów
                mvaddstr(0,27-len(str(self.state["move"])),str(self.state["move"]) + ". ruch ")
                
                # Mechanizm powiadomienia
                addstr(self.notif)

                self.notif = ""

                # Wyświetlanie stosu rezerwowego
                for c in range(self.cards_on_deck()):
                    self.display_card(self.state["deck"][c+self.state["deck_shift"]],c*5,1)

                # Wyświetlanie stosów końcowych
                for n in range(4):
                    if len(self.state["discard"][n]) > 0: self.display_card(self.state["discard"][n][-1],(n+3)*5,1)
                    else: mvaddstr(1, (n+3)*5, "XXX")

                # Wyświetlanie kolumn gry
                for r in range(len(self.state["board"])):
                    for c in range(len(self.state["board"][r])):
                        self.state["board"][r][-1].flip = True
                        self.display_card(self.state["board"][r][c],r*5,c+3)

                # Podświetlanie zaznaczonej karty
                self.highlight_card(self.state["mp"][0],self.state["mp"][1])
                if self.state["picking"]: self.highlight_card(self.state["pickupp"][0],self.state["pickupp"][1])

                # Wejście
                inp = getch()
                
                # Pauza
                if inp == ESC:
                    self.switch_screen(Screen.PAUSE)

                # Poruszanie wskaźnikiem
                elif inp == KEY_UP and self.state["mp"][1] > 0:
                    self.state["mp"][1] -= 1
                elif inp == KEY_DOWN and self.state["mp"][1] < 21: 
                    self.state["mp"][1] += 1
                elif inp == KEY_LEFT and self.state["mp"][0] > 0: 
                    self.state["mp"][0] -= 1
                elif inp == KEY_RIGHT and self.state["mp"][0] < 6: 
                    self.state["mp"][0] += 1
                
                # Cofanie ruchów
                elif inp in U:
                    if len(self.last_moves)>0: 
                        self.state = self.last_moves.pop(-1)
                    pass

                # Dobór karty ze stosu rezerwowego
                elif inp in BACKSPACE: 

                    hard = self.state["hard"]                    
                    
                    if self.state["deck_shift"]+self.cards_on_deck() >= len(self.state["deck"]):
                        self.state["deck_shift"] = 0
                        self.state["cards_on_deck"] = 0
                        if not self.get_deck_id(self.state["mp"][0],self.state["mp"][1]): self.state["mp"][1] += 1
                        self.state["deck"] = shuffle(self.state["deck"])
                    else:
                        if self.state["cards_on_deck"] < 3: self.state["cards_on_deck"] += 1+2*hard
                        else: self.state["deck_shift"] += 1+2*hard

                        if self.state["deck_shift"] + self.cards_on_deck() >= len(self.state["deck"]): self.state["deck_shift"] = len(self.state["deck"])-self.cards_on_deck()
                
                # Zaznaczanie i przenoszenie kart
                elif inp in ENTER:
                    if self.state["picking"]:

                        # Przechowanie poprzedniego stanu
                        last_state = deepcopy(self.state)

                        # żeby upewnić się, że żadna karta nie będzie zaznaczona w poprzednim stanie
                        last_state["picking"] = False                                
                        last_state["pickupp"][0] = -1
                        last_state["pickupp"][1] = -1

                        # Miejsce, na które ma trafić zaznaczona karta
                        card = self.get_card(self.state["mp"][0],self.state["mp"][1])
                        
                        # Czy doszło do przeniesienia karty?
                        moved = False

                        # Czy pole jest puste? Czy przenoszona karta to król?
                        if self.get_deck_id(self.state["mp"][0],self.state["mp"][1]) == 2 and card.value == 0 and self.get_card(self.state["pickupp"][0],self.state["pickupp"][1]).str_val() == "K":
                            if len(self.state["board"][self.state["mp"][0]]) == 0: # Czy nie ma kart w rzędzie?
                                match self.get_deck_id(self.state["pickupp"][0],self.state["pickupp"][1]):
                                    case 0:
                                        card = self.state["deck"].pop(self.state["deck_shift"]+self.state["pickupp"][0])
                                        self.state["board"][self.state["mp"][0]].append(card)
                                        if self.state["deck_shift"] > 0: self.state["deck_shift"] -= 1
                                        elif self.cards_on_deck() > 0: self.state["cards_on_deck"] -= 1
                                        moved = True
                                    case 1:
                                        card = self.state["discard"][self.state["pickupp"][0]-3].pop(-1)
                                        self.state["board"][self.state["mp"][0]].append(card)
                                        moved = True
                                    case 2:
                                        for c in range(len(self.state["board"][self.state["pickupp"][0]])-(self.state["pickupp"][1]-1)):
                                            card = self.state["board"][self.state["pickupp"][0]].pop(self.state["pickupp"][1]-1)
                                            self.state["board"][self.state["mp"][0]].append(card)
                                        moved = True
                        
                        # Czy karta jest odwrócona awersem do góry?
                        elif card.flip:

                            # Czy zaznaczona karta i miejsce jej przeniesienia są takie same? W przeciwnym razie karta pozostanie w miejscu.
                            if not (self.state["mp"][0] == self.state["pickupp"][0] and self.state["mp"][1] == self.state["pickupp"][1]) and self.get_deck_id(self.state["mp"][0],self.state["mp"][1]):

                                if self.get_deck_id(self.state["mp"][0],self.state["mp"][1]) == 2:
                                    if self.state["mp"][1] <= len(self.state["board"][self.state["mp"][0]]):
                                        match self.get_deck_id(self.state["pickupp"][0],self.state["pickupp"][1]):
                                            case 0:
                                                if opposite_colors(self.state["deck"][self.state["deck_shift"]+self.state["pickupp"][0]],self.state["board"][self.state["mp"][0]][-1]) and plus1_value(self.state["board"][self.state["mp"][0]][-1],self.state["deck"][self.state["deck_shift"]+self.state["pickupp"][0]]):
                                                    card = self.state["deck"].pop(self.state["deck_shift"]+self.state["pickupp"][0])
                                                    self.state["board"][self.state["mp"][0]].append(card)
                                                    if self.state["deck_shift"] > 0: self.state["deck_shift"] -= 1
                                                    elif self.cards_on_deck() > 0: self.state["cards_on_deck"] -= 1
                                                    moved = True
                                            case 1:
                                                if opposite_colors(self.state["discard"][self.state["pickupp"][0]-3][-1],self.state["board"][self.state["mp"][0]][-1]) and plus1_value(self.state["board"][self.state["mp"][0]][-1],self.state["discard"][self.state["pickupp"][0]-3][-1]) or not len(self.state["discard"][self.state["pickupp"][0]-3]):
                                                    card = self.state["discard"][self.state["pickupp"][0]-3].pop(-1)
                                                    self.state["board"][self.state["mp"][0]].append(card)
                                                    moved = True
                                            case 2:
                                                if opposite_colors(self.state["board"][self.state["pickupp"][0]][self.state["pickupp"][1]-1], self.state["board"][self.state["mp"][0]][self.state["mp"][1]-1]) and plus1_value(self.state["board"][self.state["mp"][0]][self.state["mp"][1]-1],self.state["board"][self.state["pickupp"][0]][self.state["pickupp"][1]-1]):
                                                    for c in range(len(self.state["board"][self.state["pickupp"][0]])-(self.state["pickupp"][1]-1)):
                                                        card = self.state["board"][self.state["pickupp"][0]].pop(self.state["pickupp"][1]-1)
                                                        self.state["board"][self.state["mp"][0]].append(card)
                                                    moved = True
                                    
                                elif self.get_deck_id(self.state["mp"][0],self.state["mp"][1]) == 1:

                                    if self.get_deck_id(self.state["pickupp"][0],self.state["pickupp"][1]) == 2:

                                        if len(self.state["discard"][self.state["mp"][0]-3]):
                                            moved = self.state["discard"][self.state["mp"][0]-3][-1].suit == self.state["board"][self.state["pickupp"][0]][-1].suit and plus1_value(self.state["board"][self.state["pickupp"][0]][-1], self.state["discard"][self.state["mp"][0]-3][-1])
                                        else:
                                            moved = self.state["board"][self.state["pickupp"][0]][-1].value == 1
                                        
                                        if moved:
                                            card = self.state["board"][self.state["pickupp"][0]].pop(-1)
                                            self.state["discard"][self.state["mp"][0]-3].append(card)

                                    elif self.get_deck_id(self.state["pickupp"][0],self.state["pickupp"][1]) == 0:
                                        if len(self.state["discard"][self.state["mp"][0]-3]):
                                            moved = self.state["discard"][self.state["mp"][0]-3][-1].suit == self.state["deck"][self.state["deck_shift"]+self.state["pickupp"][0]].suit and plus1_value(self.state["deck"][self.state["deck_shift"]+self.state["pickupp"][0]],self.state["discard"][self.state["mp"][0]-3][-1])                 
                                        else:
                                            moved = self.state["deck"][self.state["deck_shift"]+self.state["pickupp"][0]].value == 1

                                        if moved:
                                            card = self.state["deck"].pop(self.state["deck_shift"]+self.state["pickupp"][0])
                                            self.state["discard"][self.state["mp"][0]-3].append(card)
                                            if self.state["deck_shift"] > 0: self.state["deck_shift"] -= 1

                        # Reset zaznaczania karty
                        self.state["picking"] = False                                
                        self.state["pickupp"][0] = -1
                        self.state["pickupp"][1] = -1

                        # Zapisywanie ostatniego ruchu - przeniesienie karty liczy się jako ruch
                        if moved:

                            self.last_moves.append(last_state)
                            self.state["move"] += 1
                            
                        # Zapisanych ostatnio ruchów nie może być więcej niż 3
                        if len(self.last_moves) > 3:
                            self.last_moves.pop(0)

                        # Sprawdzanie warunku wygranej

                        win = True

                        for d in self.state["discard"]:
                            if len(d) > 0:
                                if d[-1].value == 13:
                                    continue
                            win = False
                            break

                        if win:
                            self.state["move"] -= 1
                            self.switch_screen(Screen.WIN)
                        
                    else:

                        # Zaznaczanie karty

                        card = self.get_card(self.state["mp"][0],self.state["mp"][1])
                        if card.value != 0 and card.flip:
                            self.state["picking"] = True
                            
                            self.state["pickupp"][0] = self.state["mp"][0]
                            self.state["pickupp"][1] = self.state["mp"][1]

                # Jeśli poziom trudności jest Trudny, gracz może wybrać tylko kartę z wierzchu

                if self.state["hard"]:
                    if not self.get_deck_id(self.state["mp"][0],self.state["mp"][1]):
                        self.state["mp"][0] = self.cards_on_deck()-1

            else:

                # Wyświetlanie wyborów
                for c in range(len(self.choices)):
                    if c == self.choice: attron(COLOR_PAIR(HIGHLIGHT))
                    addstr(self.choices[c].label+"\n")
                    attroff(COLOR_PAIR(HIGHLIGHT))

                # Wyświetlanie opisu
                addstr("\n"+self.choices[self.choice].description)
            
                # Wejście
                inp = getch()
                if inp == ESC and self.cur_screen == Screen.MENU: self.running = False
                elif inp == KEY_UP and self.choice > 0: self.choice -= 1
                elif inp == KEY_DOWN and self.choice < len(self.choices)-1: self.choice += 1
                elif inp in ENTER:
                    self.switch_screen(self.choices[self.choice].cur_screen)

        endwin()

if __name__ == "__main__":
    game = Game()
    game.run()
    del game
