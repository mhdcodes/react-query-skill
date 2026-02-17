# Mastering Mutations in React Query

We've covered a lot of ground already when it comes to the features and concepts React Query provides. Most of them are about _retrieving_ data - via the `useQuery` hook. There is however a second, integral part to working with data: updating it.

For this use-case, React Query offers the `useMutation` hook.

## What are mutations?

Generally speaking, mutations are functions that have a side effect. As an example, have a look at the `push` method of Arrays: It has the side effect of _changing_ the array in place where you're pushing a value to:

```js:title=mutable-array-push
const myArray = [1]
myArray.push(2)

console.log(myArray) // [1, 2]
```

The _immutable_ counterpart would be `concat`, which can also add values to an array, but it will return a new Array instead of directly manipulating the Array you operate on:

```js:title=immutable-array-concat
const myArray = [1]
const newArray = myArray.concat(2)

console.log(myArray) //  [1]
console.log(newArray) // [1, 2]
```

As the name indicates, _useMutation_ also has some sort of side effect. Since we are in the context of [managing server state](react-query-as-a-state-manager) with React Query, mutations describe a function that performs such a side effect _on the server_. Creating a todo in your database would be a mutation. Logging in a user is also a classic mutation, because it performs the side effect of creating a token for the user.

In some aspects, `useMutation` is very similar to `useQuery`. In others, it is quite different.

## Similarities to useQuery

`useMutation` will track the state of a mutation, just like `useQuery` does for queries. It'll give you `loading`, `error` and `status` fields to make it easy for you to display what's going on to your users.

You'll also get the same nice callbacks that `useQuery` has: `onSuccess`, `onError` and ` onSettled`. But that's about where the similarities end.

## Differences to useQuery

<Highlight>useQuery is declarative, useMutation is imperative.</Highlight>

By that, I mean that queries mostly run automatically. You define the dependencies, but React Query takes care of running the query immediately, and then also performs smart background updates when deemed necessary. That works great for queries because we want to keep what we see on the screen _in sync_ with the actual data on the backend.

For mutations, that wouldn't work well. Imagine a new todo would be created every time you focus your browser window 🤨. So instead of running the mutation instantly, React Query gives you a function that you can invoke whenever you want to make the mutation:

```jsx:title=imperative-mutate
function AddComment({ id }) {
  // this doesn't really do anything yet
  const addComment = useMutation({
    mutationFn: (newComment) =>
      axios.post(`/posts/${id}/comments`, newComment),
  })

  return (
    <form
      onSubmit={(event) => {
        event.preventDefault()
        // ✅ mutation is invoked when the form is submitted
        addComment.mutate(
          new FormData(event.currentTarget).get('comment')
        )
      }}
    >
      <textarea name="comment" />
      <button type="submit">Comment</button>
    </form>
  )
}
```

Another difference is that mutations don't share state like `useQuery` does. You can invoke the same `useQuery` call multiple times in different components and will get the same, cached result returned to you - but this won't work for mutations.

## Tying mutations to queries

Mutations are, per design, not directly coupled to queries. A mutation that likes a blog post has no ties towards the query that fetches that blog post. For that to work, you would need some sort of underlying schema, which React Query doesn't have.

To have a mutation reflect the changes it made on our queries, React Query primarily offers two ways:

### Invalidation

This is conceptually the simplest way to get your screen up-to-date. Remember, with server state, you're only ever displaying a snapshot of data from a given point in time. React Query tries to keep that up-to-date of course, but if you're deliberately changing server state with a mutation, this is a great point in time to tell React Query that some data you have cached is now "invalid". React Query will then go and refetch that data if it's currently in use, and your screen will update automatically for you once the fetch is completed. The only thing you have to tell the library is _which_ queries you want to invalidate:

```jsx:title=invalidation-from-mutation {9-11}
const useAddComment = (id) => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (newComment) =>
      axios.post(`/posts/${id}/comments`, newComment),
    onSuccess: () => {
      // ✅ refetch the comments list for our blog post
      queryClient.invalidateQueries({
        queryKey: ['posts', id, 'comments']
      })
    },
  })
}
```

Query invalidation is pretty smart. Like all [Query Filters](https://react-query.tanstack.com/guides/filters#query-filters), it uses fuzzy matching on the query key. So if you have multiple keys for your comments list, they will all be invalidated. However, only the ones that are currently active will be refetched. The rest will be marked as stale, which will cause them to be refetched the next time they are used.

As an example, let's assume we have the option to sort our comments, and at the time the new comment was added, we have two queries with comments in our cache:

```
['posts', 5, 'comments', { sortBy: ['date', 'asc'] }
['posts', 5, 'comments', { sortBy: ['author', 'desc'] }
```

Since we're only displaying one of them on the screen, `invalidateQueries` will refetch that one and mark the other one as stale.

### Direct updates

Sometimes, you don't want to refetch data, especially if the mutation already returns everything you need to know. If you have a mutation that updates the title of your blog post, and the backend returns the complete blog post as a response, you can update the query cache directly via `setQueryData`:

```jsx:title=update-from-mutation-response
const useUpdateTitle = (id) => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (newTitle) =>
      axios
        .patch(`/posts/${id}`, { title: newTitle })
        .then((response) => response.data),
    // 💡 response of the mutation is passed to onSuccess
    onSuccess: (newPost) => {
      // ✅ update detail view directly
      queryClient.setQueryData(['posts', id], newPost)
    },
  })
}
```

Putting data into the cache directly via `setQueryData` will act as if this data was returned from the backend, which means that all components using that query will re-render accordingly.

I'm showing some more examples of direct updates and the combination of both approaches in [#8: Effective React Query Keys](effective-react-query-keys#structure).


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
