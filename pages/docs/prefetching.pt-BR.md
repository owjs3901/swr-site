# Pre-obtendo Dados

## Dados de Página Top-Level

Existem muitos jeitos de pre-obter os dados para SWR. Para requisições de página top-level, [`rel="preload"`](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) é altamente recomendado:

```html
<link rel="preload" href="/api/data" as="fetch" crossorigin="anonymous">
```

Basta apenas colocá-lo dentro do seu `<head>` do HTML. É fácil, rápido e nativo.

Irá pré-obter os dados quando o HTML carregar, antes mesmo de iniciar a baixar o JavaScript. Todos os seus pedidos de obtenção de dados com o mesmo URL vão usar o resultado (inclusive SWR, de modo que você pode usar o SWR para obter os dados de página top-level).

## Prefetching Programático

SWR provides the `preload` API to prefetch the resources programmatically and store the results in the cache. `preload` accepts `key` and `fetcher` as the arguments.

You can call `preload` even outside of React.

```jsx
import { useState } from 'react'
import useSWR, { preload } from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

// Preload the resource before rendering the User component below,
// this prevents potential waterfalls in your application.
// You can also start preloading when hovering the button or link, too.
preload('/api/user', fetcher)

function User() {
  const { data } = useSWR('/api/user', fetcher)
  ...
}

export default function App() {
  const [show, setShow] = useState(false)
  return (
    <div>
      <button onClick={() => setShow(true)}>Show User</button>
      {show ? <User /> : null}
    </div>
  )
}
```

Within React rendering tree, `preload` is also available to use in event handlers or effects.

```jsx
function App({ userId }) {
  const [show, setShow] = useState(false)

  // preload in effects
  useEffect(() => {
    preload('/api/user?id=' + userId, fetcher)
  }, [useId])

  return (
    <div>
      <button
        onClick={() => setShow(true)}
        {/* preload in event callbacks */}
        onHover={() => preload('/api/user?id=' + userId, fetcher)}
      >
        Show User
      </button>
      {show ? <User /> : null}
    </div>
  )
}
```

Junto com técnicas como [page prefetching](https://nextjs.org/docs/api-reference/next/router#routerprefetch) no Next.js, você vai ser capaz de carregar ambos a próxima página e os dados instantaneamente.

In Suspense mode, you should utilize `preload` to avoid waterfall problems.

```jsx
import useSWR, { preload } from 'swr'

// should call before rendering
preload('/api/user', fetcher);
preload('/api/movies', fetcher);

const Page = () => {
  // The below useSWR hooks will suspend the rendering, but the requests to `/api/user` and `/api/movies` have started by `preload` already,
  // so the waterfall problem doesn't happen.
  const { data: user } = useSWR('/api/user', fetcher, { suspense: true });
  const { data: movies } = useSWR('/api/movies', fetcher, { suspense: true });
  return (
    <div>
      <User user={user} />
      <Movies movies={movies} />
    </div>
  );
}
```

## Dados de Pré-preenchimento

Se você quiser preencher dados existentes no cache do SWR, você pode usar a opção `fallbackData`. Por exemplo:

```jsx
useSWR('/api/data', fetcher, { fallbackData: prefetchedData })
```

Se o SWR ainda não obtiver os dados, este hook retornará `prefetchedData` como um fallback.

Você pode também configurar isso para todos os hooks SWR e várias chaves com `<SWRConfig>` e a opção `fallback`. Veja [SSG e SSR com Next.js](/docs/with-nextjs) para mais detalhes.
