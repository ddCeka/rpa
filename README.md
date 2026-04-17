# Description
rpa is a command-line tool used to create and extract from renpy archives (rpa).

The program fully supports v3.0 and v2.0 and reading v3.2.

## Features

- **Fast threaded extraction.** Extract files from multiple archives at the same time using threads. Use `-m` to enable multi-threaded extraction for a single archive by lazy reading file into memory as needed.
- **Built-in glob pattern support.** Built-in support for glob pattern matching allows adding and removing files, and extracting and updating archives using patterns.
- **Minimal memory footprint.** rpa does not read archive into memory. It copies segments from the archive into specified location (extracting file or temporary archive depending on command).

## Usage

This and the following examples are focused on [rpa][rpa], the commandline tool. For information on [rpalib][rpalib] check out the [examples][examples].

```text
USAGE:
    rpa [OPTIONS] <SUBCOMMAND>

OPTIONS:
    -h, --help
            Print help information
    -k, --key <KEY>
            The encryption key used for creating v3 archives (default=0xDEADBEEF)
    -o, --override-version
            Override with default write version (3) if archive version does not support write
    -v, --verbose
            Provide additional information (default only shows errors)
    -V, --version
            Print version information
    -w, --write-version <WRITE_VERSION>
            The write version of archives

SUBCOMMANDS:
    add        Add files to existing or new archive
    extract    Extract files with full paths
    help       Print this message or the help of the given subcommand(s)
    list       List contents of archive
    remove     Delete files from archive
    update     Update existing archive by reading from filesystem
```

[rpa]: rpa/
[rpalib]: rpalib/
[examples]: rpalib/examples/

### Config

#### Key

The key argument can be used to specify the index table encryption key for the archive.
The default key is "0xDEADBEEF".

```bash
rpa -k BA5E7023 add path/to/archive.rpa file.txt
```

### Add

Add files to an archive either existing (will overwrite the existing file with the same path) or create a new archive with:

```bash
rpa add path/to/archive.rpa file1.txt file2.txt
```

Files can alternatively mapped to different paths than in filesystem with ARCHIVE=REAL pattern.
In the below example, the file is stored as archive.txt while being read from filesystem.txt

```bash
rpa add path/to/archive.rpa archive.txt=filesystem.txt
```

Or, alternatively you can add files based on glob patterns. The example below adds all files in images folder into the archive.

```bash
rpa add path/to/archive.rpa -p "images/**/*"
```

### Extract

Extract contents of a single archive into the archive directory with:

```bash
rpa extract path/to/archive.rpa
```

Or, specify a extraction target explicitly by providing the `--out` option. In the example below the contents are extracted into the current working directory.

```bash
rpa extract path/to/archive.rpa -o .
```

Extract multiples archives by providing them consecutively.

```bash
rpa extract path/to/archive.rpa path/to/another/archive.rpa
```

Or, use glob patterns to select specific files in the current working directory (and subdirectories). Here as `--out` is not specified, the files will be extracted relative to the archive directory.

```bash
rpa extract -a "**/*.rpa"
```

Or, you can even use unix commands like `find`

```bash
find . -type f -name "*.rpa" | xargs rpa extract
```

Extract has an optional and experimental `--memory` flag which enables multi-threaded read into archives. This allows for the extraction of multiple files from the archive at the same time. This works best with large archives containing many files.

```bash
rpa extract path/to/archive.rpa -m
```

### List

List out all the files from an archive with:

```bash
rpa list path/to/archive.rpa
```

### Remove

Remove files from an archive by specifying their full paths in archive.

```bash
rpa remove path/to/archive.rpa file1.txt file2.txt
```

Or, remove files that match a glob pattern. The example below removes all files ending with `.txt`.

```bash
rpa remove path/to/archive.rpa -p *.txt
```

You can alternatively keep the files matching by passing the `--keep` flag. This example keeps only the files ending with `.txt`.

```bash
rpa remove path/to/archive.rpa -p *.txt -k
```

### Update

You can update an existing archive by reading from the surrounding file system. This example tries to read all files that exist in archive from the filessystem. If the archive contains `README.md` then rpa would attempt to read `README.md` from the directory of the archive.

```bash
rpa update path/to/archive.rpa
```

The files being updated can be filtered using `--files` and `--pattern` arguments. The command below only updates `file1.txt` **and** other files that end in `.md`.

```bash
rpa update path/to/archive.rpa -f file1.txt -p "*.md"
```

rpa can be instructed to find files relative to another directory by giving the `--relative` argument. The command below will look for `README.md` in the current working directory.

```bash
rpa update path/to/archive.rpa -f README.md -r .
```

## License

This tool and library is licensed under [MIT License](LICENSE).
