# Angular Template IntelliSense Missing on Nullable Union Types

## Problem

In Angular templates inside Zed, IntelliSense/autocomplete may fail when accessing properties on values with nullable union types.

Example:

```ts
type User = {
  name: string;
};

readonly user = signal<User | undefined>(undefined);
```

Template:

```angular
@let currentUser = user();

{{ currentUser. }}
```

No autocomplete appears after:

```html
currentUser.
```

However, autocomplete works when using optional chaining:

```html
currentUser?.
```

---

## Symptoms

* No IntelliSense suggestions after typing `.`
* Angular template autocomplete appears broken
* TypeScript files still work normally
* IntelliSense suddenly works when switching to `?.`

---

## Cause

The value has a nullable union type such as:

```ts
User | undefined
```

or

```ts
User | null
```

TypeScript considers direct property access unsafe because the value may not exist.

Unsafe access:

```html
currentUser.name
```

Safe access:

```html
currentUser?.name
```

Angular template language services often rely on safe narrowing before offering property completions.

As a result, IntelliSense may not appear until optional chaining is used.

---

## Solution

Use optional chaining for nullable union types:

```angular
{{ currentUser?.name }}
```

This:

* restores IntelliSense
* satisfies template type safety
* prevents runtime access errors

---

## Alternative Solution

If the value is guaranteed to exist, use a non-null assertion:

```angular
{{ currentUser!.name }}
```

However, this bypasses null safety and should only be used when existence is guaranteed.

---

## Why This Is Confusing

This often appears to be an editor or Angular tooling issue because:

* autocomplete works in regular TypeScript files
* Angular templates fail silently
* only nullable values are affected
* optional chaining immediately restores suggestions

This can lead developers to incorrectly assume:

* the Angular language server is broken
* Zed lacks Angular support
* template IntelliSense is malfunctioning

when the actual issue is nullable type safety.

---

## Common Nullable Union Types

Examples that commonly trigger this behavior:

```ts 
User | undefined
Project | undefined
Data | null
Response | null | undefined
```

---

## Recommended Configuration

Enable strict Angular template checking:

```json
{
  "angularCompilerOptions": {
    "strictTemplates": true
  }
}
```

Install Angular language service:

```bash
npm install -D @angular/language-service
```

---

## Tested On

* Angular 21+
* TypeScript strict mode
* Angular Signals
* `@let` template syntax
* Zed v1.2.5
* Linux(Pop!_OS)