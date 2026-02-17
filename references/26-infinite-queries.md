# How Infinite Queries work

This week, a very interesting [bug report](https://github.com/TanStack/query/issues/8046) was filed for [Infinite Queries](https://tanstack.com/query/v5/docs/framework/react/guides/infinite-queries) in React Query. It was interesting because up to this point, I firmly believed that React Query doesn't have any bugs.

Okay, not really, but I was pretty sure it doesn't have any bugs that would a) affect a large number of users and b) would be because of some architectural constraint in the library itself.

We do of course have edge-case bugs for quite specific situations that need workarounds (can't really live without those) and also some known limitations that might be annoying to accept, for example, that suspense is not working with query cancellation.

But this bug report hit different. It was <Emph>obviously wrong behavior</Emph>. We also didn't regress here - it has always worked this way. It could still be classified as an edge case, because for it to happen, you would need to:

- Have an Infinite Query that has already once successfully fetched multiple pages.
- Have a refetch where fetching at least one page succeeded, but then the next page failed to fetch.
- Use at least one retry (default is three).

This likely won't hit you every day, but it also isn't a huge edge-case. I was surprised that in the last four years, no one has reported this. So I asked on twitter and it seems like users have been getting this bug in the past, but also didn't think React Query would have such a huge flaw and thus didn't report it. Seems like we're at least all aligned on the overall quality in React Query. 🙌


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
