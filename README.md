# RevHttpFs

The [afero](github.com/spf13/afero) golang library already has an `afero.HttpFs` which goes from an afero Fs to `http.FileSystem`. This package implements the reverse,
allowing use of an `http.FileSystem` as an `afero.Fs`.

## Motivation

The reason for this is that many tools expose `http.FileSystem` interfaces, since it is the standard in golang for serving files.

In particular, tools to embed files in golang executables usually return `http.FileSystem`s (see statik, packr, vfsgen, etc).

Allowing to use these with Afero leads to some really useful stuff,
such as using `afero.CopyOnWriteFs` to "overlay" a configuration folder/memory map over the `http.FileSystem` returned from such tools.

## Example

```go
import (

    "github.com/rakyll/statik/fs"
    "github.com/spf13/afero"
    "github.com/dkumor/revhttpfs"
)

func OverwritableBuiltinFileSystem() (afero.Fs,error) {
    statikFS, err := fs.New()
    if err!=nil {
        return nil,err
    }

    aferoStatikFs := revhttpfs.NewReverseHttpFs(statikFS)
    overwritableStatikFs := afero.NewMemMapFs()

    return afero.NewCopyOnWriteFs(aferoStatikFs, pluginFs),nil
}
```

The above code uses resources embedded in the executable with [statik](https://github.com/rakyll/statik), which come in the form of an `http.FileSystem`, uses `ReverseHttpFs` to convert it to an `afero.Fs`, and then creates a copy-on-write overlay
over the static built-in files, using a memory map.
