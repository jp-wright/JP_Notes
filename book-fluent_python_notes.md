# "Fluent Python" by Luciano Romalho

```python
import collections Card = collections.namedtuple('Card', ['rank', 'suit'])
class FrenchDeck:     
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')     
    suits = 'spades diamonds clubs hearts'.split()     
    def __init__(self):         
    self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]     

    def __len__(self):         
        return len(self._cards)     

    def __getitem__(self, position):         
        return self._cards[position]
```

The first thing to note is the use of `collections.namedtuple` to construct a simple class to represent individual cards. Since Python 2.6, `namedtuple` can be used to build classes of objects that are just bundles of attributes with no custom methods, like a database record. In the example, we use it to provide a nice representation for the cards in the deck, as shown in the console session:

```python
>>> beer_card = Card('7', 'diamonds')
>>> beer_card
Card(rank='7', suit='diamonds')
```
But the point of this example is the `FrenchDeck` class. It’s short, but it packs a punch. First, like any standard Python collection, a deck responds to the `len()` function by returning the number of cards in it:

```python
>>> deck = FrenchDeck()
>>> len(deck)
52
```

Reading specific cards from the deck—say, the first or the last—should be as easy as `deck[0]` or `deck[-1]`, and this is what the `__getitem__` method provides:
```python
>>> deck[0]
Card(rank='2', suit='spades')
>>> deck[-1]
Card(rank='A', suit='hearts')
```
