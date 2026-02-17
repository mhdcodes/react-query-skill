# Concurrent Optimistic Updates in React Query

[Optimistic Updates](https://tanstack.com/query/v5/docs/framework/react/guides/optimistic-updates) are one of those techniques that are great to [make your app feel faster than it really is](https://www.youtube.com/watch?v=nxzVZ7FYdwE&t=651s), but it's also one of the things that's easiest in todo-app style demos.

Look, I can instantly append a task to a list when I press `Enter` on the input field. That's great in theory, but in practice, there's likely more challenges awaiting you.

## Re-creating server logic on the client

I have already written a bit about this in [#12: Mastering Mutation in React Query](mastering-mutations-in-react-query#optimistic-updates), but it's an important point to re-iterate on. With optimistic UI, you're essentially trying to foresee what the server will do, and implement that on the client beforehand.

This can be quite straight-forward, for example, if the user is interacting with a toggle button. Usually, all we need to do is take the current boolean state and invert it:

```ts:title=toggle-isActive
queryClient.setQueryData(['items', 'detail', item.id], (prevItem) =>
  prevItem
    ? {
        ...prevItem,
        isActive: !prevItem.isActive,
      }
    : undefined
)
```

That's not too much code to write, and it has a great effect on the UX: Rather than having to wait until the request is finished before the UI reacts, our users can now click the button and it instantly see their change reflected. There's not much that's worse than a toggle button that slides to the new state half a second after I clicked it. 😅

### More complex update logic

In other situations, it's not that easy, and we don't have to invent a complex scenario to get there. Let's just assume we have a list of items that have a category, which our users can use for filtering. When they edit an item, we show that in a modal dialog, and once they're done, we want to optimistically update the list they are currently seeing.

Finding the item in the list of items is not the problem, and merging the updates is also not that bad:

```ts:title=update-item-in-list
queryClient.setQueryData(['items', 'list', filters], (prevItems) =>
  prevItems?.map((item) =>
    item.id === newItem.id ? { ...item, ...newItem } : item
  )
)
```

This, too, works great, until we realize the edge-case when a user updates a category of our item, which would <Emph>remove</Emph> the item from the current filter. We aren't handling that yet. Even the GitHub list view gets that wrong when we're using inline editing to remove a `label` that we have currently filtered for. Let's fix that:

```ts:title=filter-wrong-categories {6}
queryClient.setQueryData(['items', 'list', filters], (prevItems) =>
  prevItems
    ?.map((item) =>
      item.id === newItem.id ? { ...item, ...newItem } : item
    )
    .filter((item) => filters.categories.includes(item.category))
)
```

Now, the item optimistically disappears, which is what we'd want, because that's what the server would do when the list gets refetched. We just forgot that we can also filter by text. Now we need to do the same thing for `item.title` and ...


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
