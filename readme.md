<!--
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: /readme.source.md
To change this file edit the source file and then run MarkdownSnippets.
-->

# <img src="/src/icon.png" height="30px"> MarkdownSnippets

[![Build status](https://ci.appveyor.com/api/projects/status/8ijthhby6mhw8fk3/branch/master?svg=true)](https://ci.appveyor.com/project/SimonCropp/MarkdownSnippets)
[![NuGet Status](https://img.shields.io/nuget/v/MarkdownSnippets.Tool.svg?label=dotnet%20tool)](https://www.nuget.org/packages/MarkdownSnippets.Tool/)
[![NuGet Status](https://img.shields.io/nuget/v/MarkdownSnippets.MsBuild.svg?label=MsBuild%20Task)](https://www.nuget.org/packages/MarkdownSnippets.MsBuild/)
[![NuGet Status](https://img.shields.io/nuget/v/MarkdownSnippets.svg?label=.net%20API)](https://www.nuget.org/packages/MarkdownSnippets/)

A [dotnet tool](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) that extract snippets from code files and merges them into markdown documents.


<!-- toc -->
## Contents

  * [Installation](#installation)
  * [Usage](#usage)
  * [Defining Snippets](#defining-snippets)
  * [Using Snippets](#using-snippets)
  * [Snippet Exclusions](#snippet-exclusions)
  * [Mark resulting files as read only](#mark-resulting-files-as-read-only)
  * [Table of contents](#table-of-contents)
  * [Header](#header)
  * [Markdown includes](#markdown-includes)
  * [LinkFormat](#linkformat)
  * [UrlPrefix](#urlprefix)
  * [Security contact information](#security-contact-information)<!-- endtoc -->
  * [.net API](/docs/api.md)
  * [MsBuild Task](/docs/msbuild.md)
  * [Config file convention](/docs/config-file.md)
  * [Indentation](/docs/indentation.md)
  * [Max Width](/docs/max-width.md)


## Installation

Ensure [dotnet CLI is installed](https://docs.microsoft.com/en-us/dotnet/core/tools/).

Install [MarkdownSnippets.Tool](https://nuget.org/packages/MarkdownSnippets.Tool/)

```ps
dotnet tool install -g MarkdownSnippets.Tool
```


## Usage

```ps
mdsnippets C:\Code\TargetDirectory
```

If no directory is passed the current directory will be used, but only if it exists with a git repository directory tree. If not an error is returned.


### Behavior

 * Recursively scan the target directory for all non [ignored files](#ignore-paths) for snippets.
 * Recursively scan the target directory for all `*.source.md` files.
 * Merge the snippets with the `.source.md` to produce `.md` files. So for example `readme.source.md` would be merged with snippets to produce `readme.md`. Note that this process will overwrite any existing `.md` files that have matching `.source.md` files.


### mdsource directory convention

There is a secondary convention that leverages the use of a directory named `mdsource`. Where `.source.md` files are placed in a `mdsource` sub-directory, the `mdsource` part of the file path will be removed when calculating the target path. This allows the `.source.md` to be grouped in a sub directory and avoid cluttering up the main documentation directory.

When using the `mdsource` convention, all references to other files, such as links and images, should specify the full path from the root of the repository. This will allow those links to work correctly in both the source and generated markdown files. Relative paths cannot work for both the source and the target file.


## Defining Snippets

Any code wrapped in a convention based comment will be picked up. The comment needs to start with `begin-snippet:` which is followed by the key. The snippet is then terminated by `end-snippet`.

```
// begin-snippet: MySnippetName
My Snippet Code
// end-snippet
```

Named [C# regions](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-region) will also be picked up, with the name of the region used as the key.

To stop regions collapsing in Visual Studio [disable 'enter outlining mode when files open'](/docs/stop-regions-collapsing.png). See [Visual Studio outlining](https://docs.microsoft.com/en-us/visualstudio/ide/outlining).


## Using Snippets

The keyed snippets can be used in any documentation `.md` file by adding the text `snippet: KEY`.

Then snippets with that key.

For example

<pre>
Some blurb about the below snippet
snippet&#58; MySnippetName
</pre>

The resulting markdown will be:

    <!-- snippet: MySnippetName -->
    Some blurb about the below snippet
    <a id='snippet-MySnippetName'/></a>
    ```
    My Snippet Code
    ```
    <sup><a href='/relativeUrlToFile#L1-L11' title='File snippet `MySnippetName` was extracted from'>snippet source</a> | <a href='#snippet-MySnippetName' title='Navigate to start of snippet `MySnippetName`'>anchor</a></sup>
    <!-- endsnippet -->

Notes:

 * The vertical bar ( | ) is used to separate adjacent links as per web accessibility recommendations: https://webaim.org/techniques/hypertext/hypertext_links#groups
 * [H33: Supplementing link text with the title attribute](https://www.w3.org/TR/WCAG20-TECHS/H33.html)


### Including full files

When snippets are read all source files are stored in a list. When searching for a snippet with a specified key, and that key is not found, the list of files are used as a secondary lookup. The lookup is done by finding all files that have a suffix matching the key. This results in the ability to include full files as snippets using the following syntax:

<pre>
snippet&#58; directory/FileToInclude.txt
</pre>

The path syntax uses forward slashes `/`.


## Snippet Exclusions


### Exclude directories from snippet discovery

To exclude directories use `-e` or `--exclude`.

For example the following will exclude any directory containing 'foo' or 'bar'

```ps
mdsnippets -e foo:bar
```


### Ignored paths

When scanning for snippets the following are ignored:

 * All directories and files starting with a period `.`
 * All binary files as defined by https://github.com/sindresorhus/binary-extensions/
 * Any of the following directory names: `bin`, `obj`


## Mark resulting files as read only

To mark the resulting `.md` files as read only use `-r` or `--readonly`.

This can be helpful in preventing incorrectly editing the `.md` file instead of the `.source.md` file.

```ps
mdsnippets -r true
```


## Table of contents

If a line is `toc` it will be replaced with a table of contents

So if a markdown document contains the following:

<!-- snippet: tocBefore.txt -->
<a id='snippet-tocBefore.txt'/></a>
```txt
# Title

toc

## Heading 1

Text1

## Heading 1

Text2
```
<sup><a href='/docs/mdsource/toc/tocBefore.txt#L1-L11' title='File snippet `tocBefore.txt` was extracted from'>snippet source</a> | <a href='#snippet-tocBefore.txt' title='Navigate to start of snippet `tocBefore.txt`'>anchor</a></sup>
<!-- endsnippet -->

The result will be rendered:

<!-- snippet: tocAfter.txt -->
<a id='snippet-tocAfter.txt'/></a>
```txt
# Title

<!-- toc -->
## Contents

 * [Heading 1](#heading-1)
 * [Heading 2](#heading-2)
<!-- endtoc -->

## Heading 1

Text1

## Heading 2

Text2
```
<sup><a href='/docs/mdsource/toc/tocAfter.txt#L1-L16' title='File snippet `tocAfter.txt` was extracted from'>snippet source</a> | <a href='#snippet-tocAfter.txt' title='Navigate to start of snippet `tocAfter.txt`'>anchor</a></sup>
<!-- endsnippet -->


### Heading Level

Headings with level 2 (`##`) or greater can be rendered. By default all level 2 and level 3 headings are included.

To include more levels use the `--toc-level` argument. So for example to include headings levels 2 though level 6 use:

```ps
mdsnippets --toc-level 5
```


### Ignore Headings

To exclude headings use the `--toc-excludes` argument. So for example to exclude `heading1` and `heading2` use:

```ps
mdsnippets --toc-excludes heading1:heading2
```

## Header

When a .md file is written, a header is include. The default header is:

<!-- snippet: HeaderWriterTests.DefaultHeader.verified.txt -->
<a id='snippet-HeaderWriterTests.DefaultHeader.verified.txt'/></a>
```txt
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: {relativePath}
To change this file edit the source file and then run MarkdownSnippets.
```
<sup><a href='/src/Tests/HeaderWriterTests.DefaultHeader.verified.txt#L1-L4' title='File snippet `HeaderWriterTests.DefaultHeader.verified.txt` was extracted from'>snippet source</a> | <a href='#snippet-HeaderWriterTests.DefaultHeader.verified.txt' title='Navigate to start of snippet `HeaderWriterTests.DefaultHeader.verified.txt`'>anchor</a></sup>
<!-- endsnippet -->

To disable the header use `--write-header`

```ps
mdsnippets --write-header false
```

To apply a custom header use `--header`. `{relativePath}` will be replaced with the relative path of the `.source.md` file.

```ps
mdsnippets --header "GENERATED FILE - Source File: {relativePath}"
```

To insert a newline use `\n`

```ps
mdsnippets --header "GENERATED FILE\nSource File: {relativePath}"
```


## Markdown includes

Markdown includes are pulled into the document prior to passing the content through the snippet insertion.


### Defining an include

Add a file anywhere in the target directory that is suffixed with `.include.md`. For example, the file might be named `theKey.include.md`.


### Using an include

Add the following to the markdown:

   ```
   include: theKey
   ```


## LinkFormat

Defines the format of `snippet source` links that appear under each snippet.

<!-- snippet: LinkFormat.cs -->
<a id='snippet-LinkFormat.cs'/></a>
```cs
namespace MarkdownSnippets
{
    public enum LinkFormat
    {
        GitHub,
        Tfs,
        Bitbucket,
        GitLab
    }
}
```
<sup><a href='/src/MarkdownSnippets/Processing/LinkFormat.cs#L1-L10' title='File snippet `LinkFormat.cs` was extracted from'>snippet source</a> | <a href='#snippet-LinkFormat.cs' title='Navigate to start of snippet `LinkFormat.cs`'>anchor</a></sup>
<!-- endsnippet -->

<!-- snippet: BuildLink -->
<a id='snippet-buildlink'/></a>
```cs
if (linkFormat == LinkFormat.GitHub)
{
    return $"{path}#L{snippet.StartLine}-L{snippet.EndLine}";
}

if (linkFormat == LinkFormat.Tfs)
{
    return $"{path}&line={snippet.StartLine}&lineEnd={snippet.EndLine}";
}

if (linkFormat == LinkFormat.Bitbucket)
{
    return $"{path}#lines={snippet.StartLine}:{snippet.EndLine}";
}

if (linkFormat == LinkFormat.GitLab)
{
    return $"{path}#L{snippet.StartLine}-{snippet.EndLine}";
}
```
<sup><a href='/src/MarkdownSnippets/Processing/SnippetMarkdownHandling.cs#L104-L124' title='File snippet `buildlink` was extracted from'>snippet source</a> | <a href='#snippet-buildlink' title='Navigate to start of snippet `buildlink`'>anchor</a></sup>
<!-- endsnippet -->


## UrlPrefix

UrlPrefix allows a string to be defined that will prefix all snippet links. This is helpful when the markdown file are being hosted on a site that is no co-located with the source code files. It can be defined in the [config file](/docs/config-file.md), the [MsBuild task](/docs/msbuild.md), and the dotnet tool.


## Security contact information

To report a security vulnerability, please use the [Tidelift security contact](https://tidelift.com/security). Tidelift will coordinate the fix and disclosure.


## Release Notes

See [closed milestones](../../milestones?state=closed).


## Credits

Loosely based on some code from https://github.com/shiftkey/scribble.


## Icon

[Down](https://thenounproject.com/AlfredoCreates/collection/arrows-5-glyph/) by [Alfredo Creates](https://thenounproject.com/AlfredoCreates) from [The Noun Project](https://thenounproject.com/).
