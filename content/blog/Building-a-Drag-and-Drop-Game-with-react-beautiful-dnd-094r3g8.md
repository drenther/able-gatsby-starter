---
title: Building a Drag-and-Drop Game with react-beautiful-dnd
date: 2020-06-02T17:43:55.719985Z
description: undefined
---

Drag and Drop user interfaces are very common in modern applications and yet it’s quite difficult to get them right. Using the [HTML Drag and Drop API ](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Drag_and_drop)can be a pain, and it’s known to have many [inconsistencies and gotchas](https://www.quirksmode.org/blog/archives/2009/09/the_html5_drag.html). Making a complex Drag and Drop UI in React is not an easy task either, it can get overwhelming pretty quickly. But, libraries like [react-dnd](https://github.com/react-dnd/react-dnd) and [react-beautiful-dnd](https://github.com/atlassian/react-beautiful-dnd) are there to help us. I am particularly fond of **react-beautiful-dnd** because of its simple API, beautiful \(as the name already suggests\) movement animations and baked-in accessibility features. Today, we will take a closer look at using **react-beautiful-dnd** by making a simple yet fun game.

Here's a [demo](https://codesandbox.io/s/github/drenther/bdnd_example) of the game we'll be creating and a quick video of how it works below.

![](https://assets.able.bio/media/u/ed3d2c21991e3bef5e069713af9fa6ca/gif.gif)

We will create this simple game, where we will have three List-like components one for Marvel heroes, one for DC heroes and the third one called the “Bench” for holding all the heroes that are yet to be placed in the correct List. To make it more interesting, the scoring will be done on not just the correct grouping of the heroes but also on sorting them alphabetically and finishing as quickly as possible.

## Getting Started

We will start by bootstrapping a React project using the amazing [create-react-app](https://github.com/facebook/create-react-app) \(v2\) or creating a [new React project on Codesandbox](https://codesandbox.io/s/new). We will install our dependencies which are **react**, **react-dom** and **react-beautiful-dnd**. Finally, we’ll also bring in spectre.css to help us make the app look decent with minimal effort.

Let’s look at the data that our game will be using:

_**data.js**_

```
/** custom/data.js **/

export const COMICS = {
  DC: 'dc',
  MARVEL: 'marvel',
};

export const HEROES = [
  {
    name: 'Superman',
    comics: 'dc',
  },
  {
    name: 'Batman',
    comics: 'dc',
  },
  {
    name: 'Flash',
    comics: 'dc',
  },
  {
    name: 'Aquaman',
    comics: 'dc',
  },
  {
    name: 'Wonder Woman',
    comics: 'dc',
  },
  {
    name: 'Green Lantern',
    comics: 'dc',
  },
  {
    name: 'Iron Man',
    comics: 'marvel',
  },
  {
    name: 'Spiderman',
    comics: 'marvel',
  },
  {
    name: 'Captain America',
    comics: 'marvel',
  },
  {
    name: 'Thor',
    comics: 'marvel',
  },
  {
    name: 'Hulk',
    comics: 'marvel',
  },
  {
    name: 'Black Widow',
    comics: 'marvel',
  },
];
```

So, we have two comics and twelve heroes \(six from each faction\). Now, let’s look at the `initialState` of our app. The initial state has three Array fields - each one holding the data for the List we discussed earlier. We start with all the heroes in the “Bench”. To make things randomized, we shuffle the entry order by using a simple [Knuth Shuffle algorithm](https://www.rosettacode.org/wiki/Knuth_shuffle). We also set the `gameState` to “ready” as in ready to start. The other two possible game states are “playing” and “done”.

_**initialState.js**_

```
/** App.js **/

import { HEROES, COMICS } from './custom/data';
import { shuffle, GAME_STATE } from './custom/utils';

const initialState = {
  // we initialize the state by populating the bench with a shuffled collection of heroes
  bench: shuffle(HEROES),
  [COMICS.DC]: [],
  [COMICS.MARVEL]: [],
  gameState: GAME_STATE.READY,
  timeLeft: 0,
};
```

_**shuffle.js**_

```
/** custom/utils.js **/

// the Knuth shuffle algorithm
export function shuffle(array) {
  let currentIndex = array.length;
  let temporaryValue;
  let randomIndex;

  // While there remain elements to shuffle...
  while (0 !== currentIndex) {
    // Pick a remaining element...
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;

    // And swap it with the current element.
    temporaryValue = array[currentIndex];
    array[currentIndex] = array[randomIndex];
    array[randomIndex] = temporaryValue;
  }

  return array;
}

// enums for representing the game state
export const GAME_STATE = {
  READY: 'ready',
  PLAYING: 'playing',
  DONE: 'done',
};
```

In the shuffle algorithm, we start from the end of the array and replace it with a random indexed item on the array, and we keep doing this until we reach the first element of the array.

## Drag and Drop

Now, let’s look at the structure of the main `App` component and specifically the render function first. Then, we will dive into the logic bits shortly after. The `<>` syntax is a convenient shorthand for [React.Fragments](https://reactjs.org/docs/react-api.html#reactfragment) which let’s you return multiple adjacent JSX elements without wrapping them up in a wrapper/container element. We are using five components here - `Header`, `Modal`, `DragDropContext`, `Dropzone` and `Footer`. The `Header` and `Modal` component help manage different game states, while the `Footer` component just renders some static text content. The `Modal` component is only rendered when the game is in a “ready” or “done” state.

_**App.js**_

```javascript
class App extends React.Component {
  render() {
    const { gameState, timeLeft, bench, ...groups } = this.state;
    const isDropDisabled = gameState === GAME_STATE.DONE;

    return (
      <>
        <Header gameState={gameState} timeLeft={timeLeft} endGame={this.endGame} />
        {this.state.gameState !== GAME_STATE.PLAYING && (
          <Modal
            startGame={this.startGame}
            resetGame={this.resetGame}
            timeLeft={timeLeft}
            gameState={gameState}
            groups={groups}
          />
        )}
        {(this.state.gameState === GAME_STATE.PLAYING ||
          this.state.gameState === GAME_STATE.DONE) && (
          <DragDropContext onDragEnd={this.onDragEnd}>
            <div className="container">
              <div className="columns">
                <Dropzone
                  id={COMICS.MARVEL}
                  heroes={this.state[COMICS.MARVEL]}
                  isDropDisabled={isDropDisabled}
                />
                <Dropzone id="bench" heroes={bench} isDropDisabled={isDropDisabled} />
                <Dropzone
                  id={COMICS.DC}
                  heroes={this.state[COMICS.DC]}
                  isDropDisabled={isDropDisabled}
                />
              </div>
            </div>
          </DragDropContext>
        )}
        <Footer />
      </>
    );
  }
}
```

The `DragDropContext` component is provided by the **react-beautiful-dnd** and is used to wrap any part of the React tree that needs to support the Drag and Drop functionality. It is usually advised to have only one of these components wrapping your entire React app and not have nested `DragDropContext` components. It is similar to the `Provider` component pattern, you might be familiar with when using Redux. In our case, this subtree is only rendered when the game state is either “playing” or “done”.

`Dropzone` components are the children of the `DragDropContext` component because that’s where we need our Drag and Drop feature. You can notice we have three `Dropzone` components, one for each List type with their respective id and heroes passed as props to them.

_**Dropzone.js**_

```javascript
/** components/Dropzone.js **/

import React from 'react';
import { Droppable, Draggable } from 'react-beautiful-dnd';

const Dropzone = ({ isDropDisabled, heroes, id }) => (
  <div className="column col-4">
    <div className="divider" data-content={id.toUpperCase()} />
    <Droppable droppableId={id} isDropDisabled={isDropDisabled}>
      {provided => {
        return (
          <div className="menu hero-list" {...provided.droppableProps} ref={provided.innerRef}>
            {heroes.map(({ name }, index) => (
              <Hero key={name} name={name} index={index} />
            ))}
            {provided.placeholder}
          </div>
        );
      }}
    </Droppable>
  </div>
);

const Hero = ({ name, index }) => (
  <Draggable key={name} draggableId={name} index={index}>
    {provided => {
      return (
        <div
          className="menu-item tile tile-centered"
          ref={provided.innerRef}
          {...provided.draggableProps}
          {...provided.dragHandleProps}
        >
          <figure style={{ backgroundColor: 'transparent' }} className="avatar tile-icon">
            <img src={`./hero_icons/${name.toLowerCase().replace(' ', '-')}.svg`} alt={name} />
          </figure>
          <div className="tile-content">{name}</div>
        </div>
      );
    }}
  </Draggable>
);

export default Dropzone;
```

**react-beautiful-dnd** exports two other major components that leverage the [Render props pattern](https://able.bio/drenther/track-page-visibility-in-react-using-render-props--78o9yw5) - `Droppable` and `Draggable`. The `Droppable` component is used for enabling Drop functionality while the `Draggable` component is used to enable Dragging functionality. The `Draggable` component child can be dropped on a `Droppable` component child. Therefore, each `Dropzone` component renders a `Droppable` list-like component with `draggableId` set to the id props passed to it. The `draggableId` works as a unique identifier for that `Droppable` and can be used for various advanced usage like allowing conditional logic based drop capability and more. We also use a `isDropDisabled` prop to disallow accepting drop elements when the game has not started or has ended already. Finally, we map over all the heroes currently in the list and render a `Draggable` component for each. So, now we have three lists that render various Superheroes, that we can drag and drop onto other lists. But, the Drag and Drop is just dummy functionality, for now, the dragged Superheroes don’t persist in the Lists they were dropped onto. For that, we need to wire up the logic to update the state accordingly.

## State Management

To handle state changes related to Drag and Drop, we use the `onDragEnd` props on the `DragDropContext` component. When a `Draggable` is dropped on a `Droppable` component inside the tree under `DragDropContext`, the method passed to `onDragEnd` props is called. There are [other optional methods](https://github.com/atlassian/react-beautiful-dnd/blob/master/docs/guides/responders.md) too but this is the only required one. We can access the source and destination `Droppable` for the interaction and accordingly update our state. If the destination is "falsy, then we don’t update the state because the Drop was invalid, otherwise we pass the current app state, source and destination value to the move function exported from **utils.js**.

_**App.js**_

```javascript
/** App.js **/

import React from 'react';
import { DragDropContext } from 'react-beautiful-dnd';

import { HEROES, COMICS } from './custom/data';
import { shuffle, move, GAME_STATE } from './custom/utils';

import Modal from './components/Modal';
import Header from './components/Header';
import Dropzone from './components/Dropzone';
import Footer from './components/Footer';

const GAME_DURATION = 1000 * 30; // 30 seconds

const initialState = {
  // we initialize the state by populating the bench with a shuffled collection of heroes
  bench: shuffle(HEROES),
  [COMICS.DC]: [],
  [COMICS.MARVEL]: [],
  gameState: GAME_STATE.READY,
  timeLeft: 0,
};

class App extends React.Component {
  state = initialState;

  onDragEnd = ({ source, destination }) => {
    if (!destination) {
      return;
    }

    this.setState(state => {
      return move(state, source, destination);
    });
  };

  render() {
    const { gameState, timeLeft, bench, ...groups } = this.state;
    const isDropDisabled = gameState === GAME_STATE.DONE;

    return (
      <>
        <Header gameState={gameState} timeLeft={timeLeft} endGame={this.endGame} />
        {this.state.gameState !== GAME_STATE.PLAYING && (
          <Modal
            startGame={this.startGame}
            resetGame={this.resetGame}
            timeLeft={timeLeft}
            gameState={gameState}
            groups={groups}
          />
        )}
        {(this.state.gameState === GAME_STATE.PLAYING ||
          this.state.gameState === GAME_STATE.DONE) && (
          <DragDropContext onDragEnd={this.onDragEnd}>
            <div className="container">
              <div className="columns">
                <Dropzone
                  id={COMICS.MARVEL}
                  heroes={this.state[COMICS.MARVEL]}
                  isDropDisabled={isDropDisabled}
                />
                <Dropzone id="bench" heroes={bench} isDropDisabled={isDropDisabled} />
                <Dropzone
                  id={COMICS.DC}
                  heroes={this.state[COMICS.DC]}
                  isDropDisabled={isDropDisabled}
                />
              </div>
            </div>
          </DragDropContext>
        )}
        <Footer />
      </>
    );
  }
}

export default App;
```

_**custom/utils.js**_

```javascript
/** custom/utils.js **/

import { HEROES, COMICS } from './data';

// the Knuth shuffle algorithm
export function shuffle(array) {
  let currentIndex = array.length;
  let temporaryValue;
  let randomIndex;

  // While there remain elements to shuffle...
  while (0 !== currentIndex) {
    // Pick a remaining element...
    randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex -= 1;

    // And swap it with the current element.
    temporaryValue = array[currentIndex];
    array[currentIndex] = array[randomIndex];
    array[randomIndex] = temporaryValue;
  }

  return array;
}

// method to handle to the heroe cards movement
export const move = (state, source, destination) => {
  const srcListClone = [...state[source.droppableId]];
  const destListClone =
    source.droppableId === destination.droppableId
      ? srcListClone
      : [...state[destination.droppableId]];

  const [movedElement] = srcListClone.splice(source.index, 1);
  destListClone.splice(destination.index, 0, movedElement);

  return {
    [source.droppableId]: srcListClone,
    ...(source.droppableId === destination.droppableId
      ? {}
      : {
          [destination.droppableId]: destListClone,
        }),
  };
};

// enums for representing the game state
export const GAME_STATE = {
  READY: 'ready',
  PLAYING: 'playing',
  DONE: 'done',
};
```

In the move function, we make a copy of the source list’s state. We also make a copy of the destination list’s state if the source and the destination are different lists, otherwise, use the source list copy as the destination list copy. Using the `Array.splice` method we remove the Dragged element from its current index in the source list and then add it to the destination list at an index which will reflect the position where it was Dropped. We then return the updated lists to be used for updating the `App` component state. Now, our application has a functioning Drag and Drop feature i.e. the core mechanics of our game. We can move onto adding the final features like score calculation, timer, etc.

## Start, Loop and Reset

Next, let’s looks at the logic behind the _startGame_, _endGame_, _resetGame_ and _gameLoop_.

The startGame method updates the state with `gameState` field set to “playing” and the `currentDeadline` instance value to 30 seconds later. It also invokes the `gameLoop` method following the state update by passing it as the second callback to `setState`.

The `gameLoop` starts a setInterval that updates the timeLeft state value every second \(a crude game loop implementation\). When the time is up, it sets the `timeLeft` to zero and `gameState` to “done” and also clears the timer. We also add a `componentWillUnmount` lifecycle hook to cleanup any uncleared timers to avoid potential memory leaks.

The game also provides a button on the `Header` component to end the game early to score bonus points. The endGame method handles that case by setting the `gameState` to “done” and clearing the timer. In this case, we do not make the `timeLeft` value zero because we will use it to calculate the score later. Finally, we have a resetGame method which simply resets to the `intialState` values.

_**App.js**_

```javascript
import React from 'react';
import { DragDropContext } from 'react-beautiful-dnd';

import { HEROES, COMICS } from './custom/data';
import { shuffle, getTimeLeft, move, GAME_STATE } from './custom/utils';

import Modal from './components/Modal';
import Header from './components/Header';
import Dropzone from './components/Dropzone';
import Footer from './components/Footer';

const GAME_DURATION = 1000 * 30; // 30 seconds

const initialState = {
  // we initialize the state by populating the bench with a shuffled collection of heroes
  bench: shuffle(HEROES),
  [COMICS.DC]: [],
  [COMICS.MARVEL]: [],
  gameState: GAME_STATE.READY,
  timeLeft: 0,
};

class App extends React.Component {
  state = initialState;

  startGame = () => {
    this.currentDeadline = Date.now() + GAME_DURATION;

    this.setState(
      {
        gameState: GAME_STATE.PLAYING,
        timeLeft: getTimeLeft(this.currentDeadline),
      },
      this.gameLoop
    );
  };

  gameLoop = () => {
    this.timer = setInterval(() => {
      const timeLeft = getTimeLeft(this.currentDeadline);
      const isTimeout = timeLeft <= 0;
      if (isTimeout && this.timer) {
        clearInterval(this.timer);
      }

      this.setState({
        timeLeft: isTimeout ? 0 : timeLeft,
        ...(isTimeout ? { gameState: GAME_STATE.DONE } : {}),
      });
    }, 1000);
  };

  endGame = () => {
    if (this.timer) {
      clearInterval(this.timer);
    }

    this.setState({
      gameState: GAME_STATE.DONE,
    });
  };

  resetGame = () => {
    this.setState(initialState);
  };

  onDragEnd = ({ source, destination }) => {
    if (!destination) {
      return;
    }

    this.setState(state => {
      return move(state, source, destination);
    });
  };

  render() {
    const { gameState, timeLeft, bench, ...groups } = this.state;
    const isDropDisabled = gameState === GAME_STATE.DONE;

    return (
      <>
        <Header gameState={gameState} timeLeft={timeLeft} endGame={this.endGame} />
        {this.state.gameState !== GAME_STATE.PLAYING && (
          <Modal
            startGame={this.startGame}
            resetGame={this.resetGame}
            timeLeft={timeLeft}
            gameState={gameState}
            groups={groups}
          />
        )}
        {(this.state.gameState === GAME_STATE.PLAYING ||
          this.state.gameState === GAME_STATE.DONE) && (
          <DragDropContext onDragEnd={this.onDragEnd}>
            <div className="container">
              <div className="columns">
                <Dropzone
                  id={COMICS.MARVEL}
                  heroes={this.state[COMICS.MARVEL]}
                  isDropDisabled={isDropDisabled}
                />
                <Dropzone id="bench" heroes={bench} isDropDisabled={isDropDisabled} />
                <Dropzone
                  id={COMICS.DC}
                  heroes={this.state[COMICS.DC]}
                  isDropDisabled={isDropDisabled}
                />
              </div>
            </div>
          </DragDropContext>
        )}
        <Footer />
      </>
    );
  }

  componentWillUnmount() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  }
}

export default App;
```

Before, we can move onto the final feature of our game let’s look at the Components that still need to be discussed. The `Footer` element just renders a link to the icon set we are using for the Superheroes elements.

The `Header` element is passed the `gameState` and the amount of `timeLeft` to complete the current game round as props and it renders the number of seconds left until a game round ends when the game is in “playing” state. It also renders a button to end game early to score bonus points.

_**Footer.js**_

```javascript
import React from 'react';

const Footer = () => (
  <footer className="container text-right text-secondary">
    <a href="https://www.freepik.com/free-vector/superhero-icons-collection_800473.htm">
      Like the icons?
    </a>
  </footer>
);

export default Footer;
```

_**Header.js**_

```javascript
import React from 'react';

import { GAME_STATE, getSeconds } from '../custom/utils';

const Header = ({ timeLeft, gameState, endGame }) => (
  <header className="navbar">
    {gameState === GAME_STATE.PLAYING && (
      <>
        <section className="navbar-center text-error">{getSeconds(timeLeft)} Seconds Left</section>
        <section className="navbar-center">
          <button className="btn btn-default" onClick={endGame}>
            End Game
          </button>
        </section>
      </>
    )}
  </header>
);

export default Header;
```

_**Modal.js**_

```javascript
import React from 'react';

import { GAME_STATE, getTotalScore } from '../custom/utils';

const Modal = ({ gameState, groups, startGame, timeLeft, resetGame }) => (
  <div className="modal modal-sm active">
    <div className="modal-overlay" />
    <div className="modal-container">
      <div className="modal-header">
        <div className="modal-title h4">Line up the Heroes</div>
      </div>
      <div className="modal-body">
        <div className="content h6">
          {' '}
          {gameState === GAME_STATE.READY
            ? `Drag and Drop the heroes in the correct comics list, sort them alphabetically and quickly for better score...`
            : `You scored - ${getTotalScore(groups, timeLeft)}`}
        </div>
      </div>
      <div className="modal-footer">
        <button
          className="btn btn-primary"
          onClick={gameState === GAME_STATE.READY ? startGame : resetGame}
        >
          {gameState === GAME_STATE.READY ? 'Start new game' : 'Restart game'}
        </button>
      </div>
    </div>
  </div>
);

export default Modal;
```

The `Modal` component handles the other two game states - “ready” and “done”. When the game is in “ready” state, the `Modal` component renders a welcome text and a button to start the game. When the game is in “done” state, it renders the score achieved and a button to reset the game.

## Scoring

The score displayed on the `Modal` component at the end of a game round is calculated by the `getTotalScore` and `calculateScore` methods in the **utils.js**.

The `calculateScore` method takes either of the Marvel or DC grouped and sorted heroes lists and compares them against the appropriate ideal list order. If a hero is present in the correct list and the correct index, full points are awarded for that hero i.e. the total number of heroes \(12 in this case\). If the hero is present in the correct list but the sort order is incorrect, the difference between the correct and incorrect index is subtracted from the points to be awarded. If the hero is not present in the correct list, zero points are awarded. Finally, the total score for the two lists are added along with any time bonus \(1 point for each second left\) in the getTotalScore method and returned as the total score.

_**index.css**_

```css
header.navbar {
  height: 40px;
  margin: 5px 20px;
}

div.menu .menu-item.tile.tile-centered {
  margin-top: 5px;
}

.menu.hero-list {
  z-index: inherit;
  transform: none;
  min-height: 500px;
}

footer {
  position: fixed;
  bottom: 0;
  font-size: 14px;
}
```

_**index.js**_

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

import 'spectre.css';
import './index.css';

import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));
```

We wrap up by mounting our `App` component in **index.js** and importing our CSS to make the application look nice and clean.

## Source code

If you're interested in playing with the source code for this project you can find that here [here](https://github.com/drenther/bdnd_example).

## Conclusion

Drag and Drop implementations are still quite complicated but a crucial part of the modern web. Libraries like **react-beautiful-dnd** make it approachable and fun to develop. There is a lot more you can do with this package, and their documentation is all you need to dive deeper. If you make something great with it, they have an issue where you can share your applications and experiments for everyone to enjoy. If you find a use case where **react-beautiful-dnd** isn’t customizable enough, you can try out **react-dnd** which is great too and is made for broader use cases. I hope you have fun playing this game \(try playing it with just your keyboard\) and building your own.