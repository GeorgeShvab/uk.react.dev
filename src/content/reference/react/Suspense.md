---
title: <Suspense>
---

<Intro>

`<Suspense>` lets you display a fallback until its children have finished loading.

`<Suspense>` дозволяє вам відображати запасний варіант доки його дочірні компоненти не завершать завантаження.


```js
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

</Intro>

<InlineToc />

---

## Опис {/*reference*/}

### `<Suspense>` {/*suspense*/}

#### Пропси {/*props*/}
* `children`: UI, який ви хочете відрендерити. Якщо `children` затримується під час рендерингу, Затримка перемкнеться на рендер `fallback`.
* `fallback`: Альтернативний UI, який рендериться замість справжнього UI, якщо той не завершив завантаження. Будь-який валідний React-вузол приймається, хоча на практиці, запасний варіант це легкий заповнювач, такий як спінер чи скелетон. Затримка автоматично перемкнеться на `fallback`, коли `children` затримується, і назад на `children`, коли дані будуть готовими. Якщо `fallback` затримується під час рендеру, найближча батьківська границя Затримки буде активована.

#### Застереження {/*caveats*/}

- React не зберігає жодного стейту для рендерів, які були затримані до того, як змогли вперше змонтуватися. Коли компонент завантажився, React ще раз спробує відрендерити затримане дерево з нуля.
- Якщо Затримка відображала контент для дерева, але тоді знову затрималася, `fallback` буде показаний знову, хіба що оновлення яке це спричинило, зумовлене [`startTransition`](/reference/react/startTransition) або [`useDeferredValue`](/reference/react/useDeferredValue).
- Якщо React потрібно сховати вже видимий контент, бо він знову затримався, він скине [ефекти макету](/reference/react/useLayoutEffect) в дереві вмісту. Коли вміст буде знову готовий для показу, React викличе ефекти макету знову. Це гарантує, що ефекти, які вимірюють DOM-макет, не намагатимуться робити цього доки контент схований.
- React включає внутрішні оптимізації, такі як *Потоковий рендеринг на стороні сервера* і *Вибіркову гідрацію*, які інтегровані в Затримку. Прочитайте [архітектурний огляд](https://github.com/reactwg/react-18/discussions/37) і подивіться [технічну розмову](https://www.youtube.com/watch?v=pj5N-Khihgc) щоб дізнатися більше.

- React does not preserve any state for renders that got suspended before they were able to mount for the first time. When the component has loaded, React will retry rendering the suspended tree from scratch.
- If Suspense was displaying content for the tree, but then it suspended again, the `fallback` will be shown again unless the update causing it was caused by [`startTransition`](/reference/react/startTransition) or [`useDeferredValue`](/reference/react/useDeferredValue).
- If React needs to hide the already visible content because it suspended again, it will clean up [layout Effects](/reference/react/useLayoutEffect) in the content tree. When the content is ready to be shown again, React will fire the layout Effects again. This ensures that Effects measuring the DOM layout don't try to do this while the content is hidden.
- React includes under-the-hood optimizations like *Streaming Server Rendering* and *Selective Hydration* that are integrated with Suspense. Read [an architectural overview](https://github.com/reactwg/react-18/discussions/37) and watch [a technical talk](https://www.youtube.com/watch?v=pj5N-Khihgc) to learn more.

---

## Використання {/*usage*/}

### Відображення запасного варіанту доки контент завантажується {/*displaying-a-fallback-while-content-is-loading*/}

You can wrap any part of your application with a Suspense boundary:

Ви можете загорнути будь-яку частину вашого додатку в границю Затримки:

```js [[1, 1, "<Loading />"], [2, 2, "<Albums />"]]
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

React відобразить ваш <CodeStep step={1}>запасний варіант завантаження</CodeStep> доки всесь код та дані, які потребує <CodeStep step={2}>дочірній компонент</CodeStep>, не будуть завантажені.

React will display your <CodeStep step={1}>loading fallback</CodeStep> until all the code and data needed by <CodeStep step={2}>the children</CodeStep> has been loaded.

У прикладі вище, компонент `Albums` *затримується* доки отримує список альбомів. Доки він не буде готовим для рендеру, React переключиться на найближчу границю Затримки вверху дерева щоб показати запасний варіант - ваш компонент `Loading`. Тоді, коли дані завантажені, React сховає запасний варіант `Loading` і відрендерить компонент `Albums` з даними.

In the example below, the `Albums` component *suspends* while fetching the list of albums. Until it's ready to render, React switches the closest Suspense boundary above to show the fallback--your `Loading` component. Then, when the data loads, React hides the `Loading` fallback and renders the `Albums` component with data.

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Відкрити сторінку виконавця The Beatles
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Завантаження...</h2>;
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент написаний використовуючи експериментальний API
// це ще не доступно в стабільних версіях React.

// Note: this component is written using an experimental API
// that's not yet available in stable versions of React.

// Для реалістичних прикладів, який ви можете спробувати насьогодні, спробуйте фреймворк
// що підтримує Затримувач, такий як Relay чи Next.js.

// For a realistic example you can follow today, try a framework
// that's integrated with Suspense, like Relay or Next.js.

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це лазівка для багу, щоб запустити демонстративний варіант.
// TODO: замінити реальною імплементацією, коли баг буде виправлено.

// This is a workaround for a bug to get the demo running.
// TODO: replace with real implementation when the bug is fixed.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить від
// фреймворка який ви використовуєте разлм з Затримувачем.
// Зазвичай, логіка кешування буде всередині фреймоврку.

// Note: the way you would do data fetching depends on
// the framework that you use together with Suspense.
// Normally, the caching logic would be inside a framework.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else {
    throw Error('Not implemented');
  }
}

async function getAlbums() {
  // Add a fake delay to make waiting noticeable.

  // Додаємо штучну затримку щоб зробити чекання помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

</Sandpack>

<Note>

**Only Suspense-enabled data sources will activate the Suspense component.** They include:

**Тільки дані, які підтримують Затримувач, активуватимуть компонент Затримувач.** Вони включають:

- Отримання даних з фреймворком, що підтримує Затримувач, таким як [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) та [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Компоненти з відкладеним завантаженням, які використовують [`lazy`](/reference/react/lazy)
- Зчитування значення Промісу з [`use`](/reference/react/use)

- Data fetching with Suspense-enabled frameworks like [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) and [Next.js](https://nextjs.org/docs/getting-started/react-essentials)
- Lazy-loading component code with [`lazy`](/reference/react/lazy)
- Reading the value of a Promise with [`use`](/reference/react/use)

Затримувач **не** виявляє коли дані отримуються всередині Ефекту чи обробника подій.

Шлях яким ви будете отримувати дані в компоненті `Albums` вище, залежить від вашого фреймоврку. Якщо ви використовуєте фреймворк з підтримкою Затримувача, ви знайдете деталі в його документації щодо отримання даних.

Отримання даних із Затримувачем але без використання фреймворку доки не підтримується. Вимоги для імплементації джерела даних з підтримуваним Затримувачем не стабільні й не задокументовані. Офіційний API для інтеграції джерел даних з підтримкою Затримувача буде випущено в майбутній версії React. 

Suspense **does not** detect when data is fetched inside an Effect or event handler.

The exact way you would load data in the `Albums` component above depends on your framework. If you use a Suspense-enabled framework, you'll find the details in its data fetching documentation.

Suspense-enabled data fetching without the use of an opinionated framework is not yet supported. The requirements for implementing a Suspense-enabled data source are unstable and undocumented. An official API for integrating data sources with Suspense will be released in a future version of React. 

</Note>

---

### Розкриття вмісту разом одночасно {/*revealing-content-together-at-once*/}

By default, the whole tree inside Suspense is treated as a single unit. For example, even if *only one* of these components suspends waiting for some data, *all* of them together will be replaced by the loading indicator:

За замовчуванням, все дерево всередині Затримувача сприймається як один компонент. Для прикладу, навіть якщо *тільки один* з цих компонентів затримується чекаючи на дані, *всі* з них буде замінено на інликатор завантаження:

```js {2-5}
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

Тоді, коли всі з них будуть готові до відображення, вони зв'являться всі разом в один момент.

Then, after all of them are ready to be displayed, they will all appear together at once.

В прикладі нижче, обидва компоненти `Biography` і `Albums` отримують якісь дані. Проте, через те що вони згруповані всередині одного затримувача, ці компоненти завжди "з'являтимуться" разом одночасно.

In the example below, both `Biography` and `Albums` fetch some data. However, because they are grouped under a single Suspense boundary, these components always "pop in" together at the same time.

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Відкрити сторінку виконавця The Beatles
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Biography artistId={artist.id} />
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Завантаження...</h2>;
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/Biography.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 1500);
  });

  return `The Beatles were an English rock band, 
    formed in Liverpool in 1960, that comprised 
    John Lennon, Paul McCartney, George Harrison 
    and Ringo Starr.`;
}

async function getAlbums() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
```

</Sandpack>

Components that load data don't have to be direct children of the Suspense boundary. For example, you can move `Biography` and `Albums` into a new `Details` component. This doesn't change the behavior. `Biography` and `Albums` share the same closest parent Suspense boundary, so their reveal is coordinated together.

Компоненти що завантажують дані не повинні бути прямими дочірніми компонентами границі Затримки. Наприклад, ви можете перенести `Biography` і `Albums` у новий компонент `Details`. Це не вплине на поведінку. `Biography` і `Albums` поділяють однакову найближчу батьківську границю Затримки, тому їхнє відображення координується разом.

```js {2,8-11}
<Suspense fallback={<Loading />}>
  <Details artistId={artist.id} />
</Suspense>

function Details({ artistId }) {
  return (
    <>
      <Biography artistId={artistId} />
      <Panel>
        <Albums artistId={artistId} />
      </Panel>
    </>
  );
}
```

---

### Відображення вкладеного контенту по мірі завантаження {/*revealing-nested-content-as-it-loads*/}

### Revealing nested content as it loads {/*revealing-nested-content-as-it-loads*/}

When a component suspends, the closest parent Suspense component shows the fallback. This lets you nest multiple Suspense components to create a loading sequence. Each Suspense boundary's fallback will be filled in as the next level of content becomes available. For example, you can give the album list its own fallback:

Коли компонент затримується, найближчий батьківський компонент Затримки показує запасний варіант. Це дозволяє вам вкладувати кілька компонентів Затримки щоб сворити послідовність завантаження. Кожен запасний варіант Затримки буде заповненим тоді як наступний рівень вмісту буде доступним. Наприклад, ви можете дати списку альбомів власний запасний варіант:

```js {3,7}
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

З цією зміною, для відображення `Biography` не потрібно "чекати" завантаження `Albums` to load.

Послідовність буде такою:

1. Якщо `Biography` ще не завантажився, `BigSpinner` буде показано замість всього вмісту.
1. Як тільки `Biography` закінчить завантаження, `BigSpinner` буде замінено вмістом.
1. Якщо `Albums` ще не завантажився, `AlbumsGlimmer` буде показано замість `Albums` і його батьківського компонента `Panel`.
1. Нарешті, як тільки `Albums` закінчить завантаження, він замінить `AlbumsGlimmer`.

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js hidden
import { useState } from 'react';
import ArtistPage from './ArtistPage.js';

export default function App() {
  const [show, setShow] = useState(false);
  if (show) {
    return (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  } else {
    return (
      <button onClick={() => setShow(true)}>
        Відкрити сторінку виконавця The Beatles
      </button>
    );
  }
}
```

```js src/ArtistPage.js active
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<BigSpinner />}>
        <Biography artistId={artist.id} />
        <Suspense fallback={<AlbumsGlimmer />}>
          <Panel>
            <Albums artistId={artist.id} />
          </Panel>
        </Suspense>
      </Suspense>
    </>
  );
}

function BigSpinner() {
  return <h2>🌀 Завантаження...</h2>;
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Panel.js
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/Biography.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles were an English rock band, 
    formed in Liverpool in 1960, that comprised 
    John Lennon, Paul McCartney, George Harrison 
    and Ringo Starr.`;
}

async function getAlbums() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

Suspense boundaries let you coordinate which parts of your UI should always "pop in" together at the same time, and which parts should progressively reveal more content in a sequence of loading states. You can add, move, or delete Suspense boundaries in any place in the tree without affecting the rest of your app's behavior.

Границі затримки допомагають вам координувати які частини UI повинні завжди з'являтися одночасно, і які частини повинні поступово показувати більше контенту в послідовності завантажених стейтів. Ви можете додавати, переставляти або видаляти границі Затримки в будь-якому місті в дереві без впливу на поведінку решти застосунку.

Don't put a Suspense boundary around every component. Suspense boundaries should not be more granular than the loading sequence that you want the user to experience. If you work with a designer, ask them where the loading states should be placed--it's likely that they've already included them in their design wireframes.

Не ставте границю Затримки навколо кожного компонента. Межі Затримки повинні бути не менш дрібними ніж послідовність завантаження яку ви хочете щоб пізнав користувач. Якщо ви працюєте з дизайнером, запитайте його, де повинні відображатися індикатори завантаження--висока вірогідність що вони вже включили їх у макети дизайну.

---

### Відображення контенту доки завантажується новий контент {/*showing-stale-content-while-fresh-content-is-loading*/}

### Showing stale content while fresh content is loading {/*showing-stale-content-while-fresh-content-is-loading*/}

In this example, the `SearchResults` component suspends while fetching the search results. Type `"a"`, wait for the results, and then edit it to `"ab"`. The results for `"a"` will get replaced by the loading fallback.

У цьому приклдаі, компонент `SearchResults` затримується доки завантажує результати пошуку. Введіть `"a"`, зачекайте на результат, а тоді змініть на `"ab"`. Результати для `"a"` будуть замінені запасним варінтом завантаження.

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <label>
        Пошук альбомів:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Завантаження...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>No matches for <i>"{query}"</i></p>;
  }
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Not implemented');
  }
}

async function getSearchResults(query) {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  const allAlbums = [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

A common alternative UI pattern is to *defer* updating the list and to keep showing the previous results until the new results are ready. The [`useDeferredValue`](/reference/react/useDeferredValue) Hook lets you pass a deferred version of the query down: 

Поширений альтернативний UI паттерн це *відкладати* оновлення списку і продовжувати показувати попередні результати доки нові результати не готові. Хук [`useDeferredValue`](/reference/react/useDeferredValue) дозволяє вам передавати відкладений варіант запиту вниз по дереву: 

```js {3,11}
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Пошук альбомів:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Завантаження...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

The `query` will update immediately, so the input will display the new value. However, the `deferredQuery` will keep its previous value until the data has loaded, so `SearchResults` will show the stale results for a bit.

`query` оновиться одразу, тому пошуковий рядок відображатиме нове значення. Проте, `deferredQuery` збереже попереднє значення доки дані не будуть завантажені, тож `SearchResults` на деякий час відобразить застарілі результати.

To make it more obvious to the user, you can add a visual indication when the stale result list is displayed:

Щоби зробити це більш очевидним для користувача, ви можете додати візуальний індикатор при відображенні застарілого результату:

```js {2}
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1 
}}>
  <SearchResults query={deferredQuery} />
</div>
```

Enter `"a"` in the example below, wait for the results to load, and then edit the input to `"ab"`. Notice how instead of the Suspense fallback, you now see the dimmed stale result list until the new results have loaded:

Введіть `"a"` у прикладі нижчее, зачекайте доки результи завантажаться, і тоді змініть поле вводу на `"ab"`. Зверніть увагу як замість запасного варіанту Затримки, ви бачите затемнений список попередніх результатів доки нові результати не завантажилися:


<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        Пошук альбомів:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Завантаження...</h2>}>
        <div style={{ opacity: isStale ? 0.5 : 1 }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>Не знайдено збігів для <i>"{query}"</i></p>;
  }
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Not implemented');
  }
}

async function getSearchResults(query) {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  const allAlbums = [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

<Note>

Обидва затримані значення та [Переходи](#preventing-already-revealed-content-from-hiding) дозволяють вам уникнути показу запасного варіанту Затримки на користь вбудованих індикаторі. Перехід помічає оновлення цілком як не термінове тож вони часто використовуються фреймворками та навігаційними бібліотеками для навігації. Відкладені значення, з іншого боку, переважно використовуються у коді застосунку там, де ви хочете помітити частину UI як не термынову ы дозволити їй "відставати" від решти UI.

</Note>

---

### Preventing already revealed content from hiding {/*preventing-already-revealed-content-from-hiding*/}

### Запобігання ховання вже показаного змісту {/*preventing-already-revealed-content-from-hiding*/}

When a component suspends, the closest parent Suspense boundary switches to showing the fallback. This can lead to a jarring user experience if it was already displaying some content. Try pressing this button:

Коли компонент затримується, найближча батьківська границя Затримки перемикається на показ запасного варіанту. Це може призвести до неприэмного користувацького досвіду якщо якийсь вміст вже відображався. Спробуйте настинути цю кнопку:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { Suspense, useState } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    setPage(url);
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Завантаження...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        Музичний браузер
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Відкрити сторінку виконавця The Beatles
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Biography.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Panel.js hidden
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles were an English rock band, 
    formed in Liverpool in 1960, that comprised 
    John Lennon, Paul McCartney, George Harrison 
    and Ringo Starr.`;
}

async function getAlbums() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

When you pressed the button, the `Router` component rendered `ArtistPage` instead of `IndexPage`. A component inside `ArtistPage` suspended, so the closest Suspense boundary started showing the fallback. The closest Suspense boundary was near the root, so the whole site layout got replaced by `BigSpinner`.

Коли ви натиснули кнопку, компонент `Router` рендерить `ArtistPage` замість `IndexPage`. Компонент всередині `ArtistPage` затриманий, тож найближча границя Затримки почала відображати запасний варіант. Найближча границя затримки була біля корневого компонента, тому весь макет сайту було замінено на `BigSpinner`.

To prevent this, you can mark the navigation state update as a *Transition* with [`startTransition`:](/reference/react/startTransition)

Щоби запобігти цьому, ви можете помітити оновлення стану навігації як *Перехід* використовуючи [`startTransition`:](/reference/react/startTransition)

```js {5,7}
function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);      
    });
  }
  // ...
```

This tells React that the state transition is not urgent, and it's better to keep showing the previous page instead of hiding any already revealed content. Now clicking the button "waits" for the `Biography` to load:

Це говорить React що перехід стейту не є терміновим, і краще продовжити показувати попередню сторінку замість того щоб ховати вже відображений контент. Тепер натиск на кнопку "чекає" доки `Biography` завантажиться:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { Suspense, startTransition, useState } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Завантаження...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children }) {
  return (
    <div className="layout">
      <section className="header">
        Музичний Браузер
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Відкрити сторінку виконавця The Beatles
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Biography.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Panel.js hidden
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles were an English rock band, 
    formed in Liverpool in 1960, that comprised 
    John Lennon, Paul McCartney, George Harrison 
    and Ringo Starr.`;
}

async function getAlbums() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

A Transition doesn't wait for *all* content to load. It only waits long enough to avoid hiding already revealed content. For example, the website `Layout` was already revealed, so it would be bad to hide it behind a loading spinner. However, the nested `Suspense` boundary around `Albums` is new, so the Transition doesn't wait for it.

Перехід не чекає на завантаження *всього* змісту. Він лише чекає достатньо довго, щоби уникнути приховання вже відображеного змісту. Для прикладу, `Layout` вебсайту вже було відображено, тому було би погано ховати його за спіннером завантаження. Проте, вкладена границя `Suspense` навколо `Albums` нова, тому Перехід не чекає на неї.

<Note>

Suspense-enabled routers are expected to wrap the navigation updates into Transitions by default.

Раутери з ввімкненим Затримувачем очікують обгортання оновлень навігації в Переходи за замовчуванням.

</Note>

---

### Indicating that a Transition is happening {/*indicating-that-a-transition-is-happening*/}

### Сигналізація про те, що відбувається перехід {/*indicating-that-a-transition-is-happening*/}

In the above example, once you click the button, there is no visual indication that a navigation is in progress. To add an indicator, you can replace [`startTransition`](/reference/react/startTransition) with [`useTransition`](/reference/react/useTransition) which gives you a boolean `isPending` value. In the example below, it's used to change the website header styling while a Transition is happening:

У прикладі зверху, як тільки ви натискаєте на кнопку, відсутній візуальний сигнал про перехід навігації. Щоби додати індикатор, ви можете замінити [`startTransition`](/reference/react/startTransition) з [`useTransition`](/reference/react/useTransition) який дає вам булеве значення `isPending`. У прикладі знизу, воно використовується щоб змінити стилі хедеру вебсайту доки відбувається перехід:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js src/App.js
import { Suspense, useState, useTransition } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout isPending={isPending}>
      {content}
    </Layout>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```js src/Layout.js
export default function Layout({ children, isPending }) {
  return (
    <div className="layout">
      <section className="header" style={{
        opacity: isPending ? 0.7 : 1
      }}>
       Музичний Браузер
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

```js src/IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Відкрити сторінку виконавця The Beatles
    </button>
  );
}
```

```js src/ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

```js src/Albums.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Albums({ artistId }) {
  const albums = use(fetchData(`/${artistId}/albums`));
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Biography.js hidden
import { fetchData } from './data.js';

// Примітка: цей компонент створений з використанням експериментального API,
// який ще не доступний у стабільних версіях React.

// Для реалістичного прикладу, який ви можете спробувати сьогодні, скористайтеся фреймворком
// з інтегрованою Затримкою, таким як Relay чи Next.js

export default function Biography({ artistId }) {
  const bio = use(fetchData(`/${artistId}/bio`));
  return (
    <section>
      <p className="bio">{bio}</p>
    </section>
  );
}

// Це тимчасове рішення для виправлення помилки та запуску демонстрації.
// TODO: замінити реальною реалізацією коли помилку буде виправлено.
function use(promise) {
  if (promise.status === 'fulfilled') {
    return promise.value;
  } else if (promise.status === 'rejected') {
    throw promise.reason;
  } else if (promise.status === 'pending') {
    throw promise;
  } else {
    promise.status = 'pending';
    promise.then(
      result => {
        promise.status = 'fulfilled';
        promise.value = result;
      },
      reason => {
        promise.status = 'rejected';
        promise.reason = reason;
      },      
    );
    throw promise;
  }
}
```

```js src/Panel.js hidden
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

```js src/data.js hidden
// Примітка: те як ви будете отримувати дані залежить
// від фреймворку який ви використовуєте разом із Затримкою.
// Зазвичай, логіка кешування буде всередині фреймворку.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url === '/the-beatles/albums') {
    return await getAlbums();
  } else if (url === '/the-beatles/bio') {
    return await getBio();
  } else {
    throw Error('Not implemented');
  }
}

async function getBio() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 500);
  });

  return `The Beatles were an English rock band, 
    formed in Liverpool in 1960, that comprised 
    John Lennon, Paul McCartney, George Harrison 
    and Ringo Starr.`;
}

async function getAlbums() {
  // Додаємо штучну затримку щоб очікування було помітним.
  await new Promise(resolve => {
    setTimeout(resolve, 3000);
  });

  return [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];
}
```

```css
main {
  min-height: 200px;
  padding: 10px;
}

.layout {
  border: 1px solid black;
}

.header {
  background: #222;
  padding: 10px;
  text-align: center;
  color: white;
}

.bio { font-style: italic; }

.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-panel {
  border: 1px dashed #aaa;
  background: linear-gradient(90deg, rgba(221,221,221,1) 0%, rgba(255,255,255,1) 100%);
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}

.glimmer-line {
  display: block;
  width: 60%;
  height: 20px;
  margin: 10px;
  border-radius: 4px;
  background: #f0f0f0;
}
```

</Sandpack>

---

### Скидання меж Затримки при навігації {/*resetting-suspense-boundaries-on-navigation*/}

During a Transition, React will avoid hiding already revealed content. However, if you navigate to a route with different parameters, you might want to tell React it is *different* content. You can express this with a `key`:

Під час переходу, React уникне приховування вже відображеного контенту. Проте, якщо ви перейдете на раут з іншими параметрами, ви захочете сказати React що це *інший* вміст. Ви можете добитися цього з `key`:

```js
<ProfilePage key={queryParams.id} />
```

Imagine you're navigating within a user's profile page, and something suspends. If that update is wrapped in a Transition, it will not trigger the fallback for already visible content. That's the expected behavior.

Уявіть що ви переходите всередині сторінки профілю користувача, і щось затримується. Якщо те оновлення огорнуте в Перехід, воно вже не буде викликати запасний варіант для вже відображеного вмісту. Така поведінка є очікуваною.

However, now imagine you're navigating between two different user profiles. In that case, it makes sense to show the fallback. For example, one user's timeline is *different content* from another user's timeline. By specifying a `key`, you ensure that React treats different users' profiles as different components, and resets the Suspense boundaries during navigation. Suspense-integrated routers should do this automatically.

Але тепер уявіть, що ви переходите між профілями двох різних користувачів. В такому випадку, є сенс відображати запасний варіант. Наприклад, стрічка одного користувача має *інший вміст* ніж стрічка іншого користувача. Вказуючи `key`, ви запевнюєтеся, що React розглядає профілі різних користувачів як різні компоненти і скидає границю Затримки під час навігації. Раутери з інтегрованим Затримувачем повинні робити це автоматично.

---

### Providing a fallback for server errors and client-only content {/*providing-a-fallback-for-server-errors-and-client-only-content*/}

### Надання запасного варіанту для серверних помилок та контенту, що опрацьовується тільки на стороні клієнта {/*providing-a-fallback-for-server-errors-and-client-only-content*/}

If you use one of the [streaming server rendering APIs](/reference/react-dom/server) (or a framework that relies on them), React will also use your `<Suspense>` boundaries to handle errors on the server. If a component throws an error on the server, React will not abort the server render. Instead, it will find the closest `<Suspense>` component above it and include its fallback (such as a spinner) into the generated server HTML. The user will see a spinner at first.

Якщо ви використовуєте один з [API для потокового серверного рендерингу](/reference/react-dom/server) (або фреймворк що покладається на них), React також використовуватими вашу границю `<Suspense>` щоб обробляти помилки на стороні сервера. Якщо компонент викидує помилку на стороні сервера, React не відмінить серверний рендеринг. Натомість, він знайде найближчий компонент `<Suspense>` зверху і включить його запасний варіант (наприклад спіннер) у згенерований сервером HTML. Спочатку користувач побачить спіннер.

On the client, React will attempt to render the same component again. If it errors on the client too, React will throw the error and display the closest [error boundary.](/reference/react/Component#static-getderivedstatefromerror) However, if it does not error on the client, React will not display the error to the user since the content was eventually displayed successfully.

На стороні клієнта, React спробує відрендерити той же компонент знову. Якщо він видає помилку і на стороні клієнта, React викине помилку і відобразить найближчу [границю помилки.](/reference/react/Component#static-getderivedstatefromerror) Проте, якщо він не видає помилки на стороні клієнта, React не буде відображати користувачу помилку тому що вміст все ж був відображений коректно.

Ви можете використати це, щоб виключити деякі компоненти з рендерингу на сервері. Щоб зробити це, киньте помилку в серверному оточенні і обгорніть їх у границю `<Suspense>` щоби замінити їхній HTML запасним варіантом:

You can use this to opt out some components from rendering on the server. To do this, throw an error in the server environment and then wrap them in a `<Suspense>` boundary to replace their HTML with fallbacks:

```js
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>

function Chat() {
  if (typeof window === 'undefined') {
    throw Error('Chat should only render on the client.');
  }
  // ...
}
```

The server HTML will include the loading indicator. It will be replaced by the `Chat` component on the client.

Відрендерений на сервері HTML включатиме лише індикатор завантаження. Його буде замінено компонентом `Chat` на стороні клієнта.

---

## Усунення неполадок {/*troubleshooting*/}

### Як я можу запобігти заміні UI резервним варіантом під час оновлення {/*preventing-unwanted-fallbacks*/}

### How do I prevent the UI from being replaced by a fallback during an update? {/*preventing-unwanted-fallbacks*/}

Replacing visible UI with a fallback creates a jarring user experience. This can happen when an update causes a component to suspend, and the nearest Suspense boundary is already showing content to the user.

Заміна видимого UI запасним варіантом провокує неприємний досвід користувача. Це може статися коли оновлення провокує компонент затриматися, і найближчий Затримувач вже показує контент користувачу.

To prevent this from happening, [mark the update as non-urgent using `startTransition`](#preventing-already-revealed-content-from-hiding). During a Transition, React will wait until enough data has loaded to prevent an unwanted fallback from appearing:

Щоби запобігти цьому, [позначте оновлення не терміновим використовуючи `startTransition`](#preventing-already-revealed-content-from-hiding). Під час Переходу, React зачекає на завантаження даних щоб запобігти відображенню небажаного запасного варіанту:

```js {2-3,5}
function handleNextPageClick() {
  // If this update suspends, don't hide the already displayed content

  // Якщо це оновлення затримається, не ховайете вже відображений вміст
startTransition(() => {
    setCurrentPage(currentPage + 1);
  });
}
```

This will avoid hiding existing content. However, any newly rendered `Suspense` boundaries will still immediately display fallbacks to avoid blocking the UI and let the user see the content as it becomes available.

Це допоможе уникнтуи приховування вже існуючого вмісту. Проте, будь-яка наново відрендерена границя `Suspense` все ще негайно відображатиме запасний варіант щоби уникнути блокування UI і дозволить користувачу бачити вміст як тільки він стає доступним.

**React will only prevent unwanted fallbacks during non-urgent updates**. It will not delay a render if it's the result of an urgent update. You must opt in with an API like [`startTransition`](/reference/react/startTransition) or [`useDeferredValue`](/reference/react/useDeferredValue).

**React лише запобігатиме небажаним запасним варіантам під час не термінових оновлень**. Він не затримуватиме рендеринг якщо це результат термінового оновлення. Ви повинні вказати це з API таким як [`startTransition`](/reference/react/startTransition) або [`useDeferredValue`](/reference/react/useDeferredValue).

If your router is integrated with Suspense, it should wrap its updates into [`startTransition`](/reference/react/startTransition) automatically.

Якщо у ваш роутер інтегровано Затримувач, він повинен огортати оновлення у [`startTransition`](/reference/react/startTransition) автоматично.
