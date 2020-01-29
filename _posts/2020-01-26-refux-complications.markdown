---
layout: post
title: "Global state management is a bit of a bummer"
date: 2020-01-26 09:52:00 -0500
permalink: "/bummertown-in-redux-land/"
---

I recently had the opportunity to work on a medium complexity React Native app.
We ultimately decided to use Redux for a few of the more complex flows in the
app - specifically around a particularly large wizard type of flow.

It took a minute to fully grok, but the project ended up leaving me with a
fairly sour taste in my mouth. It felt like a large amount of boilerplate,
obfuscation, and added state for limited benefit.

## The use case:

As mentioned above, the user flow that ended up using Redux the most was a long
drawn out wizard flow - i.e. the user started on screen A, then progressed to
screen B, C, D and so on and finally submitted a thing to an API. The thing
isn't particularly important, all that matters is that we needed the results of
all of the previous screens before we got to the final screen.

To make it a bit more concrete, let's say we were building a login flow with 4
screens - a screen for the user to enter their personal information, another
screen for them to enter their email address, a screen to enter their
credit card information, and finally a screen to do the final thing and
submit all of that jazz so the user can finally get to the app and enter their
todos or whatever.

## The non Redux way:

If we're not using Redux, the code is pretty simple. You can imagine a bunch of
models:

```typescript
interface PersonalInformation {
    name: string
    address: Address
}

interface Email {
    value: string
}

interface CreditCard {
    number: number
    securityCode: number
}
```

And a bunch of components that slowly build up the relevant information.

Here's the personal information screen:

```typescript
const PersonalInformationScreen = () => {
    const [name, setName] = useState<string | undefined>(undefined)
    const [address, setAddress] = useState<Address | undefined>(undefined)
    const onNextClicked = () => {
        if (name && address) {
            navigateToEmailScreen(name, address)
        } else {
            tellThemToGiveUsTheirPreciousData()
        }
    }

    return ...
}
```

And the email screen:

```typescript
interface EmailScreenProps {
    name: string
    address: Address
}

const EmailScreen = ({name, address}: EmailScreenProps) => {
    const [email, setEmail] = useState<Email | undefined>(undefined)
    const onNextClicked = () => {
        if (email) {
            navigateToFinalScreen(name, address, email)
        } else {
            tellThemToSignUpForOurSpam()
        }
    }

    return ...
}
```

The credit card page would look just about the same so I'll omit it.

Finally the whole flow culminates in the screen that actually does the
submission:

```typescript
interface SubmitScreenProps {
    name: string
    address: Address
    email: Email
    creditCard: CreditCard
}

const SubmitScreen = ({name, address, email, creditCard}: SubmitScreenProps) => {
    const onSubmitClicked = () => {
        postMyGarbage(name, address, email, creditCard)
    }

    return ...
}
```
There's a bit of noise since we're passing argument through as we go, and these
components may know more than we might like, but it's pretty simple to
read through and we know we have all of the data once we get to this final stage.

## The Redux way:

Redux can theoretically simplify this flow by wrapping all of the state up in
the store so individual components don't need to worry about it and the
components can become a touch less noisy.

Our models stay the same:

```typescript
interface PersonalInformation {
    name: string
    address: Address
}

interface Email {
    value: string
}

interface CreditCard {
    number: number
    securityCode: number
}
```

And we now have some chunk of state in the store that needs to be modelled:

```typescript
interface WizardState {
    name: string | null
    address: Address | null
    email: string | null
    ccNumber: number | null
    ccSecurityCode: number | null
}

const initialWizardState: WizardState = {}
```

All of our fields are nullable, since we need some initial state and before the
user has started going through the flow we don't have any data. We could provide
a bunch of default strings and numbers, but that would effectively obfuscate the
fact that we have no data.

Our components now become simpler, since we don't need to pass so much
data down the component chain:

```typescript
const PersonalInformationScreen = () => {
    const [name, setName] = useState<string | undefined>(undefined)
    const [address, setAddress] = useState<Address | undefined>(undefined)
    const dispatch = useDispatch()
    const onNextClicked = () => {
        if (name && address) {
            dispatch(personalInformationAdded(name, address)
            navigateToEmailScreen(name, address)
        } else {
            tellThemToGiveUsTheirPreciousData()
        }
    }

    return ...
}
```

The email screen gets a lot less noisy since we don't need to worry about the
arguments being passed in anymore and thus have no props:

```typescript
const EmailScreen = () => {
    const [email, setEmail] = useState<Email | undefined>(undefined)
    const dispatch = useDispatch()
    const onNextClicked = () => {
        if (email) {
            dispatch(emailAdded(email))
            navigateToFinalScreen(name, address, email)
        } else {
            tellThemToSignUpForOurSpam()
        }
    }

    return ...
}
```

And theoretically the final submission page becomes similarly simple, since
again we can omit the props:

```typescript
const SubmitScreen = () => {
    const wizardState = useSelector(state => state.wizardState)
    const onSubmitClicked = () => {
        const {name, address, email, creditCard} = wizardState;
        postMyGarbage(name, address, email, creditCard)
    }

    return ...
}
```

But there's actually some weirdness in the above code. Specifically, these two
lines:

```
const {name, address, email, creditCard} = wizardState;
postMyGarbage(name, address, email, creditCard)
```

Recall that our wizard state defines all of the necessary state fields as
nullable. So assuming we've got the appropriate strictness settings on,
typescript should yell at us because we're not handling the fact that all of
those fields could be nullable.

And therein lies the problem I have with Redux (or really any global state
management system). All of the fields are nullable because we need some initial
state to keep in the store, and we now need to handle that nullability
everywhere else. We've effectively introduced nonsensical states into our
system. What does it mean to be in the final screen of a wizard flow and not
have any of the data?

In reality, the above component would _probably_ look something like this:

```typescript
const SubmitScreen = () => {
    const wizardState = useSelector(state => state.wizardState)
    const {name, address, email, creditCard} = wizardState;
    if (!name || !address || !email || !creditCard) {
        return null;
    } else {
        const onSubmitClicked = () => {
            postMyGarbage(name, address, email, creditCard)
        }

        return ...
    }
}
```

But the damage is done. We're now explicitly allowing error states in our type
system, and we're relying on the programmer to know not to use a component in
the wrong place.

That sort of implicit coupling feels _wrong_. 

In our non Redux flow, you could never get into this state. Since `SubmitScreen`
took in all of its necessary data as props, there was no way to build it without
the required data.

And that's my main problem with Redux.

I imagine Redux is super useful if you have _really, really_ complicated state,
but when you're just trying to share data between components it feels kind of
bad.

