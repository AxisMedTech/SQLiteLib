SQLiteLib
==========

Easily build a custom SQLite static library for use in OSX and iOS frameworks and apps.

If you need a specific version of SQLite, or specific SQLite compilation options/features, read on.


---

**May 29, 2016: SQLiteLib updated for SQLite 3.13.0** ([changelog](CHANGELOG.md)).

**Requirements**: iOS 8.0+ / OSX 10.9+, Xcode 7.3+

**SQLite Included:** 3.13.0


### Installation

#### Manual Installation (ex. into a Framework)

1. Download a copy of SQLiteLib.
2. Embed the `SQLiteLib.xcodeproj` project in your own project.
3. Add the SQLiteLib.`sqlitecustom` target in the **Target Dependencies** section of the **Build Phases** tab of your application target.
4. Add `libsqlitecustom.a` to the **Linked Frameworks and Libraries** section of the **General** tab of your target.

That's it! (You'll probably also want to `#include "sqlite3.h"` somewhere. SQLiteLib copies this generated file to its project directory.)

#### Using in Swift

You probably shouldn't be using the raw SQLite C API in Swift. There are a bunch of great libraries available that wrap it.

For example: ([GRDB.swift](https://github.com/groue/GRDB.swift)).


### Customization

By default, SQLiteLib builds SQLite with options that match the built-in system version of SQLite on OSX and iOS (as of OSX 10.11.5, iOS 9.3.2), [with one exception*](#additional-details).

#### Specifying Additional SQLite Compilation Options

> By default, SQLiteLib compiles SQLite with options that match the built-in OSX/iOS version of SQLite (as of OSX 10.11, iOS 9.3.2), with one exception*.
> You only need to follow the steps below if you wish to customize the options.

To specify additional options:

1. Open `SQLiteLib-Custom.xcconfig`
2. Modify `CUSTOM_SQLLIBRARY_CFLAGS` to specify the additional options.

For example, to specify SQLITE\_ENABLE\_PREUPDATE\_HOOK, you would modify it like this:
```ini
CUSTOM\_SQLLIBRARY\_CFLAGS = -DSQLITE\_ENABLE\_PREUPDATE\_HOOK
```

That's it.
There is no need to modify any other files.


#### Compiling a Specific Version of SQLite

SQLiteLib currently ships with the source for SQLite 3.13.0.

If you'd like to compile a newer (or older) version, the process is simple:

1. Download a snapshot of the complete (raw) source tree for SQLite for the version you'd like.
2. Replace the contents of SQLiteLib's "sqlite" subdirectory with the contents of the SQLite raw source tree snapshot.

> **IMPORTANT**:
> You **must** use the complete raw SQLite source. The amalgamation source will *NOT* work properly.
>
> SQLiteLib internally builds the amalgamation source using the steps specified in ([How To Compile SQLite](https://www.sqlite.org/howtocompile.html#amal)).
>
> Setting compilation options using the SQLite amalgamation is not guaranteed to work:
> > The versions of the SQLite amalgamation that are supplied on the download page are normally adequate for most users. However, some projects may want or need to build their own amalgamations. A common reason for building a custom amalgamation is in order to use certain compile-time options to customize the SQLite library. Recall that the SQLite amalgamation contains a lot of C-code that is generated by auxiliary programs and scripts. Many of the compile-time options effect this generated code and **must be supplied to the code generators before the amalgamation is assembled**.

**Quick Guide to Using the Latest version of SQLite**:

The snapshop of the complete (raw) source tree for the *current* version of SQLite is available on the ([SQLite Download Page](https://www.sqlite.org/download.html#old)) under: **Alternative Source Code Formats**. 
You'll want the file named "sqlite-src-*version*.zip".
> Do **NOT** use the file beginning with "sqlite-preprocessed" - it will not work properly.


### Additional Details

#### Default Compilation Options

The built-in OSX/iOS version of SQLite were built with the following compilation options (as of OSX 10.11.5, iOS 9.3.2):

> Fetched using `PRAGMA compile_options;`

- MacOSX (10.11.5)
    - ENABLE_API_ARMOR
    - ENABLE_FTS3
    - ENABLE_FTS3_PARENTHESIS
    - ENABLE_LOCKING_STYLE=1
    - ENABLE_RTREE
    - ENABLE_UPDATE_DELETE_LIMIT
    - OMIT_AUTORESET
    - OMIT_BUILTIN_TEST
    - OMIT_LOAD_EXTENSION
    - SYSTEM_MALLOC
    - THREADSAFE=2

- iPhoneOS (9.3.2)
    - ENABLE_API_ARMOR
    - ENABLE_FTS3
    - ENABLE_FTS3_PARENTHESIS
    - ENABLE_LOCKING_STYLE=1
    - ENABLE_RTREE
    - ENABLE_UPDATE_DELETE_LIMIT
    - MAX_MMAP_SIZE=0
    - OMIT_AUTORESET
    - OMIT_BUILTIN_TEST
    - OMIT_LOAD_EXTENSION
    - SYSTEM_MALLOC
    - THREADSAFE=2

SQLiteLib uses these settings with one exception - on iOS:

The SQLite code (verified in: 3.13.0) uses a deprecated function (`gethostuuid()`).

D. Richard Hipp (SQLite architect), suggests working around this on iOS using `-DSQLITE_ENABLE_LOCKING_STYLE=0`:
> "The SQLITE_ENABLE_LOCKING_STYLE thing is an apple-only extension that
> boosts performance when SQLite is used on a network filesystem.  This
> is important on MacOS because some users think it is a good idea to
> put their home directory on a network filesystem.
>
> I'm guessing this is not really a factor on iOS."

Thus, SQLiteLib uses SQLITE_ENABLE_LOCKING_STYLE=1 on OSX,
**but on iOS, SQLiteLib compiles with ENABLE_LOCKING_STYLE=0**.

This removes the code that uses the deprecated function, but doesn't get rid of the warning that "`gethostuuid() is disabled`" (as of 3.13.0).

To prevent this warning, SQLiteLib separately specifies `-Wno-#warnings` when building for iOS.

All of these base settings are configured in the SQLiteLib.xcconfig file.
It is strongly recommended that you do not edit this file. If you'd like to specify additional compilation options, see [the instructions above](#specifying-additional-sqlite-compilation-options)

#### Build Locations

SQLiteLib generates intermediate files in [${DERIVED_SOURCES_DIR}](https://developer.apple.com/library/mac/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html#//apple_ref/doc/uid/TP40003931-CH3-SW43).

The generated SQLite amalgamation files are copied to:

-"${BUILT_PRODUCTS_DIR}/sqlite3.c"

-"${PROJECT_DIR}/sqlite3.h"

#### Notes

##### "sqlite3.c" shows as red/missing in Xcode

Xcode (verified in Version 7.3.1 (7D1014)) will always show "sqlite3.c" as red/missing, even after a build.

This is a UI issue in Xcode - the path is properly set in the project.pbxproj file to be "Relative to Build Products", and the build should succeed.


