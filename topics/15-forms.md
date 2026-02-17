# React Query and Forms


This approach can get problematic on bigger forms and in collaborative environments. The bigger the form, the longer it takes our users to fill it out. If multiple people work on the same form, but on different fields, whoever updates last might override the values that others have changed, because they still see a partially outdated version on their screen.

Now react hook form allows you to detect which fields have been changed by the user and only send "dirty" fields to the server with some user land code (see [the example here](https://codesandbox.io/s/react-hook-form-submit-only-dirty-fields-ol5d2)), which is pretty cool. However, this still doesn't show the latest values with updates made by other users to you. Maybe you would change your input had you known that a certain field was changed in the meantime by someone else.

So what would we need to do to still reflect background updates while we are editing our form?

## Keeping background updates on

One approach is to rigorously separate the states. We'll keep the Server State in React Query, and only track the changes the user has made with our Client State. The source of truth that we display then to our users is _derived state_ from those two: If the user has changed a field, we show the Client State. If not, we fall back to the Server State:

```js:title=separate-states {20-25,35-38}
function PersonDetail({ id }) {
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
  })
  const { control, handleSubmit } = useForm()
  const { mutate } = useMutation({
    mutationFn: (values) => updatePerson(values),
  })

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
        <div>
          <label htmlFor="firstName">First Name</label>
          <Controller
            name="firstName"
            control={control}
            render={({ field }) => (
              // ✅ derive state from field value (client state)
              // and data (server state)
              <input
                {...field}
                value={field.value ?? data.firstName}
              />
            )}
          />
        </div>
        <div>
          <label htmlFor="lastName">Last Name</label>
          <Controller
            name="lastName"
            control={control}
            render={({ field }) => (
              <input
                {...field}
                value={field.value ?? data.lastName}
              />
            )}
          />
        </div>
        <input type="submit" />
      </form>
    )
  }

  return 'loading...'
}
```

With that approach, we can keep background updates on, because it will still be relevant for untouched fields. We are no longer bound to the initialState that we had when we first rendered the form. As always, there are caveats here as well:

### You need controlled fields

As far as I'm aware, there is no good way to achieve this with uncontrolled fields, which is why I've resorted to using controlled fields in the above example. Please let me know if I'm missing something.

### Deriving state might be difficult

This approach works best for shallow forms, where you can easily fall back to the Server State using nullish coalesce, but it could be more difficult to merge properly with nested objects. It might also sometimes be a questionable user experience to just change form values in the background. A better idea might be to just highlight values that are out of sync with the Server State and let the user decide what to do.


That's it for today. Feel free to reach out to me on [bluesky](https://bsky.app/profile/tkdodo.eu)
if you have any questions, or just leave a comment below. ⬇️
