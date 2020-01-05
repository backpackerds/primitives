# Anatomy of a UI element

1. [How things can get messy](#how-things-can-get-messy)
2. [Building primitives](#building-primitives)
3. [Using primitives](#using-primitives)

## How things can get messy

I want to build a button.

> [insert img1]

I ask for the design, take a quick look and I start coding something along these lines:

```js
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

const Button = (props) => {
  const { label, onPress } = props;

  return (
    <TouchableOpacity style={styles.base} onPress={onPress}>
      <Text style={styles.text}>{label}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  base: {
    backgroundColor: '#28D9AA', // green
    height: 40,
    paddingHorizontal: 20,
    paddingVertical: 5
  },
  text: {
    fontFamily: 'Handvetica',
    fontSize: 18,
    textTransform: 'uppercase',
    color: '#FFF' // white
  }
});
```

It's all good, the component looks exactly like the designer wanted it (let's suppose the intended background was indeed a shade of **green**) It takes a `label` string and an `onPress` function as `props`, so it can be reused throughout my app. I rename it to `DefaultButton` and put it in `src/components`. By convention, the `components` dir holds the reusable UI for my app.

```js
// src/components/DefaultButton
const DefaultButton = (props) => {...}
```

Now I can use it anywhere like this:

```js
import DefaultButton from './components/DefaultButton';

<DefaultButton label='Login' onPress={login} />;
```

On some other screen, I have another button that has a **purple** background color (instead of my _default_ **green**). No big deal, I just change the `DefaultButton` so it accepts passing a `baseStyle` through `props`.

```js
// src/components/DefaultButton
const DefaultButton = (props) => {
  const { label, onPress, baseStyle } = props;
  //                         ☝️
  const baseStyle = [styles.base, baseStyle];
  return (
    <TouchableOpacity style={baseStyle} onPress={onPress}>
      <Text style={styles.text}>{label}</Text>
    </TouchableOpacity>
  );
};
```

Now I just have to update the `baseStyle` on that particular screen. If it happens to be used in another place, I should probably move the styling to a `SecondaryButton` component and use that in all places. But it's okay for now.

```js
import DefaultButton from './components/DefaultButton';

<DefaultButton
  label='Edit profile'
  onPress={editProfile}
  baseStyle={{ backgroundColor: '#8300bf' }} // purple
/>;
```

A couple of days pass and when starting the work on a new app screen, I observe that the designer added a little arrow along the text of the _default_ **green** button.

> [insert img2]

I go ahead and update the `DefaultButton` component.

```js
// src/components/DefaultButton
const DefaultButton = (props) => {
  const { label, onPress, baseStyle, rightIcon } = props;
  //                                     ☝️
  return (
    <TouchableOpacity style={baseStyle} onPress={onPress}>
      <Text style={styles.text}>{label}</Text>
      { rightIcon && <Image source={rightIcon} /> }
    </TouchableOpacity>
  )
}

const styles = StyleSheet.create({
  base: {
    flexDirection: 'row'
    ...
  },
})
```

Now, let's complicate the example so it feels more like a real life situation and less like a proof of concept. I've got some more button layouts:

- _icon_ on the left, _label_ right next to it
- _icon_ on top, _label_ below
- _subtitle_ underneath the _label_
- full-width button with the _subtitle_ right next to the _label_

It becomes really hard to fit all this new logic into my component and keep it readable (and also usable!) at the same time. And as Donald Knuth once said,

> Programs are meant to be read by humans and only incidentally for computers to execute. (The Art of Computer Programming, 1968)

One solution is to split the button into multiple components, one for each button type - `DefaultButton`, `SecondaryButton`, `ColumnButton`, `FullWidthButton`. And that works. But upon a closer look, I can identify a couple of issues with this approach.

1. All the components above share design and functionality (not in terms of code, but rather shared as in common design and functionality). If at some point I have to change a part of this shared look or behavior, I get into trouble again. To switch from `TouchableOpacity` to `TouchableHighlight`, for example, or change the `activeOpacity` value, I have to go in and change all my components.

2. A colleague of mine has to build another button that kind of looks like the ones we built so far, but not _exactly_ like that. He goes ahead and builds another component, from scratch, trying to replicate the shared look and behavior, but also adding the new functionality in.

The point is, as you might see, it can get pretty messy and inefficient. And it's just a button.

Now, let's first make the simple things _not_ messy, following the the [guiding principles](https://github.com/backpackerds/readme#guiding-principles). A UI component must be:

- consistent
- composable
- reusable

## Building primitives

- deconstruct
- the concept of a base primitive
- building on top of base

First, deconstruct to construct. Take a look at the fist button again.

## Using primitives

- shift in thinking
- thinking of what i want to achieve, not how