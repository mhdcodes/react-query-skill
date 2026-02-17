# Testing React Query

Questions around the testing topic come up quite often together with React Query, so I'll try to answer some of them here. I think one reason for that is that testing "smart" components (also called [container components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)) is not the easiest thing to do. With the rise of hooks, this split has been largely deprecated. It is now encouraged to consume hooks directly where you need them rather than doing a mostly arbitrary split and drill props down.

I think this is generally a very good improvement for colocation and code readability, but we now have more components that consume dependencies outside of "just props".

They might `useContext`. They might `useSelector`. Or they might `useQuery`.

Those components are technically no longer pure, because calling them in different environments leads to different results. When testing them, you need to carefully setup those surrounding environments to get things working.

## Mocking network requests

Since React Query is an async server state management library, your components will likely make requests to a backend. When testing, this backend is not available to actually deliver data, and even if, you likely don't want to make your tests dependent on that.

There are tons of articles out there on how to mock data with jest. You can mock your api client if you have one. You can mock fetch or axios directly. I can only second what Kent C. Dodds has written in his article [Stop mocking fetch](https://kentcdodds.com/blog/stop-mocking-fetch):

Use [mock service worker](https://mswjs.io/) by [@ApiMocking](https://twitter.com/ApiMocking)

It can be your single source of truth when it comes to mocking your apis:

- works in node for testing
- supports REST and GraphQL
- has a [storybook addon](https://storybook.js.org/addons/msw-storybook-addon) so you can write stories for your components that `useQuery`
- works in the browser for development purposes, and you'll still see the requests going out in the browser devtools
- works with cypress, similar to fixtures


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
