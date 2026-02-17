# React Query as a State Manager

React Query is loved by many for drastically simplifying data fetching in React applications. So it might come as a bit of a surprise if I tell you that React Query is in fact _NOT_ a data fetching library.

It doesn't fetch any data for you, and only a very small set of features are directly tied to the network (like [the OnlineManager](https://react-query.tanstack.com/reference/onlineManager), `refetchOnReconnect` or [retrying offline mutation](https://react-query.tanstack.com/guides/mutations#retry)). This also becomes apparent when you write your first `queryFn`, and you have to use _something_ to actually get the data, like [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), [axios](https://axios-http.com/), [ky](https://github.com/sindresorhus/ky) or even [graphql-request](https://github.com/prisma-labs/graphql-request).

So if React Query is no data fetching library, what is it?

## An Async State Manager

React Query is an async state manager. It can manage any form of asynchronous state - it is happy as long as it gets a Promise. Yes, most of the time, we produce Promises via data fetching, so that's where it shines. But it does more than just handling loading and error states for you. It is a proper, real, "global state manager". The `QueryKey` uniquely identifies your query, so as long you call the query with the same key in two different places, they will get the same data. This can be best abstracted with a custom hook so that we don't have to access the actual data fetching function twice:

```tsx:title=async-state-manager
export const useTodos = () =>
  useQuery({ queryKey: ['todos'], queryFn: fetchTodos })

function ComponentOne() {
  const { data } = useTodos()
}

function ComponentTwo() {
  // ✅ will get exactly the same data as ComponentOne
  const { data } = useTodos()
}

const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ComponentOne />
      <ComponentTwo />
    </QueryClientProvider>
  )
}
```

Those components can be _anywhere_ in your component tree. As long as they are under the same `QueryClientProvider`, they will get the same data.
React Query will also _deduplicate_ requests that would happen at the same time, so in the above scenario, even though two components request the same data, there will be only one network request.

## A data synchronization tool

Because React Query manages async state (or, in terms of data fetching: server state), it assumes that the frontend application doesn't "own" the data. And that's totally right. If we display data on the screen that we fetch from an API, we only display a "snapshot" of that data - the version of how it looked when we retrieved it. So the question we have to ask ourselves is:

Is that data still accurate after we fetch it?

The answer depends totally on our problem domain. If we fetch a Twitter post with all its likes and comments, it is likely outdated (stale) pretty fast. If we fetch exchange rates that update on a daily basis, well, our data is going to be quite accurate for some time even without refetching.

React Query provides the means to _synchronize_ our view with the actual data owner - the backend. And by doing so, it errs on the side of updating often rather than not updating often enough.

### Before React Query

Two approaches to data fetching were pretty common before libraries like React Query came to the rescue:

- fetch once, distribute globally, rarely update
  This is pretty much what I myself have been doing with redux a lot. Somewhere, I dispatch an action that initiates the data fetching, usually on mount of the application. After we get the data, we put it in a global state manager so that we can access it everywhere in our application. After all, many components need access to our Todo list.
  Do we refetch that data? No, we have "downloaded" it, so we have it already, why should we? Maybe if we fire a POST request to the backend, it will be kind enough to give us the "latest" state back. If you want something more accurate, you can always reload your browser window...

- fetch on every mount, keep it local
  Sometimes, we might also think that putting data in global state is "too much". We only need it in this Modal Dialog, so why not fetch it _just in time_ when the Dialog opens. You know the drill: `useEffect`, empty dependency array (throw an eslint-disable at it if it screams), `setLoading(true)` and so on ... Of course, we now show a loading spinner every time the Dialog opens until we have the data. What else can we do, the local state is gone...


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
