---
title: "Functional Refactoring: Wordle"
subtitle: "JavaScript to PureScript"
date: 2024-03-17T09:17:24-05:00
---

Functional programming (FP) has an odd status among developers.
It is[^cite] simultaneously loved and feared.
I suspect most programmers would agree that maps and folds are useful, but concepts like immutability and pure functions are more controversial, and the especially technical bits (e.g. [monads](/blog/monads)) are flat out polarizing: the people who use them do so obsessively, and those who don't avoid them like the plague.

[^cite]: According to what I've heard and seen among the small sample size of programmers I interact with.

I'm interested in uncovering the degree to which FP is practically useful, and I'd like to find a ways to incorporate some of the useful patterns[^easy-win] from FP into my programming in non functional-centered languages.

[^easy-win]: Some patterns will undoubtedly be harder to implement than others (e.g. currying would be very syntactically painful to do in C-style language),
but there are undoubtedly insights from FP that can be applied for easy wins.

## The Game Plan

I have a JavaScript program that I originally wrote when I was in high-school[^original-write].
I'm going to re-write the program in PureScript, trying to make it maximally congruent to FP idioms[^FP-noob].

[^original-write]: I wrote it during my senior year, in "Microsoft Office Specialist" class: it was the only class I had in the "computer lab", and it was self-paced, online, so I completed all of the course work in less than a month and spent the rest of the time working on this. The computer teacher was using "implement Wordle" as a project for one of the CS classes, and I suggested the idea of writing a solver. The majority of the HTML/CSS for the game is courtesy of Mr. Phillips.

[^FP-noob]: I'm still relatively new to FP (especially for the front-end), so I'll do this to the greatest extent I can manage in a reasonable period of time.

My hope is that this will illustrate the organizational (complexity-unraveling) nature of FP and provide insights (and practice) applying it in a practical setting.
I've also never written a web-page in a pure-functional style, and I want to better understand the buzz around tools like Halogen and Elm that are stridently dedicated to this.

## The Original Version (JS)

### ([Github Link](https://github.com/lucasscharenbroch/wordle-solver))

The code to be refactored is a [Wordle](https://en.wikipedia.org/wiki/Wordle) clone and (primitive) solver.
It's around 550 lines of vanilla JS.
Most of the code wraps the game/solver logic, and the rest involves DOM manipulation (the two are closely intertwined).

{{< highlight javascript >}}
// wordle.js

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Global State ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

const testButton, testStatus, rows; // DOM elements
var currentWordIndex, currentWord, guessedRows, enteredWord, solved; // game state
var stopTesting, numSolved, numUnsolved, testMode, currentTestcase; // testing state

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Pure DOM Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function fillKeyboardAndBox(boxElement, sKey, sColor) { ... } // helper for updating the DOM
function changePageToSolver() { ... } // navigate to solver page

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ DOM Manipulation + Game-State Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function clickKey(letter) { ... } // handle keyboard input (update DOM accordingly)
function clickEnter() { ... } // execute game logic associated with word-entry (update DOM accordingly)
function win() { ... } // alert the user of a win, according to the global state
function lose() { ... } // alert the user of a loss, according to the global state
function reset() { ... } // reset the DOM and global state
function executeGuess(guess) { ... } // update game state/DOM according to the string `guess'
function solve() { ... } // solve the current game state, update the DOM accordingly
function resetTestingVars() { ... } // reset global testing state and DOM
function testAllWords() { ... } // set global state to "test mode", begin testing
function testWord() { ... } // run one word through the game (in the DOM);
                            // schedule another call or abort, according to the current state and DOM
{{< /highlight >}}

{{< highlight javascript >}}
// solving.js

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Constants ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

var GREEN, YELLOW, GRAY;
var wordList, validWords;

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Global State ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

const settingsBox, infoBox; // DOM elements
var wordbase, // selected word list

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Data Types ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

class wordRequirements {
    constructor(mustMatch, mustNotMatch, mustExist, mustNotExist) {
        this.mustMatch = mustMatch;
        this.mustNotMatch = mustNotMatch;
        this.mustExist = mustExist;
        this.mustNotExist = mustNotExist;
    }
}

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Pure Functions ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function charMatch(a, b) { ... } // case-insensitive string equality
function stringReplace(string, index, newCharacter) { ... } // return a new string, updated at given index
function inc(object, key) { ... } // hash-table increment-or-default
function countInArray(arr, item) { ... } // count occurrences of `item' in `arr'
function indexOfNthInstance(arr, item, n) { ... } // find index of `n'th occurrence of `item' in `arr'

function formulateGuess(possibleGuesses) { ... } // return "best guess" given all possible answers
function mostRepresentativeGuess(bodyToRepresent, chooseFrom) { ... } // find the "best guess" in `choseFrom' given
                                                                      // all possible answers (`bodyToRepresent')

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Pure DOM Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function showSettingsBox() { ... } // toggle property of DOM element
function hideSettingsBox() { ... } // toggle property of DOM element
function showInfoBox() { ... } // toggle property of DOM element
function hideInfoBox() { ... } // toggle property of DOM element

function getWordRequirements() { ... } // return a wordRequirements object, according to the DOM
function isValidGuess(guess) { ... } // return whether `guess' is valid, based on DOM

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ DOM Manipulation + Global-State Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function displayEnteredWord() { ... } // update DOM according to global state
function updateWordbase() { ... } // update global state according to DOM

{{< /highlight >}}

{{< highlight javascript >}}
// solver.js

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Global State ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

const rows, boxes; // DOM elements
var guessedRows, currentBox, enteredWord, isCurrentRowGuessed, possibleGuesses; // solver state

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ Pure DOM Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function changePageToGame() { ... } // navigate to game page

/* ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ DOM Manipulation + Global-State Manipulation ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ */

function reset() { ... } // reset DOM and global state
function setColor(color) { ... } // update DOM and global state
function backspace() { ... } // update DOM and global state
function getGuess() { ... } // generate a guess, updating the DOM and global state accordingly
function invalidWord() { ... } // set current guess (from global state) as invalid, call getGuess
{{< /highlight >}}

Besides a handful of noob mistakes and ugly kludges, at its core, this is a pretty standard JS program: DOM state and global variables are very closely linked, and the vast majority of functions are triggered by some event (or are indirectly by some event).

This seems to be a the most natural way to implement Wordle in HTML/JS.

## Enter PureScript

### ([Github Link](https://github.com/lucasscharenbroch/wordle-purs))

**PureScript** is a pure-functional language (heavily inspired by Haskell) that transpiles to JavaScript.
**Halogen** is a library for PureScript for writing reactive web-pages.

### UI

In Halogen, instead of writing HTML and attaching functions to it, the HTML itself is constructed[^optimization] by the **`render`** function, which takes in some **`State`**.

[^optimization]: Apparently this is optimized by some sort of HTML-diffing algorithm (kind of like in React), but I don't know the specifics of how it works.

{{< highlight haskell >}}
-- Page.purs

render :: forall slots. State -> HH.HTML (H.ComponentSlot slots Aff Action) Action
render state =
    HH.main_
      [ container state
      , whenElem state.showInfo (\_ -> infoBox state.currentPage)
      , whenElem state.showSettings (\_ -> settingsBox state)
      ]

container :: forall w. State -> HH.HTML w Action
container state =
  HH.div
    [HP.id "container"]
    [ header state.currentPage
    , mkBoard (getBoard state)
    , (case state.currentPage of
        (Game {keyboard}) -> mkKeyboard keyboard
        (Solver _) -> solverButtons)
    ]
  where getBoard state = case state.currentPage of
          Game {board} -> board
          Solver {board} -> board

header :: forall w. Page -> HH.HTML w Action
mkBoard :: forall w. Board -> HH.HTML w Action
infoBox :: forall w. Page -> HH.HTML w Action
settingsBox :: State -> forall w. HH.HTML w Action
mkKeyboard :: forall w. KeyboardState -> HH.HTML w Action
solverButtons :: forall w. HH.HTML w Action

-- etc.
{{< /highlight >}}

{{< highlight haskell >}}
-- State.purs

type State =
  { showInfo :: Boolean
  , showSettings :: Boolean
  , useFullDict :: Boolean
  , currentPage :: Page
  }

data Page = Game GameState
          | Solver SolverState

type GameState =
  { keyboard :: KeyboardState
  , currentWord :: String
  , sentGuesses :: Array String
  , currentGuess :: String
  , board :: Board
  , testStatus :: TestStatus
  }

type SolverState =
  { guesses :: Array String
  , board :: Board
  , sentColorings :: Array (Array Color)
  , currentColoring :: Array Color
  , invalidWords :: Array String
  }
{{< /highlight >}}

The **`State`** is initialized in the main function (when the page is first loaded), and the **`runUI`** function calls **`render`** whenever the state is updated.

{{< highlight haskell "hl_lines=8 13-16" >}}
-- Main.purs

main :: Effect Unit
main =
  do gameSeed <- mkDefGameSeed
     HA.runHalogenAff
      do body <- HA.awaitBody
         runUI component gameSeed body

component :: forall output t. H.Component t Int output Aff
component =
  H.mkComponent
    { initialState
    , render
    , eval: H.mkEval $ H.defaultEval { handleAction = handleAction, initialize = Just InitKeybinds }
    }
{{< /highlight >}}

<b>`Action`</b>s are used to change the **`State`**: they can be emitted by HTML elements or events and are handled within **`handleAction`**.

{{< highlight haskell "hl_lines=14" >}}
-- Page.purs

wordleInfo :: forall w. HH.HTML w Action
wordleInfo =
  HH.div
    [ HP.id "infoBox"
    , HP.classes [HH.ClassName "messageBox"]
    ]
    [HH.h1_ [HH.text "Wordle"]
    , HH.text "Wordle is a fun word-guessing game. You have six attempts to guess a secret word."
    -- ...
    , HH.button
        [ HP.classes [ HH.ClassName "okButton" ]
        , HE.onClick \_ -> SetInfoBoxVis false -- (SetInfoBoxVis false) is an Action
        ]
        [HH.text "OK"]
    ]
{{< /highlight >}}

{{< highlight haskell >}}
-- State.purs

data Action = SetInfoBoxVis Boolean
            | SetSettingsBoxVis Boolean
            | TestAllWords
            | SetUseDictWords
            | SetUseWordleWords
            | ChangePageToGame
            | ChangePageToSolver
            | Reset
            | PressKeyButton Key
            | PressColorKey Color
            | GenerateGuess
            | RegenerateGuess
            | SolveGame
            | InitKeybinds
            | HandleKeypress KeyboardEvent
            | PressEnter
            | TestStep
            | StopTesting

handleAction :: forall o. Action -> H.HalogenM State Action () o Aff Unit
handleAction = case _ of
  SetInfoBoxVis isVis -> H.modify_ (\s -> s {showInfo = isVis})
  SetSettingsBoxVis isVis -> H.modify_ (\s -> s {showSettings = isVis})
  SetUseDictWords -> H.modify_ (\s -> s {useFullDict = true})
  SetUseWordleWords -> H.modify_ (\s -> s {useFullDict = false})
  ChangePageToSolver -> H.modify_ (\s -> s {currentPage = Solver defSolverState})
  ChangePageToGame -> mkGameStateM >>= \gs -> H.modify_ (\s -> s {currentPage = Game gs})
  PressKeyButton k -> pressKeyButton k
  InitKeybinds -> initKeybinds
  HandleKeypress event -> handleKeypressEvent event
  PressEnter -> pressEnter
  Reset -> resetState
  GenerateGuess -> generateGuess
  RegenerateGuess -> regenerateGuess
  PressColorKey c -> pressColorButton c
  SolveGame -> solveGame
  TestAllWords -> testAllWords
  TestStep -> testStep
  StopTesting -> stopTesting

pressKeyButton :: forall o. Key -> H.HalogenM State Action () o Aff Unit
initKeybinds :: forall o. H.HalogenM State Action () o Aff Unit
handleKeypressEvent :: forall o. KeyboardEvent -> H.HalogenM State Action () o Aff Unit
pressEnter :: forall o. H.HalogenM State Action () o Aff Unit
resetState :: forall o. H.HalogenM State Action () o Aff Unit
generateGuess :: forall o. H.HalogenM State Action () o Aff Unit
regenerateGuess :: forall o. H.HalogenM State Action () o Aff Unit
pressColorButton :: forall o. Color -> H.HalogenM State Action () o Aff Unit
solveGame :: forall o. H.HalogenM State Action () o Aff Unit
testAllWords :: forall o. H.HalogenM State Action () o Aff Unit
testStep :: forall o. H.HalogenM State Action () o Aff Unit
stopTesting :: forall o. H.HalogenM State Action () o Aff Unit
{{< /highlight >}}

The above functions are the most impure part of the program: through the **`H.HalogenM`** monad, they have complete access to the entirity of the current **`State`** and the outside world (through the **`Effect`** and **`Aff`** monads).
There is no constraint on where (or what **`State`**) <b>`Action`</b>s can come from, so <b>`Action`</b>s that assume a certain **`State`** have to do a dispatch (case-analysis) on the current **`State`** to unwrap the data they're looking for.
This is useful for actions like **`Reset`** that have defined behavior regaurdless of state but inconvenient/hypothetically-bug-prone for actions like **`SolveGame`** that expect for some invariant to be true.

{{< highlight haskell >}}
-- State.purs

resetState :: forall o. H.HalogenM State Action () o Aff Unit
resetState =
  do state <- H.get
     state' <- case state.currentPage of
                 Game _ -> do gState <- mkGameStateM
                              pure $ defState {useFullDict = state.useFullDict, currentPage = Game $ gState}
                 Solver _ -> pure $ defState {useFullDict = state.useFullDict}
     H.put state'

solveGame :: forall o. H.HalogenM State Action () o Aff Unit
solveGame =
  do state <- H.get
     case state.currentPage of
       Solver _ -> pure unit -- do nothing if the current page is Solver
       Game gState -> do gState' <- solveGameState (getWordList state) gState
                         H.put $ state {currentPage = Game $ gState'}
{{< /highlight >}}

### Game Logic

This is where PureScript really shines.
Wordle requires a somewhat-involved[^involved] algorithm for coloring guesses.
Matching letters are colored green, then remaining letters in the guess that correspond to letters in the secret word (the word-to-be-guessed) are colored yellow; all remaining letters are colored gray.
The (somewhat) tricky part is the one-to-one correspondence[^hs-missed].
For example, guessing "LLAMA" when the secret word is "LANCE"
would yeild "GREEN GRAY YELLOW GRAY GRAY", not "GREEN YELLOW YELLOW GRAY YELLOW" (the duplicate "L" and "A" are gray, not yellow).
This isn't mathematically difficult or anything, but it makes the implementation trickier (it involves counting rather than a trivial mapping+lookup), and that's reflected in the code.

[^involved]: It's trickier to implement than one would initially expect.

[^hs-missed]: My high-school CS teacher missed this.

Here's the original version, in JS:

{{< highlight javascript >}}
function clickEnter() {
    if (enteredWord.length !== 5)
        return alert("Please enter a word of length 5.");
    else if(validWords.indexOf(enteredWord.toLowerCase()) == -1)
        return alert("Word not in word list.");

    let boxes = rows[guessedRows].getElementsByClassName("box");

    //check letters & fill.
    let dummyWord = currentWord.toUpperCase(); // make a copy of current(will remove characters to prevent double-matches)
    let lettersCorrect = 0;

    // check for exact matches.
    for(let i = 0; i < 5; i++) {
        if(charMatch(currentWord[i], enteredWord[i])) { // match
            fillKeyboardAndBox(boxes[i], enteredWord[i], GREEN); // set color
            dummyWord = stringReplace(dummyWord, i, '-'); // remove this letter from dummy word. (prevent double-matching)
            enteredWord = stringReplace(enteredWord, i, '-'); // remove this letter from entered word to prevent double matching
            lettersCorrect++;
        } else {
            fillKeyboardAndBox(boxes[i], enteredWord[i], GRAY); // set gray
        }
    }

    // check for partial matches
    for(let i = 0; i < 5; i++) {
        if(enteredWord[i] == '-') // this index is already matched
            continue;

        let index;
        if((index = dummyWord.indexOf(enteredWord[i])) != -1) { // found match...
            fillKeyboardAndBox(boxes[i], enteredWord[i], YELLOW);
            dummyWord = stringReplace(dummyWord, index, '-'); // remove letter from dummy word.
        }
    }

    // reset rows and word
    guessedRows++;
    enteredWord = "";

    if(lettersCorrect === 5) { // guess is correct
        solved = true;
        setTimeout(win, 10);
    } else if(guessedRows === 6) { // all guesses expended
        setTimeout(lose, 10);
    }
}
{{< /highlight >}}

And the PureScript:

{{< highlight haskell >}}
gradeGuess_ :: String -> String -> Array Cell
gradeGuess_ = on gradeGuess toCharArray

gradeGuess :: Array Char -> Array Char -> Array Cell
gradeGuess correct given = zipWith (\letter color -> {letter, color}) given $ colors
    where
          step (Tuple _ unmatched) (Tuple letter isGreen)
            | isGreen = (Tuple Green unmatched)
            | letter `elem` unmatched = (Tuple Yellow (delete letter unmatched))
            | otherwise = (Tuple Gray unmatched)
          colors = map fst <<< scanl step (Tuple Gray unmatched) $ zip given isGreen
            where
              unmatched = map fst <<< filter (not <<< snd) $ zip correct isGreen
              isGreen = zipWith (==) given correct
{{< /highlight >}}

The size-comparison alone isn't really fair, since the JS version has all the extra page-related logic, but even without that, the functional version is way cleaner and less bug-prone.

Readability is debatable: the scan in the functional version is admittedly cryptic[^tuple] (perhaps there's a better way to do the one-to-one mapping), but once that part is understood, it's way easier to be confident in exactly what **`gradeGuess`** does than in what **`clickEnter`** does (even without the side-effects of **`clickEnter`** (the imperative style alone is flimsy)).

[^tuple]: The Tuple constructor doesn't help with readability either - the Haskell version would be 30 characters shorter for that reason alone.

We can do a similar comparison for the heart of the solving algorithm: determining whether a given word fits the board.

{{< highlight javascript >}}
class wordRequirements {
    constructor(mustMatch, mustNotMatch, mustExist, mustNotExist){
        this.mustMatch = mustMatch;
        this.mustNotMatch = mustNotMatch;
        this.mustExist = mustExist;
        this.mustNotExist = mustNotExist;
    }
}

function getWordRequirements() {
    let mustMatch = "-----";
    let mustNotMatch = [{},{},{},{},{}];
    let mustExist = {};
    let mustNotExist = {};

    // set mustMatch, mustNotMatch and mustNotExist
    // any greens / yellows / grays
    for(let row = 0; row < guessedRows; row++) {
        let boxes = rows[row].getElementsByClassName("box");

        for(let b = 0; b < 5; b++) {
            let letter = boxes[b].innerHTML;
            if(boxes[b].style.backgroundColor == GREEN) { // must match
                mustMatch = stringReplace(mustMatch, b, boxes[b].innerHTML);
            }
            if(boxes[b].style.backgroundColor == YELLOW) { // must not match
                mustNotMatch[b][letter] = true;
            } else if(boxes[b].style.backgroundColor == GRAY) { //must not exist
                mustNotExist[letter] = true;
            }
        }
    }

    // mustExist
    // max yellows of any letter in any row

    for(let row = 0; row < guessedRows; row++) {
        let boxes = rows[row].getElementsByClassName("box");
        let yellowsPerLetter = {};

        for(let b = 0; b < 5; b++) {
            let letter = boxes[b].innerHTML;

            if(boxes[b].style.backgroundColor == YELLOW) {
                inc(yellowsPerLetter, letter);
            }
        }

        // check total letters in row- if the # of yellows for any respective leter
        // exceeds the # of that letter in mustExist, add the difference to mustExist.
        for(letter in yellowsPerLetter) {
            if(mustExist[letter] == undefined || yellowsPerLetter[letter] > mustExist[letter]) {
                mustExist[letter] = yellowsPerLetter[letter];
            }
        }
    }

    let req = new wordRequirements(mustMatch, mustNotMatch, mustExist, mustNotExist);
    return req;
}

function isValidGuess(guess) {
    let requirements = getWordRequirements();

    let mustMatch = requirements.mustMatch;
    let mustExist = requirements.mustExist;
    let mustNotMatch = requirements.mustNotMatch;
    let mustNotExist = requirements.mustNotExist;

    guess = guess.toUpperCase(); // make all cases match

    let dummyWord = guess.slice(); // copy of guess

    // greens (must match)
    // mask and check, remove matched letters from guess

    for(let c = 0; c < 5; c++) {
        if(mustMatch[c] != '-') {
            if(!(guess[c] == mustMatch[c])) {
                return false;
            } else {
                dummyWord = stringReplace(dummyWord, c, '-');
            }
        }
    }

    // must exist (check for instances in GUESS, modify the DUMMY WORD)
    for (letter in mustExist) {
        if(mustExist[letter] > countInArray(guess, letter))
            return false;
        else {
            //mask that many instances of that letter (read from guess, mask in dummy)
            for(let i = 0; i < mustExist[letter]; i++) {
                dummyWord = stringReplace(dummyWord, dummyWord.indexOf(letter), '-');
            }
        }
    }

    // must not match/ must not exist
    // check each letter
    for(let i = 0; i < 5; i++) {
        if(mustNotMatch[i][guess[i]])
            return false;
        if(mustNotExist[dummyWord[i]])
            return false;
    }

    return true;
}
{{< /highlight >}}

{{< highlight haskell >}}
wordsFittingBoard :: Array String -> Board -> Array String
wordsFittingBoard words board = filter fitsConstraints words
  where
    fitsPos = fitsPosConstraints board
    fitsCnts = fitsCntConstraints board
    fitsConstraints s = fitsPos s && fitsCnts s

-- yellows => net minimum char count
-- grays => net maximum char count
fitsCntConstraints :: Board -> String -> Boolean
fitsCntConstraints board = _fitsCntConstraints
  where
    minCnts = foldl (Map.unionWith max) Map.empty <<< map rowMinCnts $ board
    maxCnts = foldl Map.union Map.empty $ map rowMaxCnts $ board
    rowToColorCountMap color = countIntoMap <<< map (\c -> c.letter) <<< filter (\c -> c.color == color)
    rowMinCnts = rowToColorCountMap Yellow
    rowMaxCnts row = mapWithKey (\c _ -> lookupOr 0 c yellows + lookupOr 0 c greens) grays
      where
        grays = rowToColorCountMap Gray $ row
        yellows = rowToColorCountMap Yellow $ row
        greens = rowToColorCountMap Green $ row
    _fitsCntConstraints s = Map.intersection minCnts guessCnts == minCnts
                         && Foldable.all id (Map.intersectionWith (<=) minCnts guessCnts)
                         && Foldable.all id (Map.intersectionWith (>=) maxCnts guessCnts)
      where
        guessCnts = countIntoMap <<< toCharArray $ s

data CurriedEqualityCheck a = CurriedEq a
                            | CurriedNe a

derive instance curriedEqalityCheckEq :: Eq a => Eq (CurriedEqualityCheck a)

applyCurriedEqualityCheck :: forall a. Eq a => CurriedEqualityCheck a -> a -> Boolean
applyCurriedEqualityCheck (CurriedEq x) y = x == y
applyCurriedEqualityCheck (CurriedNe x) y = x /= y

-- greens => exact position match
-- yellows, grays => position mismatch
fitsPosConstraints :: Board -> String -> Boolean
fitsPosConstraints board = \s -> all id $ zipWith ($) charFns (toCharArray s)
  where
    charFns = map conjChecks charChecks
    conjChecks checks x = all (flip ($) x) checks'
      where checks' = map applyCurriedEqualityCheck $ nubEq checks
    charChecks :: Array (Array (CurriedEqualityCheck Char))
    charChecks = foldl (zipWith (<>)) (replicate 5 []) cellChecks
    cellChecks :: Array (Array (Array (CurriedEqualityCheck Char)))
    cellChecks = map (map getCellConstraint) board
    getCellConstraint :: Cell -> Array (CurriedEqualityCheck Char)
    getCellConstraint {letter, color} =
      case color of
        Green -> [CurriedEq letter]
        Yellow -> [CurriedNe letter]
        Gray -> [CurriedNe letter]
        None -> []
{{< /highlight >}}

This time, the JS version doesn't have any side-effects besides its reading the board from the DOM.

This example is pretty representative of the differences between functional and imperative styles: the imperative (JS) version does a chain of local modifications, setting up new (and heavily relying on previous) invariants, until the invariant is the desired result.
The functional version makes a seres of interlinked global observations.

I.E.

Imperative (JS):
- Initialize variables for requirements, guess, and dummyWord
- Incrementally modify dummyWord, ensuring it abides by each requirement
- Report the result according to dummyWord and requirements

Functional (PURS):
- Take the words and board as arguments
- Apply helper functions that deal with narrower versions of the requirements
- Combine the results of those helpers

The PureScript version uses partial application to avoid re-computation of the constraint data structures (e.g. **`minCnts`**, **`maxCnts`**, and **`charFns`**) (analagous to **`wordRequirements`** in the JS version) for each word.
This makes for a nice mix of modularity and efficiency: the caller (**`wordsFittingBoard`**) need not compute, nor even know about these structures.

The functions that begin with "_" are necessary because PureScript waits until all variables are recieved before computing the values of variables in the where-clauses.

Here's the transpiled version of fitsPosConstraints: note that the majority of the computation is done immediately after **`board`** is recieved (and before **`s`** is recieved: the values of the variables are stored in the closure).

{{< highlight javascript "hl_lines=26-28" >}}
var fitsPosConstraints = function (board) {
    var getCellConstraint = function (v) {
        if (v.color instanceof Green) {
            return [ new CurriedEq(v.letter) ];
        };
        if (v.color instanceof Yellow) {
            return [ new CurriedNe(v.letter) ];
        };
        if (v.color instanceof Gray) {
            return [ new CurriedNe(v.letter) ];
        };
        if (v.color instanceof None) {
            return [  ];
        };
        throw new Error("Failed pattern match at Wordle (line 111, column 7 - line 115, column 19): " + [ v.color.constructor.name ]);
    };
    var conjChecks = function (checks) {
        return function (x) {
            var checks$prime = map(applyCurriedEqualityCheck1)(nubEq(checks));
            return Data_Array.all(Data_Function.flip(Data_Function.apply)(x))(checks$prime);
        };
    };
    var cellChecks = map(map(getCellConstraint))(board);
    var charChecks = Data_Array.foldl(Data_Array.zipWith(append))(Data_Array.replicate(5)([  ]))(cellChecks);
    var charFns = map(conjChecks)(charChecks);
    return function (s) {
        return Data_Array.all(Util.id)(Data_Array.zipWith(Data_Function.apply)(charFns)(Data_String_CodeUnits.toCharArray(s)));
    };
};
{{< /highlight >}}

## Reservations

The above examples do a pretty good job at highlighting the upsides of FP.
While Halogen+PureScript is a novel way of approaching UI development, there are a lot of factors (many of them being downright negative) to consider before using it seriously.

- Steep Learning Curve
- Questionable Performance
- Compilation Step
- Boilerplate[^purs-lc]
- Complexity in Type System (from complexity of DOM)
    - Difficulty Doing Easy Things[^difficulty-doing-easy-things]
- Sparse Documentation (Younger, Smaller, etc.)

[^difficulty-doing-easy-things]: For example, it took me a considerable amount of time figuring out how to handle kepyresses and to implement a timer-like regularly-recurring function (for testing).
There's a lot more complex logic required to do these in PureScript, and documentation isn't great.

[^purs-lc]: The line-count for the PureScript (~1,000) is nearly 2x the JavaScript version. This is partially because of the code for the HTML, but even accounting for that, it's still a few hundred more lines.
The non-UI logic is significantly shorter in the PureScript version, though, as shown in the examples above.

The web is a very complicated beast.
JavaScript embraces the complexity, making for an unsafe-yet-easy tool.
Halogen tries really hard to formalize it, making for a mostly-safe-yet-difficult tool.

There has to be a certain requirement for safety to warrant bothering with all of the downsides of something like Halogen.
Most web-pages don't require such safety, and it's rarely the main focus, since there are so many other things that can go wrong on the web (usually on the back-end).
This along with all of the other middle-grounds out there (e.g. TypeScript + other frameworks/libraries, which offer more safety than JS and more ease-of-use than Halogen) makes me skeptical towards the practical use of Halogen-like tools for the web.
