import React from 'react';
import './App.css';
import logoImage from "./dance.png";
import { questions } from "./questions";

const GameModes = {
  defs: "defs" as const,
};

type DefsMode = typeof GameModes.defs;
type GameMode = DefsMode;

type GameStat = Record<GameMode, {
  [question: string]: {
    numer: number, // number of correct guesses
    denom: number; // total number of guesses
    wrong: boolean; // wrongly guessed in last run?
  }
}>;

interface Entry {
  question: string;
  answer: string;
  wrong: boolean;
  bucket: number;
}



// const tableByName = Object.entries(questions).reduce<Record<string, string>>((table, [s, n]) => { table[n] = s; return table }, {});

const storagePrefix = "15_06_2022"

function gameModeSerializer() {
  const storageKey = `${storagePrefix}_GAME_MODE`;
  const stored = localStorage.getItem(storageKey)
  const initial: GameMode = stored as GameMode || GameModes.defs;

  const serialize = (mode: GameMode) => localStorage.setItem(storageKey, mode);
  return { initial, serialize };
}

function gameStatSerializer() {
  const storageKey = `${storagePrefix}_GAME_STAT`;
  const stored = localStorage.getItem(storageKey);

  const initial: GameStat = stored
    ? JSON.parse(stored) as GameStat
    : {
      [GameModes.defs]: {},
    };

  const serialize = (state: GameStat) => localStorage.setItem(storageKey, JSON.stringify(state));
  return { initial, serialize };

}

const gameModeState = gameModeSerializer();
const gameStatState = gameStatSerializer();

function updateGameStats(gs: GameStat, mode: GameMode, question: string, correct: boolean, wrong: boolean): GameStat {
  const cgs = gs[mode];
  let { numer, denom } = cgs[question] || { numer: 0, denom: 0 };

  numer += correct ? 1 : 0;
  denom += 1;

  gs = {
    ...gs,
    [mode]: {
      ...cgs,
      [question]: {
        numer,
        denom,
        wrong
      }
    }
  };

  console.log(JSON.stringify(gs, null, "\t"));

  return gs;
}

function getRandomEntries(mode: GameMode) {
  const stats = gameStatState.initial[mode];

  const table = questions; // mode === GameModes.defs ? questions : tableByName;

  const buckets: Record<number, Entry[]> = [];

  Object.entries(table).forEach(([answer, question]) => {
    const guess = stats[question] || { numer: 0, denom: 0, wrong: false };
    // If guessed correctly 1/1 -> 1000
    // If guessed incorrectly 0/1 -> 0, 1/2 -> 500, 2/3 -> 666, 3/4 -> 750
    // If not guessed yet -> 700
    // Last guess was wrong -> 0
    const bucket = guess.wrong
      ? 0
      : guess.denom
        ? Math.round(guess.numer * 1000 / guess.denom)
        : 700;
    const entries = buckets[bucket] = (buckets[bucket] || []);
    const random = Math.floor(Math.random() * entries.length);
    entries.splice(random, 0, { answer, question, wrong: guess.wrong, bucket });
  });

  const entries: readonly Entry[] = Object.values(buckets).concat().flat();
  console.log("entries", JSON.stringify(entries, null, "\t"));
  return entries;
}

const initialRandomEntries = {
  [GameModes.defs]: getRandomEntries(GameModes.defs),
}

function App() {

  const [randomEntries, setRandomEntries] = React.useState<ReadonlyArray<Entry | null>>([]);

  const [mode, setMode] = React.useState(gameModeState.initial);
  React.useEffect(() => gameModeState.serialize(mode), [mode]);

  React.useEffect(() => {
    setRandomEntries(initialRandomEntries[mode]);
  }, [mode]);

  const [stat, setStat] = React.useState(gameStatState.initial)
  React.useEffect(() => gameStatState.serialize(stat), [stat]);

  const isCorrectAnswer = React.useMemo(
    () => (correctAnswer: string, userAnswer: string) => correctAnswer.split(",").some(a => {
      return a.toLowerCase().trim() === userAnswer.toLowerCase().trim();
    }),
    []
  );

  // const onGuessNames = React.useCallback(() => {
  //   setMode(GameModes.names)
  //   setCorrect(0);
  //   setWrong(0);
  //   setCurrentIndex(0);
  // }, []);

  // const onGuessSymbols = React.useCallback(() => {
  //   setMode(GameModes.defs)
  //   setCorrect(0);
  //   setWrong(0);
  //   setCurrentIndex(0);
  // }, []);


  const [correct, setCorrect] = React.useState(0);
  const [wrong, setWrong] = React.useState(0);

  const [currentIndex, setCurrentIndex] = React.useState(0);

  const currentEntry: Entry | null = randomEntries[currentIndex] || null;

  const inputRef = React.useRef<HTMLInputElement>(null);
  const correctRef = React.useRef<HTMLElement>(null);
  const wrongRef = React.useRef<HTMLElement>(null);

  const lastWrongRef = React.useRef(-1);
  const lastWrongCnt = React.useRef(0);

  const advance = React.useCallback((wasWrong: boolean) => {
    if (wasWrong) {
      // Make sure we ask again after 5 questions
      setRandomEntries(rs => {
        const mrs = rs.slice();
        mrs.splice(currentIndex + 5, 0, currentEntry);
        return mrs;
      });
    }

    setCurrentIndex(index => {
      const next = index + 1;
      if (next === randomEntries.length) {
        alert("Klaar!");
        return 0;
      } else {
        return next;
      }
    })
  }, [currentEntry, currentIndex, randomEntries.length]);

  const onKey = React.useCallback((e: React.KeyboardEvent<HTMLInputElement>) => {
    if (currentEntry && correctRef.current && wrongRef.current && inputRef.current && e.key === "Enter") {
      const guess = e.currentTarget.value;
      if (!isCorrectAnswer(currentEntry.answer, guess)) {
        if (lastWrongRef.current !== currentIndex) {
          lastWrongRef.current = currentIndex;
          setWrong(w => w + 1);
        }

        lastWrongCnt.current += 1;

        if (lastWrongCnt.current >= 2) {
          alert(`Het juiste antwoord was\n\n${currentEntry.answer}`);
          advance(true);
        }

        correctRef.current.className = "";
        wrongRef.current.className = "wrong";

        setStat(gs => updateGameStats(gs, mode, currentEntry.question, false, true));
      } else {
        inputRef.current.value = "";

        const wasWrong = lastWrongRef.current === currentIndex;
        setStat(gs => updateGameStats(gs, mode, currentEntry.question, true, wasWrong));
        advance(wasWrong);

        if (!wasWrong) {
          setCorrect(c => c + 1);
        }

        correctRef.current.className = "correct";
        wrongRef.current.className = "";
      }
    }
  }, [advance, currentEntry, currentIndex, isCorrectAnswer, mode]);

  React.useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
      inputRef.current.value = "";
    }
    lastWrongCnt.current = 0;
    lastWrongRef.current = -1;
  }, [currentIndex]);

  const autoCapitalize = mode === GameModes.defs ? "on" : "off";

  return (
    <div className="App">
      {/* <div>
        <button onClick={onGuessNames} className={mode === GameModes.names ? "active" : ""}>Namen raden</button>
        &nbsp;
        <button onClick={onGuessSymbols} className={mode === GameModes.defs ? "active" : ""}>Symbolen raden</button>
      </div>
      <br /> */}
      <div>
        <span ref={correctRef}>
          <code>Juist:</code>
          <code>{correct}/{correct + wrong}</code>
        </span>
        &nbsp;
        &nbsp;
        &nbsp;
        <span ref={wrongRef}>
          <code>Fout:</code>
          <code>{wrong}/{correct + wrong}</code>
        </span>
      </div>
      <br />
      <input ref={inputRef} onKeyUp={onKey} autoComplete="off" autoCorrect="off" autoCapitalize={autoCapitalize} spellCheck="false" />
      <br />
      <br />
      <em className={currentEntry?.wrong ? "retry" : ""}>{currentEntry?.question}</em>
      <br />
      <br />
      <img src={logoImage} alt="ball" />
    </div>
  );
}

export default App;
