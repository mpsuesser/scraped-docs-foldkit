---
url: https://foldkit.dev/api-reference/file
title: "File"
description: "API documentation for the File module."
access_date: 2026-08-11T03:02:29.236Z
current_date: 2026-08-11T03:02:29.236Z
---

# File

## Functions

### mimeType

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/file.ts#L28)

```
/** The file's MIME type (e.g. `"application/pdf"`), or empty string if the browser cannot determine one. */
(file: File): string
```

### name

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/file.ts#L22)

```
/** The file's name including extension, as reported by the browser. */
(file: File): string
```

### readAsArrayBuffer

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/reader.ts#L117)

```
/**
 * Reads the file's contents as raw binary data. Mirrors Elm's `File.toBytes`.
 * 
 * Use this when you need the full binary payload (e.g. to upload, hash, or
 * parse a custom file format). For images you usually want `readAsDataUrl`
 * instead.
 */
(file: File): Effect<ArrayBuffer, FileReadError>
```

### readAsDataUrl

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/reader.ts#L92)

```
/**
 * Reads the file's contents as a base64-encoded data URL. Useful for rendering
 * image previews without uploading the file first. Mirrors Elm's `File.toUrl`.
 */
(file: File): Effect<string, FileReadError>
```

### readAsText

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/reader.ts#L73)

```
/**
 * Reads the file's contents as a UTF-8 string. Mirrors Elm's `File.toString`.
 * 
 * Fails with a `FileReadError` if the browser's `FileReader` encounters an
 * error (e.g. the file was deleted while reading). Handle failures with
 * `Effect.catch` to convert them into a failure Message.
 */
(file: File): Effect<string, FileReadError>
```

### select

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/select.ts#L68)

```
/**
 * Opens the native file picker allowing a single file to be selected. Resolves
 * with `Option.some(file)` if the user picked one, or `Option.none()` if they
 * cancelled. Mirrors Elm's `File.Select.file`.
 * 
 * The `accept` argument is a list of MIME types or file extensions that
 * restrict what the picker shows. Pass an empty array to allow any file.
 */
(accept: readonly Array<string>): Effect<Option<File>>
```

### selectMultiple

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/select.ts#L87)

```
/**
 * Opens the native file picker allowing multiple files to be selected at
 * once. Resolves with the array of selected files, or an empty array if the
 * user cancelled. Mirrors Elm's `File.Select.files`.
 */
(accept: readonly Array<string>): Effect<readonly Array<File>>
```

### size

function

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/file.ts#L25)

```
/** The file's size in bytes. */
(file: File): number
```

## Types

### File

type

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/file.ts#L9)

```
/**
 * A file selected by the user. Direct alias for the browser's native `File`
 * type — opaque by convention (Foldkit never constructs files itself, only
 * receives them from `File.select`, `File.selectMultiple`, or from
 * `OnFileChange`/`OnDropFiles` event attributes).
 */
type File = globalThis.File
```

## Constants

### File

const

[source](https://github.com/foldkit/foldkit/blob/6d495f0d10e1576968c413e1d1fd7287a670d8ea/packages/foldkit/src/file/file.ts#L9)

```
/**
 * Schema that accepts any value that is an instance of the DOM `File` class.
 * Use in Model fields that hold user-selected files:
 * 
 * ```ts
 * attachedResume: S.Option(File.File)
 * ```
 */
const File: Schema<File>
```
