@title[Title]

# Elm at Shore

Christoph

Shore GmbH

github: ChristophP

---
@title[Where were we]

### Where were we?

![LOGO](assets/js-frameworks.png)

Rails Backend + Server rendered pages

- Bad Scalabilty
- Prone to become legacy

+++

JS hard to scale, refactor.

=> Elm seems to allow clean adaptions to business requirements.

---

How could switch to Elm?

+++

We need to be compatible with
- I18Next
- MomentJS

---

This can't be a new problem

JSON files
```json
{
  "elm": {
    "munich": {
      "greeting": "Hello"
    }
  }
}
```

+++
@title[Existing solutions]

Pre-build solution: [elm-i18n](https://github.com/iosphere/elm-i18n)

- All locales built into js bundle |
- only PO/CSV files |

---

### First attempt at solving it

Link npm `elm-i18n-gen`

Pre-build translation into Elm code as functions.

+++

[elm-i18n-gen](https://github.com/ChristophP/elm-i18n-gen)

Supports JSON

```json
{
  "hello": "Hello",
  "gooddaySalute": "Good Day {{name}} {{assi}}",
  "tigers": {
    "roar": "Roar!"
  }
}

{
  "hello": "Hallo",
  "gooddaySalute": "Guten Tag {{name}} {{assi}}",
  "tigers": {
    "roar": "Brüll!"
  }
}
```

+++

To this
```elm
module Translations exposing (..)

type Lang
  =  De
  |  En

getLnFromCode: String -> Lang
getLnFromCode code =
   case code of
      "de" -> De
      "en" -> En
      _ -> En

hello: Lang -> String
hello lang  =
  case lang of
      De -> "Hallo"
      En -> "Hello"

gooddaySalute: Lang -> String -> String -> String
gooddaySalute lang str0 str1 =
  case lang of
      De -> "Guten Tag " ++ str0 ++ " " ++ str1 ++ ""
      En -> "Good Day " ++ str0 ++ " " ++ str1 ++ ""

tigersRoar: Lang -> String
tigersRoar lang  =
  case lang of
      De -> "Brüll!"
      En -> "Roar!"
```

+++

Not nice to have all in one bundle.

What about runtime?

---
@title[Now at runtime]

Second attempt: Load and fetch + decode

[elm-i18next](http://package.elm-lang.org/packages/ChristophP/elm-i18next/latest)

+++

```elm
init: Json.Encode.Value -> (Model, Cmd msg)
init flags =
  case Json.Decode.decodeValue decodeTranslations flags of
    Ok translations ->
      ({ model | translations = translations }, Cmd.none)
    Err err ->
      ... -- handle the error or use `Result.withDefault`
```

+++

### Recursive decoder

```elm
type Tree
    = Branch (Dict String Tree)
    | Leaf String

decodeTree : Decoder Tree
decodeTree =
    Decode.oneOf
        [ Decode.string |> Decode.map Leaf
        , Decode.lazy
            (\_ -> Decode.dict decodeTree
               |> Decode.map Branch)
        ]
```
@[1-3]
@[5-12]

---

### Downsides

- no more typesafety

### Where are we now?

Solved translations and momentJS problem with Web components.

=> Elm code now over 50 %
=> very stable and changeable code
