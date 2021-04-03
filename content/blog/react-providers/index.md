---
title: React Providers
date: '2021-05-01T24:00:00Z'
description: ''
lang: 'en'
path: 'react-providers'
draft: true
---

I have written and seen code like this in React apps:

```jsx
//...

function App() {

//...

  return (
    <AuthProvider>
      <ReduxProvider store={store}>
        <ThemeProvider theme={theme}>
          <IntlProvider locale={locale}>
            <Router>
              <Switch>
                <Route path="/about">
                    <About />
                </Route>
                <Route path="/">
                    <Home />
                </Route>
              </Switch>
            </Router>
          </IntlProvider>
        </ThemeProvider>
      </ReduxProvider>
    </AuthProvider>
  );
}
```

Nothing inherently wrong with this, but it can be better. For example, if we want to use Storybook, and we probably should, then it is very, very likely that many React components will need some, if not all, of these providers. Most likely the IntlProvider and ThemeProvider, maybe even the ReduxProvider. Also, we may need them for the unit tests of the React components. If some components use React hooks that access a context, then is a guarantee that a provider is needed.

## How to make it better

Since almost everything in React is a component, let's make a new one to centralize all the providers in the app. It would look like this:

```jsx
// file: AppProvider.tsx

const AppProvider = ({ children, store, theme, locale }) => {
  return (
    <AuthProvider>
      <ReduxProvider store={store}>
        <ThemeProvider theme={theme}>
          <IntlProvider locale={locale}>
            {children}
          </IntlProvider>
        </ThemeProvider>
      </ReduxProvider>
    </AuthProvider>
  );
};

export default AppProvider;

// ------------------------- //

// file: App.tsx

import AppProvider from './AppProvider';

function App() {
  
  //...

  return (
    <AppProvider store={store} theme={theme} locale={locale}>
      <Router>
        <Switch>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/">
            <Home />
          </Route>
        </Switch>
      </Router>
    </AppProvider>
  );
}
```

The AppProvider component centralizes all global providers of the app and takes props needed for any of them because we should be able to pass different values in other contexts. Now if we want to have all the providers available in Storybook, for example, we could do this:

```jsx
// file: SomeComponent.stories.js

export default {
  component: SomeComponent,
  decorators: [
    (Story) => (
      <AppProvider store={store} theme={theme} locale={locale}>
        <Story />
      </AppProvider>
    ),
  ],
};

```

The advantages of this approach are:

- Single source of truth for providers, if they need to be modified, the change is done in one place. It also helps with maintainability.
- Reusability
- Easier to read. When we go through the App file, we focus on what's important. When we see the AppProvider component, we get that it gives the app some shared state and capabilities. If we need to dig further, it's just to open that file and focus only on the providers. In my experience, minor readability optimizations like this one help to avoid unnecessary mental gymnastics when reading code. Most likely, the code base will grow, and readability and simplicity will be of huge importance for maintenance.

## Caveats

As professionals, a big part of our job is to use our judgment and experience to make good decisions. So, we have to exercise that. If some providers cause more complications than solutions, then don't include it in the AppProvider component. One example, most likely something like a AuthProvider is not needed in Storybook or on the tests. Maybe it is better not to include it in the AppProvider and keep it in the App file. It is something to be decided on a case-by-case.

I hope this was useful to you.

If you have any recommendations or suggestions, let me know in the comments.

Cheers!